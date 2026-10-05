# Verde data provenance and reproducibility requirements

Every acquired source and every generated dataset must be reproducible to the extent allowed by the provider. No raw data is currently included in this repository.

## Acquisition manifest

For each download or API extraction, record a manifest in `metadata/provenance/` when it contains non-secret reproducibility metadata. Manifests are tracked when appropriate. Raw, interim, and processed datasets remain gitignored, and secrets or API credentials must never be stored in manifests. Each manifest must contain at least:

- provider and dataset/report name;
- source URL or API endpoint and query parameters (excluding secrets);
- access timestamp in UTC;
- retrieval timestamp in UTC;
- provider publication timestamp, when available;
- effective data timestamp or interval represented by the artifact;
- whether the artifact represents a retrospective revision;
- data-availability cutoff used for modeling;
- provider publication/release date or version, when available;
- requested geographic scope, time range, and resolution;
- original filename or response identifier;
- byte size and SHA-256 checksum of the raw artifact;
- original file format and encoding;
- source timezone and timestamp semantics;
- license/terms and required attribution;
- transformation script/version used afterward; and
- notes about revisions, failures, missing data, or manual intervention.

If a provider does not expose a publication timestamp, effective interval, revision indicator, or availability cutoff, record that limitation rather than inventing a value.

## Transformation records

Each interim or processed dataset must identify:

- input artifact checksums or manifest IDs;
- code revision used for transformation;
- configuration and parameter values;
- unit conversions;
- timezone and daylight-saving handling;
- filtering, joins, aggregation, and missing-data treatment;
- quality-control checks and their results; and
- output checksum and row/time-range summary.

The manifest or transformation record must also identify the source-native interval and any aggregation or resampling rule used to produce hourly data.

## Time-series split policy

Any modeling dataset must publish explicit chronological train, validation, and test date ranges. Random row-level splitting is prohibited because it can leak temporal information across evaluation boundaries.

## Carbon-signal provenance

The primary target is currently an **estimated operational carbon-intensity signal**. Until its construction is finalized, every derived target must document its inputs, emissions factors, geographic mapping, assumptions, units, and uncertainty/limitations. It must not be described as measured hourly marginal carbon intensity.

## Reproduction checklist

Before an experiment is treated as reproducible, verify that another researcher can determine:

- which source releases and artifacts were used;
- how to access the same public inputs within provider constraints;
- which transformations produced the canonical table;
- which unresolved assumptions were fixed for the experiment;
- which chronological splits and random seeds were used; and
- which software and hardware environment produced the result.

If a source artifact cannot legally or practically be redistributed, retain the source name, URL/access instructions, retrieval instructions, release/version information, query parameters where applicable, retrieval timestamp, checksum where possible, and transformation/provenance metadata in `metadata/provenance/`; keep the raw artifact outside the repository.
