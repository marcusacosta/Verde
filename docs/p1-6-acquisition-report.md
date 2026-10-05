# Phase 1.6 acquisition reconnaissance report

Date: 2026-10-05
Scope: CAISO / California; representative official samples only
Study period: 2020–2024 remains the intended period, not yet verified as complete

No full ingestion pipeline, canonical dataset, carbon target, forecast model, or scheduler was implemented. The artifacts inspected below are raw samples and were not silently transformed.

## A. Acquisition status

| Source | Acquired | Access method | Format | Coverage observed | Major issue |
|---|---:|---|---|---|---|
| CAISO historical EMS load | Yes | Official CAISO Library XLSX | XLSX | 2023 annual report; one header plus 8,760 hourly rows | Workbook uses Excel serial dates and local report semantics require explicit timezone/DST verification |
| CAISO production/resource | Yes | Official CAISO Library XLSX | XLSX | 2023 annual report; 105,120 five-minute intervals plus header | Native resolution is five minutes, not hourly; categories are aggregates and do not directly match every Verde field |
| CAISO OASIS prices | Yes | Public OASIS `SingleZip` API | ZIP/CSV | One requested 2024-01-15 operating hour; all-APNODE result set | Generic `MW` CSV column carries numeric LMP result; query returns many nodes; final price concept remains unresolved |
| NOAA GHCNh | Yes | Official NCEI bulk download | PSV | 2023 LAX candidate file; 12,453 observations plus header; station list also acquired | Observations include minute-level synoptic reports, not one clean hourly row per hour; station selection and aggregation remain unresolved |
| EPA eGRID | Yes | Official EPA XLSX download | XLSX | eGRID2023 Rev. 2 workbook; multiple annual plant, generation, emissions, balancing-authority, and subregion sheets | Annual emissions/generation information is not an hourly carbon signal |
| EIA emissions factors | Yes | Official EIA Table A.3 XLSX | XLSX | Fuel-factor table inspected | Factors are fuel-specific CO₂ factors in kg CO₂/MMBtu, not hourly or necessarily CO₂e factors |

## B. Canonical mapping findings

| Verde field | Source field(s) | Native/derived | Unit | Confidence | Open question |
|---|---|---|---|---|---|
| `demand_mw` | CAISO EMS load columns `SDGE`, `VEA`, `PGE`, `SCE`, `CAISO` | Source-native report values; canonical CAISO series interpretation still requires confirmation | MW by report context | Medium | Whether `CAISO` total or another scope is the correct canonical demand definition |
| `solar_generation_mw` | CAISO production `Solar` | Source-native aggregate; hourly canonical value would be derived by aggregation | MW | High for field identity | Five-minute-to-hourly aggregation rule and DST treatment |
| `wind_generation_mw` | CAISO production `Wind` | Source-native aggregate; hourly canonical value would be derived by aggregation | MW | High for field identity | Five-minute-to-hourly aggregation rule |
| `hydro_generation_mw` | CAISO production `Large Hydro`; possibly part of `Renewables` for small hydro | Requires category mapping; not a direct one-to-one canonical field | MW | Low/medium | Treatment of large versus small hydro and category definitions |
| `geothermal_generation_mw` | CAISO production `Renewables` only in the inspected aggregate | Requires source/category detail not present in the top-level production columns | MW | Low | Whether geothermal can be separated from the available report |
| `biomass_generation_mw` | CAISO production `Renewables` only in the inspected aggregate | Requires category mapping/detail | MW | Low | Whether biomass/biogas can be separated |
| `natural_gas_generation_mw` | CAISO production `Thermal` | Requires technology/fuel decomposition | MW | Low/medium | Thermal includes multiple technologies; natural gas cannot be assumed equal to thermal |
| `imports_mw` | CAISO production `Imports` | Source-native aggregate | MW | High | Sign convention and net actual interchange semantics |
| `other_generation_mw` | No direct inspected field | Derived residual or category mapping would be required | MW | Low | Must not be constructed until category coverage and residual definition are established |
| `electricity_price_usd_mwh` | OASIS `XML_DATA_ITEM=LMP_PRC`; numeric value appeared under generic `MW` column | Source-derived candidate; final hourly price field not established | Likely USD/MWh by LMP semantics, but CSV header does not state unit | Medium | Final node/hub, market/product, LMP component, and hourly aggregation |
| `temperature_c` | GHCNh `temperature` | Source-native observation; conversion/scaling requires documentation | Source encoding requires GHCNh documentation confirmation | Medium | Unit encoding, report-type selection, and station representation |
| `wind_speed_mps` | GHCNh `wind_speed` | Source-native observation; conversion/scaling requires documentation | Source encoding requires GHCNh documentation confirmation | Medium | Unit encoding and hourly representation |
| `wind_direction_deg` | GHCNh `wind_direction` | Source-native observation | Degrees, convention unresolved | Medium | Meteorological convention and handling of calm/variable reports |
| `precipitation_mm` | GHCNh `precipitation` and duration-specific fields | Requires selecting a duration-specific field and conversion | Source encoding requires documentation confirmation | Low/medium | Hourly accumulation semantics and field selection |
| `hour` | GHCNh `Hour`, CAISO/OASIS timestamps after timezone normalization | Derived local-calendar feature | 0–23 | High | Must derive from timezone-aware canonical timestamp |
| `day_of_week` | Canonical timestamp | Derived | 0–6, convention unresolved | High | Numbering convention |
| `day_of_year` | Canonical timestamp | Derived | 1–366 | High | DST does not change calendar-day semantics |
| `month` | Canonical timestamp | Derived | 1–12 | High | None beyond timezone handling |
| `is_weekend` | Canonical timestamp | Derived | Boolean | High | Depends on `day_of_week` convention only operationally |
| `is_holiday` | Canonical timestamp plus future holiday calendar | Derived | Boolean/unknown | Low | Holiday jurisdiction and calendar source |
| `estimated_carbon_intensity_kg_mwh` | Future combination of CAISO data and emissions information/factors | Derived target; not present in acquired samples | Declared unit/scope required before construction | Not established | Carbon methodology, boundary, factors, mapping, and version remain open |

## C. Timestamp / DST findings

### CAISO EMS load

The workbook stores dates as Excel serial values and an `HR` column. The 2023 sheet contains 8,760 data rows, consistent with one non-leap-year row per hour. The raw workbook does not itself provide an explicit timezone label in the inspected header. CAISO local-time/DST semantics must therefore be verified from report documentation before canonicalization.

### CAISO production/resource

The production sheet contains `Date`, `Hour`, and `Interval`; the first data row has `Interval=1`, and the workbook read-me states that the data is in five-minute intervals. The 2023 sheet has 105,120 data rows, consistent with 365 × 24 × 12. This is incompatible with directly treating the source as hourly. The report’s local-time/DST representation and any repeated/missing interval behavior require explicit inspection across a DST transition.

### CAISO OASIS

The sample CSV uses `INTERVALSTARTTIME_GMT` and `INTERVALENDTIME_GMT` with explicit `-00:00` offsets. The inspected rows use one-hour intervals and `OPR_DT`/`OPR_HR` fields. The result is GMT/UTC-oriented and must be converted only under a documented canonical-time rule. No DST ambiguity was observed in the one-hour sample.

### NOAA GHCNh

The PSV uses ISO-like timestamps without an offset, for example `2023-01-01T00:00:00`, and separate `Year`, `Month`, `Day`, `Hour`, and `Minute` fields. The file includes observations at `00:00`, `00:53`, `01:20`, and `01:53`, demonstrating that the station file is not inherently one row per hour. The source station list provides coordinates, while the observation file carries station metadata. A timezone interpretation and hourly aggregation rule must be established before joining to CAISO data.

### EPA/eGRID and EIA

These samples are annual/reference tables rather than time-series observations, so DST handling does not apply directly.

## D. Coverage findings

- CAISO EMS load: official library search exposed annual reports for 2020, 2021, 2022, and 2023, plus monthly 2024 reports. The 2023 sample is available and complete-looking by row count; continuity and report revisions across 2020–2024 remain unverified.
- CAISO production/resource: official library search exposed annual reports for 2020–2024. CAISO states that the comprehensive production and curtailments report stopped being published after 2025-06-01; the 2023 report is available for the intended period. Product continuity and category changes across years remain to be checked.
- CAISO OASIS: the official OASIS page states that historical data beyond 39 months is available through the Historical OASIS Data Downloader and as far back as 2016. The sample demonstrates access for 2024 but does not establish complete 2020–2024 coverage for any final query.
- NOAA GHCNh: a 2023 LAX candidate file and a current station list were obtained. Station period-of-record coverage for all candidate California stations and all intended years remains unverified.
- EPA eGRID: the official page identifies eGRID2023 and revisions; the acquired artifact is Revision 2. eGRID historical releases include 2020–2022, but release-year alignment to each study year and boundary mapping remain unresolved.
- EIA: Table A.3 is a current Electric Power Annual fuel-factor table with 2024 release context. It is a reference table, not a 2020–2024 hourly coverage source.

## E. Data-quality findings

- CAISO EMS load: no null scan or full semantic validation was performed; the workbook has a simple tabular header and numeric-looking values. Excel serial dates and report-hour indexing require validation. No imputation or correction was applied.
- CAISO production/resource: the read-me explicitly describes the data as raw, not operational or settlement quality. Production values are in MW. Curtailments are in a separate table and are not counted in production quantities. Resource aggregates do not expose all desired Verde fuel categories.
- CAISO OASIS: the ZIP contained a large all-APNODE CSV for a one-hour query. Rows were not returned in chronological order in the sample. The generic `MW` column name is potentially misleading for LMP values. No duplicate/null policy was applied.
- NOAA GHCNh: the observation file includes per-variable measurement, quality, report-type, source-code, and source-station fields. Multiple observations per hour occur. Blank fields are common for unavailable variables. Quality flags must be interpreted from GHCNh documentation; no filtering or imputation was applied.
- EPA eGRID: the workbook contains multiple sheets and revision materials; sheet-level semantics must be selected deliberately. It should not be interpreted as an hourly dataset.
- EIA: Table A.3 contains fuel codes, factor values, notes, and special cases such as geothermal steam versus binary cycle and factors marked as dependent on source fuel. It reports CO₂ factors, not automatically CO₂e factors.

## F. Licensing / redistribution findings

- The samples were acquired from official public CAISO, NOAA/NCEI, EPA, and EIA URLs without credentials.
- CAISO page terms/copyright and OASIS usage/redistribution terms require explicit review before redistribution of raw or derived artifacts.
- NOAA/NCEI, EPA, and EIA source attribution and data-use terms should be retained with the relevant manifest; this reconnaissance does not make a legal determination.
- If an artifact cannot be redistributed, Verde should retain source/access instructions, release/version, query parameters, retrieval timestamp, checksum, and transformation metadata while keeping the raw artifact outside Git.

## G. Unresolved decisions

- Final CAISO load scope and exact report interpretation/timezone semantics.
- CAISO production category mapping for hydro, geothermal, biomass, natural gas, and other generation.
- Five-minute-to-hourly aggregation rule and DST treatment for production.
- OASIS final node/hub, market, product, LMP component, and aggregation rule.
- NOAA station set, spatial representation, timezone interpretation, variable scaling, and hourly aggregation.
- NOAA wind-direction convention, precipitation-duration field, and quality-flag handling.
- eGRID release selection, boundary mapping, and use of plant/subregion/balancing-authority fields.
- EIA factor selection, emissions scope, and precedence relative to EPA information.
- Carbon target construction, geographic boundary, unit/scope, and methodology version.
- Missing-data, outlier, revision, imports, negative-price, and source-quality policies.
- Final chronological train/validation/test dates.

## H. P1.7 recommendations

1. Build source-specific validation scripts that inspect schemas without changing raw files.
2. Validate CAISO 2020–2024 report availability and revisions, including representative spring/fall DST periods.
3. Define and test the production five-minute-to-hourly aggregation semantics before creating canonical generation fields.
4. Enumerate OASIS query products and node/hub candidates, then document a selection decision separately.
5. Inspect the OASIS historical downloader for older-than-39-month retrieval and version behavior.
6. Produce a NOAA candidate-station comparison covering period of record, variables, report types, quality flags, and local-time interpretation without finalizing the station set yet.
7. Map eGRID and EIA categories/factors to possible CAISO categories without constructing the carbon target.
8. Add automated checks for timezone awareness, duplicate timestamps, DST folds/gaps, nulls, sentinels, units, and chronological ordering.
9. Record all source revisions and availability cutoffs in tracked provenance manifests under `metadata/provenance/`.
