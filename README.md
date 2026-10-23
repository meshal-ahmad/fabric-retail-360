# RUH Flights Data Platform on Microsoft Fabric

End-to-end data platform built on Microsoft Fabric using Medallion architecture
on 153K flight movements at King Khalid International Airport (RUH / OERK), Riyadh.

**Data scope:** arrivals and departures, 14 March 2025 - 10 October 2025.

## Architecture
Raw Parquet -> Bronze (Delta) -> Silver (cleaned) -> Gold (star schema) -> Power BI

## Progress
- [x] Data profiling
- [x] Bronze layer
- [x] Silver layer (cleaning, deduplication, enrichment)
- [ ] Gold layer (Warehouse, star schema)
- [ ] Second source (weather API)
- [ ] Real-time (Eventstream, Eventhouse, Activator)
- [ ] Orchestration, security, performance tuning
- [ ] Power BI (Direct Lake)

## Data source
Add the dataset link and license here. The raw file is not included in this repo.

## Known limitations
- No actual flight times, so delay analysis is not possible
- About 7 months of data, so no year-over-year comparison