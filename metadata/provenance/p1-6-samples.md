# P1.6 sample acquisition manifest

These records describe the small representative samples acquired for Phase 1.6 reconnaissance on 2026-10-05. Raw artifacts remain under gitignored `data/raw/` paths. No credentials were used.

| Source | Artifact | Access method | Effective interval | Native resolution / format | Release/version | SHA-256 |
|---|---|---|---|---|---|---|
| CAISO EMS load | `data/raw/caiso_load/historicalemshourlyloadfor2023.xlsx` | Official CAISO Library document URL: `https://www.caiso.com/documents/historicalemshourlyloadfor2023.xlsx` | 2023 calendar year | Hourly workbook / XLSX | Posted 2024-05-07; report title “Historical EMS Hourly Load for 2023” | `f15e3453b0ec374ee081ad013ce50e3b0811bc53439e12b64061094c1e149e3c` |
| CAISO production/resource | `data/raw/caiso_production/productionandcurtailmentsdata_2023.xlsx` | Official CAISO Library document URL: `https://www.caiso.com/documents/productionandcurtailmentsdata_2023.xlsx` | 2023 calendar year | Five-minute workbook / XLSX | Posted 2024-01-19; 2023 production and curtailments report | `5fb479a724157e0889787065d45f1f3de262c5347600b132151a6515397ee65f` |
| CAISO OASIS | `data/raw/caiso_oasis/oasis_sample.zip` | Public OASIS `SingleZip` query; one-hour exploratory DAM, all-apnode group, 2024-01-15 08:00–09:00 UTC | One requested operating interval | ZIP containing CSV; one row per node/resource result | API `version=12`; query/report details retained below | `fbde8f1c8d47f3a9f4ade61ca9f0340683235b9a0cae440d59b92c13f846af32` |
| NOAA GHCNh | `data/raw/noaa_ghcnh/GHCNh_USW00023174_2023.psv` and `ghcnh-station-list.txt` | Official NCEI GHCNh bulk-download URLs | 2023; LAX candidate station only for reconnaissance | PSV observations plus station list; observations are not hourly-only | GHCNh station-period file; current NCEI product | `bf86b0c6ce7fd2129bdcd59bdc4d88d3c90ab5aecc8a9a3d1a993fc550161b4b` (observation file); `ff451fafc8f251560029e9ed31ebf0f4d12a2ddd492516c0b166c3744cad182a` (station list) |
| EPA eGRID | `data/raw/epa_egrid/egrid2023_data_rev2.xlsx` | Official EPA detailed-data download URL: `https://www.epa.gov/system/files/documents/2025-06/egrid2023_data_rev2.xlsx` | 2023 eGRID reporting year | Multi-sheet XLSX; annual plant/generator/subregion data | eGRID2023 Revision 2, released 2025-06-12 | `895cd81dd8662406189ad8adf5a2578dcb362cdb5cff7cca81b10ee6bd2447c6` |
| EIA fuel factors | `data/raw/eia_emissions_factors/epa_a_03.xlsx` | Official EIA Electric Power Annual Table A.3 XLSX URL: `https://www.eia.gov/electricity/annual/xls/epa_a_03.xlsx` | Current table; factor reference rather than time series | XLSX table | Electric Power Annual Table A.3, 2024 data release context | `88c53efe7b0ad0e35d290b938d96914f4621dccff363a94665b14147a133d934` |

## OASIS query record

```text
https://oasis.caiso.com/oasisapi/SingleZip
?queryname=PRC_LMP
&startdatetime=20240115T08:00-0000
&enddatetime=20240115T09:00-0000
&version=12
&market_run_id=DAM
&grp_type=ALL_APNODES
&resultformat=6
```

This query was used only to inspect report structure and available fields. It does not select a final node/hub, market, product, or price aggregation methodology.

## Limitations

Provider publication timestamps and revision status were recorded where available from the official source pages. Exact data-availability cutoffs were not exposed by every source; that limitation remains part of the acquisition record. No transformations were applied to the raw artifacts.
