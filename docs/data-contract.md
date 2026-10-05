# Verde dataset schema and data contract

This contract defines the intended canonical hourly table. It does not implement ingestion and does not finalize the carbon-intensity methodology.

## Canonical record

Each row represents one canonical hourly interval for the CAISO/California scope. The canonical time key is `timestamp` with timezone `America/Los_Angeles`. A timestamp identifies the start of the interval unless a future source-specific contract explicitly states otherwise.

| Field | Type | Unit / allowed values | Required | Role |
|---|---|---|---|---|
| `timestamp` | timezone-aware datetime | `America/Los_Angeles` | Yes | Hourly interval key |
| `demand_mw` | float | MW | No | Grid feature |
| `solar_generation_mw` | float | MW | No | Grid feature |
| `wind_generation_mw` | float | MW | No | Grid feature |
| `hydro_generation_mw` | float | MW | No | Grid feature |
| `geothermal_generation_mw` | float | MW | No | Grid feature |
| `biomass_generation_mw` | float | MW | No | Grid feature |
| `natural_gas_generation_mw` | float | MW | No | Grid feature |
| `imports_mw` | float | MW | No | Grid feature |
| `other_generation_mw` | float | MW | No | Grid feature |
| `electricity_price_usd_mwh` | float | USD/MWh | No | Secondary target / market feature |
| `temperature_c` | float | °C | No | Weather feature; station/aggregation TBD |
| `wind_speed_mps` | float | m/s | No | Weather feature; station/aggregation TBD |
| `wind_direction_deg` | float | degrees, convention TBD | No | Weather feature; station/aggregation TBD |
| `precipitation_mm` | float | mm over source interval; aggregation TBD | No | Weather feature |
| `hour` | integer | 0–23, canonical local hour | Yes once derived | Temporal feature |
| `day_of_week` | integer | 0–6; convention must be fixed in code/docs | Yes once derived | Temporal feature |
| `day_of_year` | integer | 1–366 | Yes once derived | Temporal feature |
| `month` | integer | 1–12 | Yes once derived | Temporal feature |
| `is_weekend` | boolean | true/false | Yes once derived | Temporal feature |
| `is_holiday` | boolean or nullable | true/false/unknown | Yes once holiday calendar is selected | Temporal feature |
| `estimated_carbon_intensity_kg_mwh` | float | Declared unit, including emissions scope, required when constructed | Derived target; unavailable until methodology is finalized | Primary forecasting target |

## Field provenance classes

Canonical field names do not imply that a provider publishes a field under the same name. Each field must be classified in dataset metadata as one of:

- **Source-native observation:** retained from a provider with its native semantics and resolution.
- **Source-derived/normalized observation:** created by unit conversion, timestamp normalization, category mapping, or aggregation of source data. For example, CAISO resource categories may require mapping or aggregation before becoming canonical generation fields.
- **Derived modeling feature:** calculated locally from canonical observations or timestamps. Temporal fields are derived locally; future cyclical encodings are also derived features if introduced.
- **Derived target variable:** calculated from documented source inputs and methodology. `estimated_carbon_intensity_kg_mwh` is a derived target.

The carbon target must ultimately declare an emissions scope, unit, geographic boundary, construction methodology, and methodology/version identifier. Its construction remains unresolved and it must not be described as measured marginal carbon intensity.

## Integrity rules

1. `timestamp` must be unique and sorted ascending in canonical tables.
2. Canonical records must use hourly intervals; daylight-saving transitions must be represented explicitly and validated.
3. No random train/test split is permitted. Evaluation splits must be chronological and documented by date.
4. Missing values must remain missing or be handled by a documented, source-appropriate rule. They must not be silently filled.
5. Every numeric field must retain its source unit and a documented conversion if converted.
6. Values derived from multiple sources must be traceable to those sources and their versions.
7. Raw data must remain immutable. Transformations belong in interim or processed layers.
8. A processed dataset must not imply that the carbon signal is directly measured or marginal unless that methodology is separately established and documented.
9. Source-specific fields may exist in raw/interim layers, but canonical processed fields must use the names and semantics above.
10. For a forecast origin at time `t`, every feature must be limited to information that would have been available by `t` under the documented acquisition and revision policy. This includes weather observations, CAISO observations, market prices, derived carbon signals, imputed values, and revised historical values. Retrospectively revised or future observations must not be used as though they were available at forecast time.
11. Timestamps must remain timezone-aware; naive timestamps are invalid. Interval start/end semantics must be documented. Canonical timestamps use `America/Los_Angeles`. DST fold/repeated-hour behavior must be preserved and validated, and spring-forward gaps must not be silently fabricated. The representation of fold metadata remains a decision to verify during source inspection.
12. Local calendar fields such as `hour` and `day_of_week` must be derived from the canonical timezone-aware timestamp and must not replace it. UTC may be used internally for ordering or storage, but the canonical semantic timestamp remains explicit.

## Dataset layers

- **Raw source tables:** Provider-delivered files/responses, unchanged except for safe packaging and metadata, containing source-native fields. Not committed to Git.
- **Interim normalized tables:** Parsed, normalized, and source-specific tables retaining provenance, quality flags, source-native resolution, and documented transformations. Not committed to Git.
- **Processed feature tables:** Joined canonical hourly tables containing the features required for a relevant modeling task. Source-dependent fields may remain unavailable; missingness must be explicit and documented rather than invented. Not committed to Git.
- **Processed supervised-target/modeling tables:** Processed feature tables with a constructed target and its methodology metadata. Once constructed, `estimated_carbon_intensity_kg_mwh` is required for the primary supervised forecasting dataset. Not committed to Git.

The contract may be extended with provenance columns or sidecar manifests; such additions must not change the meaning of the canonical fields without a documented revision.
