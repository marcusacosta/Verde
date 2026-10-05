# Verde Phase 1 specification

Status: Phase 1.8 finalization
Scope: CAISO / California
Canonical resolution: hourly
Canonical timezone: `America/Los_Angeles`
Intended initial study period: 2020–2024

This document is the highest-level Phase 1 data specification. It supersedes earlier provisional wording where explicitly stated here. It does not authorize full acquisition, forecasting, scheduler implementation, or construction of a final carbon-intensity target.

Decision labels:

- **LOCKED** — supported by current evidence and part of the implementation contract.
- **PROVISIONAL** — a defensible initial implementation choice that must be revisited if source validation contradicts it.
- **UNRESOLVED** — evidence is insufficient; implementation must not silently choose.

## 1. Research objective

Verde will investigate whether lightweight time-series forecasting can provide useful information for scheduling flexible AI workloads under cost, carbon, and deadline constraints. Phase 1 establishes reproducible data semantics; it makes no savings or emissions-reduction claim.

**LOCKED:** Forecasting is the first research focus. The primary target is reserved for an estimated operational carbon-intensity signal; electricity price is a secondary target/feature.

## 2. Canonical hourly timeline

**LOCKED:** Canonical records represent one-hour intervals with a timezone-aware timestamp in `America/Los_Angeles`. The timestamp denotes the interval start once a source has been mapped to the canonical timeline. The original source timestamp, timezone, interval semantics, and native resolution must be retained in provenance metadata.

**LOCKED:** UTC may be used for ordering or storage, but it does not replace the canonical local-time semantic timestamp. Naive timestamps are invalid.

**LOCKED:** A repeated fall-back local hour represents two distinct instants and must not be deduplicated. A spring-forward gap must not be fabricated. The exact storage representation of fold metadata is **UNRESOLVED** until CAISO transition-period samples are inspected.

**PROVISIONAL:** Source-local timestamps will be converted using an explicit timezone-aware transformation only after the source's timestamp convention is verified. No transformation may infer missing DST information from row position alone.

### CAISO time findings

The EMS sample uses an Excel date serial and `HR=1..24`; the production sample uses `Date`, `Hour`, and `Interval`, with five-minute intervals. The samples do not alone establish whether EMS `HR=1` is interval-start or interval-end, nor the exact local-time encoding across DST. Those details remain **UNRESOLVED** and must be verified before canonicalization.

## 3. Source datasets

| Source | Phase 1 role | Status |
|---|---|---|
| CAISO historical EMS hourly load | Demand feature | **LOCKED** source family; exact report interpretation and scope are **PROVISIONAL/UNRESOLVED** as described below |
| CAISO historical production/resource data | Generation and import features | **LOCKED** source family; five-minute native resolution is **LOCKED** |
| CAISO OASIS | Candidate electricity-price feature | **LOCKED** source family; final location/product is **UNRESOLVED** |
| NOAA GHCNh | Weather features | **LOCKED** source family; station representation is **UNRESOLVED** |
| EPA eGRID | Annual emissions, boundary, and comparison information | **LOCKED** role; not an hourly signal |
| EIA fuel factors | Fuel-specific CO₂ factor reference | **LOCKED** role; not an hourly signal |

Raw artifacts remain immutable and gitignored. Source versions, revisions, retrieval timestamps, URLs, query parameters, effective intervals, and checksums belong in tracked provenance metadata when legally and practically possible.

## 4. Canonical feature schema

The canonical field names remain those in [`data-contract.md`](data-contract.md). Their Phase 1 status is:

| Field group | Status | Semantics |
|---|---|---|
| `timestamp` | **LOCKED** | Timezone-aware hourly interval key |
| `demand_mw` | **PROVISIONAL** | CAISO total EMS load candidate; final scope semantics require confirmation |
| `solar_generation_mw`, `wind_generation_mw` | **LOCKED** mapping rule | CAISO production aggregates after hourly aggregation |
| `imports_mw` | **PROVISIONAL** mapping rule | CAISO Imports after sign/net-interchange validation |
| `hydro_generation_mw` | **UNRESOLVED** | Large Hydro is available; small hydro may be inside Renewables; do not combine silently |
| `geothermal_generation_mw`, `biomass_generation_mw` | **UNRESOLVED** | Not separately exposed by the inspected production report |
| `natural_gas_generation_mw` | **UNRESOLVED** | `Thermal` is not equivalent to natural gas |
| `other_generation_mw` | **UNRESOLVED** | No defensible residual without a complete category reconciliation |
| `electricity_price_usd_mwh` | **UNRESOLVED** | Requires final OASIS location/product decision |
| Weather fields | **PROVISIONAL** | NOAA fields have documented units; station and hourly representation remain open |
| Calendar fields | **LOCKED/PROVISIONAL** | Derived from canonical timestamp; conventions specified below |
| `estimated_carbon_intensity_kg_mwh` | **UNRESOLVED target construction** | Reserved field; no values until the methodology is approved |

Source-native and source-derived fields may be retained in raw/interim tables. Processed feature tables may omit source-dependent fields, but missingness must be explicit. A supervised target table must not claim to satisfy the primary target contract until the target methodology is versioned.

## 5. Source → canonical mappings

### CAISO load scope

**PROVISIONAL:** Use the EMS workbook's `CAISO` total column as the initial demand candidate because it is the column representing the CAISO total in the acquired report and is the closest scope match to CAISO production totals. Utility-area columns (`SDGE`, `VEA`, `PGE`, `SCE`) remain source-specific alternatives, not separate canonical demand values.

**UNRESOLVED:** The exact balancing-area/report definition, whether the production report's load uses identical boundaries, and the EMS hour interval semantics require source documentation and transition-period validation. The initial implementation must retain all available load columns so this choice remains auditable.

### CAISO production

**LOCKED:** Map `Solar` to `solar_generation_mw` and `Wind` to `wind_generation_mw` only after the five-minute-to-hourly rule in Section 6.

**PROVISIONAL:** Retain `Imports` as `imports_mw` after confirming its sign convention and net actual interchange semantics.

**LOCKED:** Do not map `Thermal` to natural gas, `Renewables` to geothermal/biomass, or create `other_generation_mw` as an unexplained residual. Retain `Large Hydro`, `Renewables`, `Thermal`, `Nuclear`, and `Generation` as source-specific fields until category semantics are resolved.

### OASIS

**UNRESOLVED:** No single node or hub is selected by this specification. All-node/all-APNODE output is explicitly not a canonical series.

### NOAA

**UNRESOLVED:** No station or station set is selected. Candidate comparison must consider period of record, variable availability, quality-code rate, observation density, geography, and relationship to the CAISO scope.

### EPA/eGRID and EIA

**LOCKED:** These sources can support annual emissions/factor inputs, boundary/crosswalk information, or sensitivity comparisons. They cannot by themselves supply hourly dispatch or justify a measured hourly marginal signal.

## 6. Temporal aggregation rules

### CAISO production

**LOCKED:** Source values are five-minute MW averages. For each canonical hour, aggregate exactly twelve source intervals using their arithmetic mean:

```text
hourly_MW = mean(the 12 complete five-minute MW values)
```

Do not sum MW values. Do not interpolate missing five-minute intervals. If the block is incomplete, emit the hourly field as missing with a reason code. A later sensitivity study may evaluate alternative partial-block policies, but no such policy is part of the Phase 1 contract.

### OASIS

**UNRESOLVED:** The final market interval and hourly aggregation statistic depend on the selected product. The raw GMT interval start/end fields must be preserved. No generic averaging rule is authorized yet.

### NOAA

**PROVISIONAL:** Retain all observations first. Apply variable-specific within-hour aggregation only after station selection and quality semantics are fixed. Do not combine overlapping precipitation-duration fields.

## 7. Weather representation

**LOCKED units and semantics from GHCNh documentation:** temperature is °C, wind speed is m/s, wind direction is whole degrees from true north, and precipitation is liquid precipitation in millimeters with nominal hourly totals that may include intermediate reports. NOAA applies general QC to several core variables and preserves component-source QC for others. [GHCNh documentation](https://www.ncei.noaa.gov/oa/global-historical-climatology-network/hourly/doc/ghcnh_DOCUMENTATION.pdf)

**PROVISIONAL representation:** Preserve station-level observations and metadata in interim data. For the first modeling dataset, use a documented station or station-set representation rather than an implicit nearest-station join.

**UNRESOLVED:** Final station set, geographic weighting, local timezone treatment, within-hour selection/aggregation, missingness threshold, and quality-code acceptance rules.

**LOCKED:** Wind direction must not be arithmetically averaged across 0/360. If aggregated, use a documented circular/vector method or a representative valid observation. Precipitation must use one documented duration-semantic field; overlapping durations must not be summed.

## 8. Electricity-price representation

**Validated facts:** OASIS provides LMP report families with node identifiers and GMT interval fields. The exploratory DAM sample was hourly; other products can be sub-hourly. Hub pages expose candidate hub concepts such as NP15, SP15, and ZP26, but this does not establish a single CAISO-wide price.

**PROVISIONAL candidate strategy:** Prefer a hub-level series over an arbitrary node for the first comparison because it is easier to reproduce and less dependent on a single physical location. Compare NP15, SP15, and ZP26, and retain the location identifier. This is not a final selection.

**UNRESOLVED:** Final hub/node, DAM versus RTM/HASP, LMP component, hourly aggregation, interval start/end interpretation, and geographic consistency with the selected demand/generation scope. Until resolved, `electricity_price_usd_mwh` remains absent or source-specific rather than silently populated.

Negative prices are valid market observations unless the source identifies them as invalid; they must not be clamped to zero.

## 9. Carbon-target definition

### Target contract

**LOCKED:** The field name is `estimated_carbon_intensity_kg_mwh`, and it must never be described as measured hourly marginal carbon intensity. No target value may be released without a methodology/version identifier.

The eventual target metadata must specify:

- **Name:** `estimated_carbon_intensity_kg_mwh`.
- **Unit:** kg CO₂e/MWh or another explicitly declared emissions unit/scope; the final unit must be fixed before target construction.
- **Geographic boundary:** explicit CAISO/California boundary and treatment of imports.
- **Emissions scope:** operational/direct versus lifecycle/other scope.
- **Construction methodology:** formula, category mapping, aggregation, and assumptions.
- **Sources and versions:** CAISO generation/load, EPA eGRID release, EIA factor release, and any other source.
- **Limitations:** annual factors, incomplete fuel categories, imports, revisions, and uncertainty.
- **Methodology identifier:** versioned identifier such as `carbon_methodology_v1` once approved.

### Current construction decision

**UNRESOLVED / DEFERRED:** Do not construct the final target in Phase 1. The acquired CAISO report does not provide all fuel categories required for a defensible fuel-specific estimate, and eGRID/EIA are annual/reference sources rather than hourly marginal-emissions data. A future target may be an estimated operational average or another explicitly defined estimate, but Phase 1 does not choose between those constructs.

Measured emissions, annual emissions factors, hourly generation, average carbon intensity, and marginal carbon intensity must remain separate concepts. Forecasting experiments may begin with non-carbon features or a clearly labeled placeholder only if the target contract is not misrepresented.

## 10. Data-quality rules

**LOCKED:**

- Preserve missing values and reason codes; never silently impute raw data.
- Reject duplicate canonical instants except distinct, explicitly represented DST-fold instants.
- Preserve negative prices.
- Validate units before conversion and record conversions.
- Preserve quality, measurement, source, and report flags.
- Flag non-finite values, invalid calendar fields, impossible physical values, and source-inconsistent signs; do not automatically delete them.
- Do not interpolate incomplete CAISO production hours.
- Do not fabricate spring-forward intervals.
- Use chronological ordering and chronological evaluation splits.

**PROVISIONAL:** Apply variable-specific outlier checks only after source ranges and quality flags are fully characterized. Do not winsorize or delete values during raw/interim ingestion.

## 11. Provenance/versioning

**LOCKED:** Every acquired artifact and derived table must record, where available:

- provider, dataset/report, URL/API query, and access method;
- retrieval timestamp and provider publication/release timestamp;
- effective data interval and native resolution;
- release/revision/version and retrospective-revision status;
- checksum of the raw artifact;
- source timezone and interval semantics;
- transformations, aggregation rules, unit conversions, and QC results;
- code/configuration version and input manifest identifiers.

If a provider does not expose a value, record that limitation rather than inventing it. Manifests belong in `metadata/provenance/`; secrets never belong there. Raw/interim/processed datasets remain gitignored.

**PROVISIONAL:** Treat each provider revision as a distinct input version. A canonical dataset must identify which revision family it uses, even if the project later chooses a “latest available” policy.

## 12. Feature-availability/leakage rules

**LOCKED:** For a forecast origin at time `t`, a feature may use only information that would have been available by `t` under the documented publication and revision policy. This includes weather reports, CAISO observations, OASIS prices, imputed values, revised historical observations, and derived carbon estimates.

Historical corrected values must not be treated as real-time-known values unless the experiment explicitly models that retrospective availability. Late-arriving values must retain their publication/availability metadata. Feature engineering must be performed after chronological splitting when the transformation could use future observations.

## 13. Intended study period

**LOCKED as intent, not coverage guarantee:** 2020–2024 is the initial study period. Acquisition validation must verify actual coverage, gaps, product changes, source revisions, and release differences before final inclusion.

## 14. Train/validation/test split policy

**LOCKED:** Splits must be chronological and must not randomly divide rows. The final date boundaries are **UNRESOLVED** until coverage, revisions, and target availability are validated. Any published experiment must state the exact date ranges, feature-availability cutoff, source versions, and exclusion windows.

## 15. Known limitations

- CAISO EMS `HR` interval-start/interval-end and DST semantics are not fully established from the current sample.
- The production report is five-minute, raw, and aggregate; it does not cleanly support all desired fuel fields.
- A single OASIS price may not represent the entire CAISO scope.
- GHCNh station observations are irregular and spatially incomplete; no station set is yet justified.
- eGRID and EIA cannot independently produce hourly marginal emissions.
- Source revisions and publication delays may limit historical real-time availability reconstruction.
- No environmental, financial, or scheduling benefit has been demonstrated.

## 16. Remaining research questions

Before implementation is treated as complete, Verde must resolve:

1. CAISO load interval semantics, scope definition, and DST fold encoding.
2. Production transition-day behavior and confirmation of complete-block aggregation on real DST days.
3. OASIS location/product selection and geographic matching.
4. NOAA station set, local-time mapping, hourly aggregation, and quality thresholds.
5. Carbon target type: estimated operational average or another explicitly justified construct.
6. EPA/eGRID and EIA release selection, factor precedence, and emissions scope.
7. Holiday calendar, jurisdiction, and `day_of_week` numbering convention.
8. Revision/availability cutoff policy for the first forecasting dataset.
9. Final chronological split dates and any source-specific exclusion windows.
