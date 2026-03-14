# jbp-new Notes

This folder contains a QGIS/QField project for field observations.

## Main Files

- Project: [jbp-new.qgs](/Users/pma/QField/cloud/jbp-new/jbp-new.qgs)
- Active observation data: [observations.gpkg](/Users/pma/QField/cloud/jbp-new/observations.gpkg)
- Species lookup: [species_list.gpkg](/Users/pma/QField/cloud/jbp-new/species_list.gpkg)
- Collector lookup: [collector_list.gpkg](/Users/pma/QField/cloud/jbp-new/collector_list.gpkg)
- Observation subject lookup: [observation_subject.gpkg](/Users/pma/QField/cloud/jbp-new/observation_subject.gpkg)

## Important Project Points

- The observation layer is stored in `EPSG:4326`.
- The project CRS is `EPSG:5514`.
- This is correct. Do not reassign the observation layer CRS to `5514` unless you explicitly reproject the data.
- `x_coord` and `y_coord` currently come from `$x` / `$y`, so they store layer CRS coordinates, i.e. lon/lat.

## Attribute Form Logic

- `taxon_name` is the list-based taxon field.
- `name_proposition` is the manual-entry taxon field.
- `no_name_on_list` is used as the manual-entry mode switch.
- `MatchedCanonical` and `TaxonId` are lookup-derived fields from `species_list`; they stay empty for names that do not match the fixed list exactly.
- For a more robust setup, use a merged canonical field such as `taxon_name_final`:

```qgis
coalesce(
  nullif(trim("taxon_name"), ''),
  nullif(trim("name_proposition"), '')
)
```

- If `taxon_name_final` is added, downstream visibility and lookup logic should use it instead of raw `taxon_name`.

## QField / Photo Naming

- QField photo naming is driven by `QFieldSync/attachment_naming`, not `photo_naming`.
- One bug that was identified and fixed was a missing dot in the `picture_detail` filename pattern (`_03jpg` vs `_03.jpg`).
- The label `[Image #1]` shown in chat is only a chat attachment label and is unrelated to the actual stored filename on disk.

## GeoPackage / Project Naming

- Avoid using the same base name for the project file and a GeoPackage.
- Example of a problematic pairing:
  - `jbp-new.qgs`
  - `jbp-new.gpkg`
- SQLite sidecar files like `*.gpkg-wal` and `*.gpkg-shm` can trigger QFieldSync warnings.
- Preferred pattern:
  - project: `jbp-new.qgs`
  - data: `observations.gpkg`

## Layer/Form Recovery

- The attribute form definition lives in the QGIS project file, not in the raw GeoPackage table.
- If a layer is removed from the project, do not simply add the GeoPackage back as a new layer if you want to keep the existing form.
- Instead, repair the broken datasource for the existing project layer.
- The project backup [jbp-new.qgs~](/Users/pma/QField/cloud/jbp-new/jbp-new.qgs~) can also help recover the form configuration.

## Current Caution

- If QGIS reports a broken observation layer, check whether the project still points to an old datasource such as `jbp-new.gpkg|layername=jbp-new`.
- If needed, repair the source to:
  - file: [observations.gpkg](/Users/pma/QField/cloud/jbp-new/observations.gpkg)
  - layer: `observations`

## Practical Rules

- Use exact species-list spelling if you want `MatchedCanonical` and `TaxonId` to populate.
- Use manual entry only for names that are intentionally not in the fixed list.
- Do not rely on visibility alone to clear values; hidden fields can still retain data.
- Field default values with `Apply default value on update` affect future edits, not old rows. Use the Field Calculator to backfill existing records.
