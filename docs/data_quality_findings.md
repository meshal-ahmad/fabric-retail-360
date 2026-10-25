# Data Quality Findings

Profiling was done on the raw file (153,308 rows, 23 columns) before any transformation.

| # | Finding | Evidence | Decision |
|---|---|---|---|
| 1 | No actual flight times | Only `scheduledTime` (UTC and local) exists | Delay analysis excluded ([ADR-002](adr/002-no-delay-analysis.md)) |
| 2 | `status` is mostly uninformative | Unknown 128,618 (84%), Expected 13,059, Departed 11,520, Canceled 78, CanceledUncertain 33 | Kept as descriptive only; derived `is_canceled` |
| 3 | `origin_*` columns are always RUH, even for arrivals | `origin_airport_iata` has a single distinct value: RUH | Dropped origin columns; renamed `destination_*` to `other_airport_*` (the airport on the other end of the flight) |
| 4 | Near-duplicate records | 2,789 flights appear twice with the same flight number, scheduled time and type but different field completeness (for example one row has the aircraft registration, the other does not) | Keep the most complete record, quarantine the rest ([ADR-001](adr/001-deduplication-strategy.md)) |
| 5 | Sparse aircraft data | `aircraft.reg` missing 117,074; `aircraft.modeS` 107,977; `callSign` 108,171 | `dim_aircraft` at model level only |
| 6 | Missing airport codes | 4,106 rows have an airport name but no IATA/ICAO/timezone | Enriched from a reference built from complete rows |
| 7 | Cities with several airports | Istanbul (IST, SAW), Beijing (PEK, PKX), Moscow (VKO, SVO), Paris (CDG, LBG) | Never guess: tagged `ambiguous_city`; `dim_airport` key includes the IATA code |
| 8 | Missing airline ICAO | 563 rows, 554 of them Air Sial | Left NULL (no reliable source), documented |
| 9 | Missing terminal | 850 rows | Filled with `Unknown` |
| 10 | Timestamps stored as strings | UTC like `2025-03-14 21:01Z`; local like `2025-03-15 00:01+03:00` | Parsed to timestamp; Riyadh time = UTC + 3 (no DST) |
| 11 | `movement.quality` is an array | Spark type array&lt;string&gt; | Converted to comma-separated string |
| 12 | Column names contain dots | `aircraft.model`, `movement.scheduledTime.utc`, ... | Replaced `.` with `_` in Bronze (technical change only) |

## Airport code enrichment results (`airport_code_source`)

| Source | Rows |
|---|---|
| original | 146,421 |
| reference (filled from complete rows) | 2,425 |
| manual_override | 724 |
| ambiguous_city | 548 |
| unresolved | 401 |

Manual overrides (verify against an authoritative source before relying on them): Neom Bay Airport = NUM, Kannur International Airport = CNN, Kozhikode = CCJ, Entebbe = EBB, Rize = RZV.

## Reconciliation

| Check | Result |
|---|---|
| Raw rows = Bronze rows | 153,308 = 153,308 |
| Silver + quarantine = Bronze | 150,519 + 2,789 = 153,308 |
| Failed timestamp parses | 0 |
| Fact rows = Silver rows | 150,519 = 150,519 |
| Fact rows with missing date, airline, aircraft or airport key | 0 |
