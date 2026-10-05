# Verde data source specification

This document records the current data acquisition plan for Verde's California/CAISO data foundation. It is a specification, not an assertion that the sources have already been acquired or that every source can support the proposed variables without transformation.

## Common scope

- **Geography:** CAISO / California.
- **Intended initial study period:** 2020–2024. Acquisition inspection must verify actual coverage, gaps, product/report changes, source-specific availability, and release/version differences before final inclusion rules are set.
- **Canonical resolution:** hourly.
- **Canonical timezone:** `America/Los_Angeles`.
- **Budget:** zero-dollar by default; use public access methods only.
- **Primary target:** `estimated_carbon_intensity_kg_mwh`.
- **Secondary target:** `electricity_price_usd_mwh`.
- **Forecast horizon:** 24 hours.

Raw data must preserve the provider's native resolution and timestamp representation. The canonical Verde dataset is hourly. Any aggregation or resampling from sub-hourly source data must be explicitly documented; transformation metadata must record the source-native interval and aggregation rule. Source timestamps must be retained in provenance records and normalized to the canonical timezone only in interim/processed data. Daylight-saving transitions must not be silently dropped, duplicated, or interpolated.

## Source inventory

| Source | Provider | Dataset/report | Intended variables | Expected resolution | Relevant coverage | Access method | Licensing / redistribution | Open questions |
|---|---|---|---|---|---|---|---|---|
| CAISO load | California Independent System Operator (CAISO) | Historical EMS hourly load dataset/report | `demand_mw` | Intended source-native hourly; verify report representation during acquisition | Intended study period 2020–2024; coverage to be verified | CAISO public data downloads or APIs; exact endpoint/report identifier to be verified | Confirm current terms, attribution, redistribution, and historical endpoint stability before publication | Exact load definition, balancing-area scope, treatment of revisions, and any required transformation |
| CAISO production/resource | CAISO | Historical production/resource data | `solar_generation_mw`, `wind_generation_mw`, `hydro_generation_mw`, `geothermal_generation_mw`, `biomass_generation_mw`, `natural_gas_generation_mw`, `imports_mw`, `other_generation_mw` | Hourly or sub-hourly; native interval and any aggregation to be verified/documented | Intended study period 2020–2024; coverage to be verified | CAISO public data downloads or APIs | Confirm terms and whether derived/aggregated series may be redistributed | Resource categories, category mapping, missing/unknown categories, negative values, imports treatment, and reconciliation to load |
| CAISO OASIS | CAISO | OASIS market data | `electricity_price_usd_mwh` | Market interval to be selected and aggregated to hourly | Intended study period 2020–2024; coverage and node/hub availability to be verified | Public OASIS query/API | Confirm OASIS usage limits, attribution, and redistribution of downloaded/derived data | Price node or hub is intentionally not selected yet; product, market, timezone, and aggregation rules remain open |
| Weather | NOAA | GHCNh | `temperature_c`, `wind_speed_mps`, `wind_direction_deg`, `precipitation_mm` | Station observations may be sub-hourly/daily; canonical hourly representation is not yet specified | Intended study period 2020–2024; coverage subject to station selection and verification | NOAA public access/download services | Review NOAA data policy and source attribution requirements; preserve source identifiers and release/version metadata | Station selection, spatial aggregation/representation, variable availability, quality flags, wind-direction convention, precipitation accumulation semantics, and conversion rules |
| Emissions information | U.S. EPA | eGRID | Emissions information supporting carbon-signal construction | Typically annual plant/generator or subregion information; not an hourly signal | Relevant release for the intended 2020–2024 study period; release year to be selected | EPA public downloads | Confirm EPA terms, citation, release version, and redistribution conditions | Which eGRID release/year, geographic mapping, factors used, and how it contributes to the estimated operational signal |
| Fuel emissions factors | U.S. Energy Information Administration (EIA) | Fuel-specific emissions factors | Emissions factors supporting carbon-signal construction | Usually annual/fuel-specific rather than hourly | Relevant release for the intended 2020–2024 study period; release year to be selected | EIA public data/download services | Confirm API/download terms, attribution, and release/version requirements | Which factors, units, lifecycle/scope assumptions, and precedence relative to EPA information |

## Deliberately unspecified decisions

The following decisions must be resolved in a later research/data-design step and must not be inferred by an ingestion implementation:

- NOAA station set and any spatial aggregation method.
- CAISO OASIS price node or hub, product, market, and price aggregation rule.
- Exact operational carbon-intensity construction methodology.
- Emissions-factor source precedence and mapping from generation categories to factors.
- Treatment of data revisions, missing observations, outliers, curtailment, imports, and negative prices.
- Final train/validation/test date boundaries.
- `day_of_week` numbering convention.
- Holiday calendar and jurisdiction for `is_holiday`.
- NOAA wind-direction convention.
- NOAA precipitation accumulation semantics.
- Source-specific quality-flag handling.

If a source artifact cannot legally or practically be redistributed in the GitHub repository, Verde should retain the source name, URL/access instructions, retrieval instructions, version/release information, query parameters where applicable, retrieval timestamp, checksums where possible, and transformation/provenance metadata while keeping the raw artifact outside the repository.

Until these decisions are documented, the carbon field must be labeled an **estimated operational carbon-intensity signal**, not measured hourly marginal carbon intensity.
