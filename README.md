# RUH Flights Data Platform on Microsoft Fabric

An end-to-end data platform built on **Microsoft Fabric** using the **Medallion architecture**, processing 153K flight movements (arrivals and departures) at **King Khalid International Airport (RUH / OERK), Riyadh**.

**Data scope:** 14 March 2025 - 10 October 2025 (about 7 months).

## Architecture

```
Raw Parquet -> Bronze (Delta) -> Silver (cleaned) -> Gold (Warehouse, star schema) -> Power BI
```

| Layer | Item | Rows |
|---|---|---|
| Raw | `Files/raw/ruh_flights/flights_RUH.parquet` (Lakehouse `lh_ruh_airport`) | 153,308 |
| Bronze | `bronze_flights` (Delta, untouched copy + tracking columns) | 153,308 |
| Silver | `silver_flights` (cleaned, deduplicated, enriched) | 150,519 |
| Silver | `quarantine_duplicates` (rejected duplicates, kept for audit) | 2,789 |
| Gold | `wh_ruh_flights`: `fact_flights` + 4 dimensions | 150,519 |

Row counts reconcile across layers: 150,519 + 2,789 = 153,308.

### Gold star schema

| Table | Rows |
|---|---|
| `gold.fact_flights` | 150,519 |
| `gold.dim_airline` | 68 |
| `gold.dim_aircraft` | 49 |
| `gold.dim_airport` | 135 |
| `gold.dim_date` | 210 |

Quality check on the fact table: 0 missing date, airline, aircraft or airport keys.

## Progress

- [x] Data profiling and quality findings
- [x] Bronze layer
- [x] Silver layer (cleaning, deduplication, enrichment)
- [x] Gold layer (Warehouse, star schema)
- [ ] Second source (weather API via Data Pipeline)
- [ ] Master pipeline (scheduling, parameters, error handling)
- [ ] Security (RLS / CLS)
- [ ] Real-time (Eventstream, Eventhouse / KQL, Activator)
- [ ] Performance tuning (OPTIMIZE, V-Order, before/after)
- [ ] Power BI (Direct Lake)

## Repository layout

```
notebooks/   Fabric notebooks (exported .ipynb)
sql/gold/    T-SQL scripts for the Gold layer, run in numeric order
docs/        Data quality findings and architecture decision records (adr/)
```

## Data source

King Khalid International Airport (RUH) Flights Dataset by Mohammed Alsubaie on Kaggle:
https://www.kaggle.com/datasets/mohammedalsubaie/king-khalid-international-airport-flights-dataset

License: Apache 2.0

According to its description, the dataset was compiled from several aviation APIs and stored as Parquet (153,308 records, 23 columns).
The raw file is **not** included in this repository. Download it from the link above and place it inohammed Alsubaie on Kaggleof the Lakehouse.
## Known limitations

- **No actual flight times.** Only scheduled times exist, so delay analysis is not possible. See [ADR-002](docs/adr/002-no-delay-analysis.md).
- **About 7 months of data.** No year-over-year comparison.
- **Aircraft details are sparse.** Registration is missing in about 76% of raw rows, so `dim_aircraft` is model-level only.
- **Some airport codes are unresolved.** 548 rows belong to multi-airport cities and 401 rows could not be resolved. See [data quality findings](docs/data_quality_findings.md).
- `airline_icao` is missing for 563 raw rows (554 are Air Sial).

## Lessons learned

- Fabric Warehouse does not support `nvarchar`. Cast `DATENAME(...)` results to `VARCHAR`.
- Renaming a Lakehouse after its SQL analytics endpoint exists broke metadata sync ("Database does not exist"). Name items correctly from the start.
- `CREATE SCHEMA` must run in its own batch.
- Native Fabric Git integration was unavailable because GitHub sync is disabled at the tenant level, so notebooks and SQL are versioned manually. See [ADR-004](docs/adr/004-manual-versioning.md).
