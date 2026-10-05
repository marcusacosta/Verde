# Phase 1.7 source-specific validation

Date: 2026-10-05
Scope: CAISO / California; validation of the P1.6 representative samples

This document converts P1.6 observations into rules for a future ingestion pipeline. It deliberately distinguishes:

- **Validated fact:** observed in the acquired samples or supported by official source documentation.
- **Proposed rule:** a conservative implementation rule recommended for Verde, pending implementation-level testing.
- **Unresolved research decision:** a choice that remains open and must not be inferred by ingestion code.

No full 2020–2024 dataset was acquired. No forecasting model, scheduler, or final carbon-intensity target was built.

## 1. Executive findings

### Validated facts

- CAISO EMS load 2023 contains 8,760 data rows, an `HR` field, and utility-area columns including `CAISO`.
- CAISO production 2023 contains 105,120 data rows at five-minute granularity. Its read-me states that values are MW, production data is raw and not settlement-quality, and the production categories include Solar, Wind, Renewables, Nuclear, Large Hydro, Imports, Thermal, and Generation.
- The production report does not expose clean direct fields for natural gas, biomass, geothermal, or a general residual `other_generation_mw`.
- The OASIS sample provides GMT interval start/end timestamps, LMP results, node identifiers, market-run identifiers, and one-hour DAM results for the exploratory query. The all-APNODE result is not a single geographic price series.
- GHCNh station files contain hourly and synoptic observations, separate date/time components, per-variable quality/source/report metadata, and multiple observations within an hour. NOAA defines temperature in °C, wind speed in m/s, wind direction in whole degrees from true north, and precipitation as liquid precipitation in millimeters with totals nominally for the hour but potentially including intermediate reports.
- EPA eGRID and EIA Table A.3 are annual/reference emissions datasets. Neither is an hourly operational carbon-intensity measurement.

### Proposed rules

- Preserve raw source timestamps, native intervals, source fields, quality flags, and report metadata unchanged.
- Convert CAISO five-minute production MW to hourly MW using the arithmetic mean of a complete set of twelve five-minute intervals within the canonical hour. Do not interpolate missing intervals or sum MW values.
- Keep CAISO production fields at the source-supported aggregation level. Do not populate fuel-level canonical fields from `Thermal` or `Renewables` unless a future source provides an explicit defensible mapping.
- Treat hourly LMP construction as a derived feature. Require a documented node/hub, market run, LMP component, timezone conversion, interval rule, and geographic-scope rationale before populating `electricity_price_usd_mwh`.
- For NOAA, retain all source observations first; derive hourly weather values only after selecting a station/representation and an explicit variable-specific aggregation rule.
- Treat NOAA quality codes and source/report codes as provenance-bearing fields. A failed quality check should produce a missing/invalid value with a reason, not silent replacement.
- Use annual eGRID/EIA information only as inputs or comparison factors for a future estimated operational carbon signal. Do not label the result measured hourly marginal carbon intensity.

### Unresolved research decisions

- Exact CAISO load scope and local timestamp interpretation.
- DST representation in CAISO local-time reports and the final fold encoding.
- Final OASIS node/hub, market, product, LMP component, and geographic matching rule.
- NOAA station set or station aggregation, local timezone interpretation, and weather aggregation window.
- Carbon boundary, emissions scope, factor precedence, resource mapping, and methodology version.
- Final missingness thresholds, outlier policy, revision policy, and chronological split dates.

## 2. CAISO temporal validation

### 2.1 EMS load

**Validated facts.** The 2023 workbook stores the date as an Excel serial value and includes `HR`. The first data row is date serial `44927` (2023-01-01) with `HR=1`; the final date is serial `45291` (2023-12-31) with `HR=24`. The report therefore uses a 1–24 hourly ordinal associated with a report date, not a zero-based hour field.

**Proposed rules.** Preserve `Date` and `HR` in raw data. Treat `HR` as a source ordinal until the report's interval-start/interval-ending convention is verified. Convert to a timezone-aware interval key only in an interim transformation, recording the source convention and conversion. Reject `HR` outside 1–24 and validate expected row counts against the source calendar.

**Unresolved.** The workbook header does not state its timezone or explicitly say whether `HR=1` means the interval beginning at midnight or the first interval ending during the date. CAISO's local-time/DST guidance and transition-period samples must settle this before canonicalization.

### 2.2 Production/resource data

**Validated facts.** The production sheet contains `Date`, `Hour`, and `Interval`. The workbook formula representation shows `Hour = 1 + HOUR(Date)` and `Interval = 1 + MINUTE(Date)/5`; the first row is `Hour=1`, `Interval=1`. The annual row count is exactly 365 × 24 × 12. The source read-me states that data is in five-minute intervals and MW units.

**Proposed hourly aggregation rule.** For a canonical hourly interval, use the arithmetic mean of the twelve source MW observations belonging to that interval:

```text
hourly_MW = mean(five_minute_MW[12 complete intervals])
```

This preserves the average power over the hour; it is not a sum of MW values. A complete-block requirement is preferred for the first validated dataset. If one or more intervals are absent, emit the hourly value as missing with a documented reason rather than interpolating or silently averaging a partial block. A later sensitivity analysis may compare explicit partial-block policies, but that is not locked in here.

**DST rule.** Construct the grouping key from the source's verified local-time semantics and a timezone-aware representation. Preserve both occurrences of a repeated fall-back hour as distinct instants, or convert to UTC for ordering while retaining the local timestamp and fold metadata. Do not fabricate a spring-forward hour. The exact source-to-fold encoding remains unresolved until transition-period samples are acquired.

## 3. CAISO production mapping

| Canonical field | Validated source support | P1.7 decision |
|---|---|---|
| `solar_generation_mw` | Production `Solar` | Map is supported after five-minute-to-hourly aggregation. |
| `wind_generation_mw` | Production `Wind` | Map is supported after five-minute-to-hourly aggregation. |
| `imports_mw` | Production `Imports` | Map is supported, subject to confirming sign/net-interchange semantics. |
| `hydro_generation_mw` | `Large Hydro`; `Renewables` may include small hydropower | Do not combine these automatically. Large hydro can be retained as a separate source field; canonical hydro remains unresolved until category semantics are selected. |
| `geothermal_generation_mw` | No separate inspected field; included within aggregate `Renewables` if present | Do not map. |
| `biomass_generation_mw` | No separate inspected field; included within aggregate `Renewables` if present | Do not map. |
| `natural_gas_generation_mw` | `Thermal` is an aggregate of multiple technologies | Do not equate Thermal with natural gas. |
| `other_generation_mw` | No direct field | Do not create a residual without a documented reconciliation and category inventory. |

`Renewables`, `Thermal`, and `Generation` may be retained as source-specific interim fields. Canonical fields must not imply more fuel-level resolution than the source supports. Curtailment is separate and is not included in production quantities; it must not be added to generation without a future methodology decision.

## 4. OASIS price validation

### Validated facts

The exploratory `PRC_LMP` query returned `MARKET_RUN_ID=DAM`, `LMP_TYPE=LMP`, `XML_DATA_ITEM=LMP_PRC`, node identifiers, and `INTERVALSTARTTIME_GMT`/`INTERVALENDTIME_GMT`. The response's generic numeric column was named `MW`, so the semantic LMP field must be identified from report metadata, not from that column name. CAISO documents hub LMP pages for NP15, SP15, and ZP26 and states that the displayed real-time interval is in ending format; this demonstrates candidate hub concepts but does not select one.

Official OASIS documentation describes both day-ahead and real-time LMP report families. Real-time products may be five-minute or other sub-hourly intervals, while the exploratory DAM result was hourly. [CAISO OASIS](https://www.caiso.com/systems-applications/portals-applications/open-access-same-time-information-system-oasis) [OASIS interface specification](https://www.caiso.com/documents/oasis-interfacespecification.pdf)

### Proposed rules

- Do not use an all-node/all-APNODE response as the canonical price series.
- The eventual price series must select one documented market product and location representation, retain the raw node/hub identifier, and state whether the value is interval-start or interval-end.
- Convert the selected price interval to `America/Los_Angeles` only after preserving the original GMT timestamp.
- If sub-hourly prices are selected, document the hourly aggregation statistic. A simple arithmetic mean is not locked in because the appropriate statistic depends on the chosen market product and workload-cost interpretation.
- Require geographic consistency between the price location and the demand/generation scope. A hub price may be more reproducible than a node price but may represent a broader or different geography; a node may be more localized but less representative of CAISO-wide execution.

### Unresolved

The final node/hub, DAM versus RTM/HASP product, LMP component, interval aggregation, and geographic matching to the CAISO load/generation series remain open. `electricity_price_usd_mwh` is therefore not yet a populated canonical field.

## 5. NOAA weather validation

### Candidate comparison

The acquired station list shows multiple California candidates, including Los Angeles International (`USW00023174`), Sacramento (`USW00023232`), San Francisco (`USW00023234`), San Jose (`USW00023293`), and Fresno (`USW00093193`). The LAX file is a reconnaissance sample only. It does not justify selecting LAX or any fixed station set.

**Proposed station-screening rule.** Compare candidate stations on period of record across 2020–2024, availability of all required variables, observation density, quality-code rates, geographic position, and a documented relationship to the chosen CAISO scope. Defer spatial aggregation until that comparison is complete.

### Variable semantics

NOAA GHCNh documentation defines:

- `temperature`: degrees Celsius;
- `wind_speed`: meters per second;
- `wind_direction`: whole degrees from true north, with `000` used for calm winds;
- `precipitation`: liquid precipitation in millimeters, nominally for the hour but potentially including intermediate reports and duration-specific fields.

Source: [GHCNh documentation](https://www.ncei.noaa.gov/oa/global-historical-climatology-network/hourly/doc/ghcnh_DOCUMENTATION.pdf).

### Proposed weather representation rules

- Retain the original observation timestamp and report type.
- Do not select the nearest observation blindly. For each variable, choose observations using an explicit within-hour rule after station selection; the rule may differ by variable.
- Temperature and wind speed may be summarized hourly only after a coverage rule is defined.
- Wind direction must not be averaged arithmetically across 0/360 degrees. If an hourly aggregate is needed, use a documented circular/vector method or retain a representative observation.
- Use the documented nominal-hour precipitation field only after confirming its duration semantics; do not sum overlapping duration-specific precipitation fields.
- Preserve `*_Quality_Code`, measurement code, report type, source code, and source station ID through interim data.

NOAA states that general QC is applied to temperature, dew point, station pressure, sea-level pressure, wind direction, and wind speed, while other variables preserve component-source QC information. Quality codes must be interpreted using the source documentation before filtering.

## 6. EPA eGRID and EIA roles

### Validated facts

- eGRID2023 Revision 2 contains annual generator/plant, generation, emissions, balancing-authority, and subregion information. It includes emissions rates and boundaries, but it does not provide an hourly operational dispatch signal.
- EIA Table A.3 provides fuel-specific uncontrolled CO₂ factors, including fuel codes and special cases. It is not an hourly generation or marginal-emissions dataset.

### Defensible role

eGRID may provide annual emissions information, boundary/crosswalk context, and comparison factors. EIA may provide fuel-specific CO₂ factors or an independently documented factor reference. Either may support a future derived estimate if the mapping, boundary, scope, and version are explicitly documented.

Neither source justifies calling Verde's target measured hourly marginal carbon intensity. No final carbon formula, factor precedence, or imports treatment is established in P1.7. The target remains an estimated operational carbon-intensity signal whose methodology must be versioned when created.

## 7. General validation rules

These are proposed ingestion/data-contract rules, not claims that every source has already satisfied them.

| Area | Proposed rule |
|---|---|
| Missing data | Preserve missingness and reason codes. Do not interpolate raw data. Any modeling-time fill must be a separately versioned transformation with availability-safe inputs. |
| Duplicate timestamps | Reject duplicate canonical instants unless they are distinct DST-fold instants represented with unambiguous timezone/fold metadata. Never deduplicate by dropping a row. |
| DST/folds | Keep source timezone and original timestamp; canonicalize to timezone-aware `America/Los_Angeles`; use UTC ordering if useful; retain repeated-hour identity; never fabricate spring-forward intervals. |
| Outliers | Flag using source-aware physical and statistical checks; do not delete or winsorize during ingestion. Investigate source flags and revisions first. |
| Negative prices | Preserve negative OASIS LMPs as valid observations unless source documentation identifies them as invalid. Do not clamp to zero. |
| Source revisions | Record provider release/revision, retrieval time, effective interval, checksum, and whether the artifact is retrospective. Treat revised values as a new input version. |
| Quality flags | Preserve quality, measurement, source, and report codes. Exclude or downgrade values only under a documented variable-specific rule. |
| Units | Validate source units before conversion; store unit conversions and source field names in transformation metadata. |
| Physical validity | Flag impossible values such as negative generation where the source semantics disallow it, wind direction outside its documented domain, invalid calendar fields, or non-finite numeric values. Do not assume every negative value is impossible. |
| Provenance | Every derived table must reference input artifact checksums/manifests, source versions, code/configuration version, effective interval, transformations, and QC results. |
| Leakage | At forecast origin `t`, features may use only information available by `t` under the documented publication/revision policy. This includes revised values, weather reports, prices, imputed values, and derived carbon estimates. |
| Chronology | Sort by unambiguous instant, validate interval continuity after DST handling, and use chronological—not random—model splits. |

## 8. Decisions locked in by P1.7

The following are sufficiently supported to become implementation constraints:

1. CAISO production is treated as five-minute MW data; it is not directly hourly.
2. The default canonical production aggregation is the mean of twelve complete five-minute intervals per hour, with no interpolation and explicit missingness for incomplete blocks.
3. Solar, wind, and imports can be carried forward as candidate canonical mappings; fuel-level mappings from aggregate Thermal/Renewables cannot.
4. OASIS all-node results are not a canonical price series; a documented location/product decision is required.
5. GHCNh values and quality/source metadata must remain separate from any later hourly weather aggregation.
6. NOAA wind direction is circular/true-north data and must not be averaged arithmetically.
7. eGRID/EIA are annual/reference inputs only and cannot independently produce a measured hourly marginal signal.
8. Raw data remains immutable and ignored; all transformations, checks, versions, and checksums belong in provenance records.

## 9. Decisions required before P1.8

- Verify CAISO load and production DST transition behavior with targeted spring-forward and fall-back samples.
- Establish the exact CAISO load scope and interval semantics.
- Test the complete-block production aggregation rule on DST days and missing-interval cases.
- Choose a candidate OASIS product/location only after comparing geographic consistency, interval semantics, coverage, and revision behavior.
- Complete the NOAA station comparison and choose a station representation or explicitly defer weather features.
- Define variable-specific NOAA aggregation and quality-flag handling.
- Select eGRID/EIA releases, geographic boundaries, emissions scope, and factor precedence for a future carbon-methodology study.
- Define source revision and feature-availability policies for historical forecasting evaluation.
- Decide the canonical holiday calendar and `day_of_week` convention.
- Define chronological split dates only after source coverage and revisions are validated.

## 10. Validation checks performed

- Inspected all P1.6 raw sample formats, headers, source fields, and workbook read-me content.
- Verified CAISO 2023 load row count and 1–24 `HR` structure from the workbook XML.
- Verified CAISO production five-minute row count and `Date`/`Hour`/`Interval` structure.
- Inspected OASIS interval, market, node, and LMP fields from the extracted CSV.
- Inspected NOAA station candidates, sub-hourly timestamps, variable fields, and quality/source columns.
- Reviewed official NOAA GHCNh variable and QC documentation, CAISO OASIS documentation, and CAISO production publication notes.
- Confirmed eGRID/EIA workbook structure and annual/reference scope.
- Confirmed raw/interim/processed paths remain gitignored and no credentials were added.
