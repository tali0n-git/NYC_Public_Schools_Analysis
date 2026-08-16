# Data Processing Triage

Issues found in the existing cleaning notebooks, raised before resuming ML experimentation. Grouped by pipeline stage. Not yet prioritized — organize/triage below as needed.

## School Location Data (`SchoolLocationData_EDA.ipynb`)
- No canonical school key decided. `bn` (building number) and `Location Code` are not 1:1 — a building can house multiple schools. The notebook finds duplicate `Location Name`s under unique `Location Code`s but never resolves whether analysis should happen at the building level or the individual-school level. This ripples through every later join.
- Two different vintages of the same source file (`LCGMS_SchoolData_20251204_1133.csv` primary, `LCGMS_SchoolData_2025_2.csv` used later to backfill missing rows) are merged without documenting which is authoritative or why they diverge. `extraneous_data/LCGMS_SchoolData_additional_geocoded_fields_added.csv` was loaded and compared but never actually used in the final export — decide if it's needed or should be dropped.
- SYOSSET has a Queens (`Q`) location code despite being on Long Island, flagged as an open question in the notebook and never investigated. Could be a real DOE oddity (satellite program) or a data error — matters if it's silently fuzzy-matched into "QUEENS."
- Rows outside the five boroughs are dropped entirely with no count logged of how many, or a check on whether any are legitimate NYC DOE schools with just a messy `City` field.
- Lat/Long is ZIP-centroid, not geocoded — a precision problem baked in at the cleaning stage, not just an ML caveat.
- `Community District` is cleaned/typed but never actually used downstream — dead cleaning work, or a sign the geographic-grouping plan changed mid-project without cleanup.

## Quality Review Ratings (`QualityReviewRatings_2005-2020_EDA.ipynb`)
- ~~Rating-code meaning is still unverified.~~ **RESOLVED (2026-08-15).** The DOE publishes an actual data dictionary for this dataset — it's an attached `.xlsx` on the [NYC Open Data page](https://data.cityofnewyork.us/Education/2005-2020-Quality-Review-Ratings/3wfy-sn5g/about_data), not visible in the web UI, but reachable via the Socrata metadata API. Official codes: `O`=Outstanding (2007-08 only), `WD`=Well Developed, `P`=Proficient, `D`=Developing, `U`/`UD`=Underdeveloped (same rating, notation just varies by year), `UPF`=Underdeveloped with Proficient Features (2007-08 through 2009-10 only). Notebook and `ratings_rank` updated accordingly.
- **NEW — correctness bug found and fixed**: the old `ratings_rank` assigned `U` and `UD` two *different* ordinal values (4 and 3), but they're the same rating per the DOE dictionary. This was silently penalizing schools rated "Underdeveloped" in years that happened to use the `UD` spelling relative to years using `U`. Fixed to map both to the same rank.
- **NEW — data availability cliff, not just a cleaning issue**: cross-checked the DD's claim against the raw CSV directly — `overall_rating` is `'No Data'` for literally every row from school year 2014-15 through 2019-20 (3,180 of 9,009 rows, ~35%). This isn't sparse missingness, it's the DOE's own note: *"beginning in 2014-15, overall ratings were removed."* Any analysis using `overall_rating` as a target is really a **2005–2014 study**, not 2005–2020 — worth stating that explicitly rather than letting the dataset's name imply otherwise. A validation cell demonstrating this now lives in the notebook.
- **Still open**: `DYO` is not a documented code anywhere in the official dictionary. It appears only in school year 2006-07 (30 rows) and isn't a valid option for that year per the DD. Notebook now drops these rows (with a printed count) rather than guessing at a rank — flagged as a genuine unresolved mystery, good candidate for a FOIL request rather than more searching.
- Confirmed by the same DD (not just inferred): the rating scale genuinely isn't ordinal-comparable across eras — 2007-08 briefly had a 6-level scale including `O`, 2008-09/09-10 added `UPF` as a 5th tier, then from 2010-11 on it settled to 4 levels. A `D` in 2007-08 isn't drawn from the same rubric as a `D` in 2013-14. No code change made for this yet — still worth deciding whether cross-era averaging should be adjusted or footnoted.
- `No Data` is still encoded as `0` on the same numeric scale as real ratings, and nothing filters it out before averaging — this silently drags down every average/aggregate computed downstream. (Not touched by the rating-code fix above — separate open item.)
- Trailing label row dropped by hardcoded index (`.drop(results_df.index[-1])`) instead of by content — fragile if the raw export format ever changes.
- Lowercase `bn` prefix fix only checks `'q'` and `'k'` — no verification that `'m'`, `'r'`, `'x'` don't have the same casing issue.
- Unresolved count discrepancy: notebook expects 1,766 schools, gets 1,762, and just notes the gap rather than tracking down the missing 4.
- Multi-school buildings get one aggregated rating. Where a `bn` maps to multiple `Location Code`s, `agg_add_schools` concatenates the *names* into a semicolon-joined string but the QR score itself stays a single value attributed to all of them — distinct schools sharing a building inherit an identical rating that may only really belong to one of them.
- Excel serial date conversion assumes `origin='1900-01-01'` — worth double-checking this wasn't exported from a Mac-Excel workbook (1904 date system), which would silently shift every date by ~4 years if wrong.

## MTA Subway Stations (`MTA_Subway_Stations_EDA.ipynb`)
- Two different nearest-neighbor techniques used inconsistently across the pipeline: ZIP-code assignment here uses a plain Euclidean `KDTree` on raw lat/lng degrees (distorted at NYC's latitude), while the later school-to-station distance calc in `k-means-tests.ipynb` correctly uses a haversine `BallTree`. Same category of problem, solved two different ways.
- Census tract geocoding depends on a live Census Bureau API call (8–9 min run), with no caching of raw results and no audit of how many rows came back `"Error: ..."` or `"No Tract Found"` before being written to `cleaned_data/`.
- Duplicate `Station ID`s dismissed as "no concern" based on unique `GTFS Stop ID`, but never checked for downstream impact on station counts per line.

## MTA On-Time Performance (`MTA_On-Time_Performance_EDA.ipynb`)
- No overlap/duplicate check when concatenating the 2015–2019 and 2020–2024 files — e.g., whether January 2020 exists in both.
- `day_type` (weekday/weekend) is preserved but never split out in any downstream grouping — everything gets averaged across day types, which matters if commute-hour performance is ever the story (per the README's "peak commute" idea).
- Granularity mismatch that's really a data-modeling issue, not just cleanup: this is *terminal*-level on-time performance, not station-level. Every stop on a line gets the same number regardless of how close it is to the terminal vs. the far end of the route — so "MTA performance near this school" is actually "line-wide performance," a much coarser claim than the geographic framing implies.

## MTA Subway Groupings (`MTA_Subway_Groupings.ipynb`)
- Shuttle-line borough heuristic has a silent fallback gap: `update_daytime_routes` only reassigns `S` → `GS`/`FS`/`RS` for Manhattan/Brooklyn/Queens; any shuttle station in another borough silently keeps the generic `'S'` label instead of erroring or flagging.
- SIR (Staten Island Railway) has no performance metrics and was excluded from the metrics join, but it's unclear whether it still exists in the final grouped output with an empty/missing `mta_performance_data` field — git history shows this already caused an imbalanced-clustering issue once.
- Performance data is stored as the full multi-year history per line, never time-aligned to a specific QR assessment date — this is the concrete version of the "MTA performance isn't a real feature yet" gap.
- List-typed columns (station lat/lng lists, tract lists) round-tripped through CSV — they get serialized as string representations and have to be re-parsed with `ast.literal_eval` downstream, wrapped in a broad try/except that silently skips malformed rows. Fragile format choice for anything meant to be reloaded.

## Cross-cutting
- Two unreconciled "sources of truth" for Census tract: schools get tract from the DOE/LCGMS export, stations get tract from a live Census Bureau geocode — not verified to be the same tract vintage (2010 vs. 2020 boundaries) or even the same methodology.
- No schema/row-count validation anywhere. None of the notebooks assert final null rates, row counts, or dtypes before writing to `cleaned_data/` — a silent regression in an upstream join wouldn't be caught.
- No unified missing-value convention — missingness shows up as `NaN`, the string `'No Data'`, and numeric `0` depending on which notebook you're in, with no single rule for how the ML stage should treat any of them.
