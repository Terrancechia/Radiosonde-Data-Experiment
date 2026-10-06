# Sounding inputs

The eight example text files are copied byte-for-byte from the original project. They contain two station labels (Singapore and Ulaanbaatar), two filename dates (2021-01-07 and 2021-07-07) and two UTC launch labels (00Z and 12Z). These labels are recorded in `soundings.csv`.

## Provenance

The accompanying previous report identifies the **University of Wyoming Atmospheric Science Radiosonde Archive** as the source. The supplied files contain stripped observation tables rather than complete download records, so station identifiers, exact retrieval URLs, retrieval dates and prior transformations remain unavailable. The manifest records the report's attribution in `notes` and leaves `source_url` blank rather than inventing an exact download link. Source units are checked against the supplied units rows, but that cannot verify earlier conversions.

For replacement or newly collected data, record the source URL, station identifier, UTC observation time, retrieval date, original units, missing-value codes and every conversion in the manifest notes or accompanying source metadata. Check provider terms for redistribution. No licence for these observations is inferred from their presence in the project.

## Required text format

Each observation uses **eleven fields of exactly seven characters each** (77 characters across a complete row). Preserve spaces: split-on-whitespace parsing would shift columns when a field is blank. A field can be blank, which the loader retains as `NaN`.

```text
-----------------------------------------------------------------------------
   PRES   HGHT   TEMP   DWPT   RELH   MIXR   DRCT   SPED   THTA   THTE   THTV
    hPa      m      C      C      %   g/kg    deg    m/s      K      K      K
-----------------------------------------------------------------------------
 1007.0     16   24.4   24.4    100  19.43     15    4.6  297.0  353.4  300.4
```

| Field | Meaning | Unit |
| --- | --- | --- |
| PRES | Pressure | hPa |
| HGHT | Recorded height | m |
| TEMP | Temperature | C |
| DWPT | Dew point | C |
| RELH | Relative humidity | % |
| MIXR | Water-vapour mixing ratio | g/kg |
| DRCT | Wind direction | deg |
| SPED | Wind speed | m/s |
| THTA | Potential temperature | K |
| THTE | Equivalent potential temperature | K |
| THTV | Virtual potential temperature | K |

The header tokens must match exactly, and the units row must immediately follow them. Blank lines and separator lines are allowed. Leading titles are allowed; trailing station summaries, HTML and footnotes are rejected. Readings must be numeric or blank: convert missing-value sentinels to blanks before writing. Archive formats such as native IGRA need conversion before use.

When creating normalized rows programmatically, a blank-preserving pattern is:

```python
fields = ["" if value is None else str(value) for value in values]
if len(fields) != 11 or any(len(field) > 7 for field in fields):
    raise ValueError("Expected eleven fields fitting within seven characters each.")
line = "".join(field.rjust(7) for field in fields)
```

Choose numeric precision appropriate to the original archive and explicitly convert its sentinel/NaN representation to `None` before formatting. Do not silently round a value merely to fit the table.

## Manifest

`soundings.csv` contains one row per launch. Required fields are `station`, `date`, `time_utc` and `file`. The station/date/time combination must be unique; dates are `YYYY-MM-DD`; supported launch labels are `00Z` and `12Z`. File paths are relative to this directory and must remain inside it. Optional `source_url` and `notes` fields document provenance. The notebook uses manifest labels for figures, so update them together with the input files.
