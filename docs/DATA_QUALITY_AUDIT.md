# Data Quality Audit — Bronze Layer

This document records the data quality findings from the Bronze layer (`nyc_taxi.bronze.raw_trips`) and the cleaning rules applied in the Silver layer.

**Audit date:** October 2026
**Data source:** NYC TLC Yellow Taxi Trip Records — April, May, June 2025
**Total rows audited:** 12,885,358

---

## Findings

### 1. Unexpected Data Outside Expected Date Range

| Month | Rows | Notes |
|---|---|---|
| 2009-01 | 1 | Historical artifact, almost certainly a data entry error |
| 2025-03 | 4 | Outside our target range |
| 2025-04 | 3,970,566 | ✅ Target |
| 2025-05 | 4,591,844 | ✅ Target |
| 2025-06 | 4,322,943 | ✅ Target |

**Action:** Filter Silver to `pickup_datetime` between `2025-04-01` and `2025-07-01` (exclusive). Removes 5 rows.

---

### 2. Missing Values

3,154,852 rows (~24.5%) have nulls in **exactly these 5 columns simultaneously**:

| Column | Null Count |
|---|---|
| `passenger_count` | 3,154,852 |
| `RatecodeID` | 3,154,852 |
| `store_and_fwd_flag` | 3,154,852 |
| `congestion_surcharge` | 3,154,852 |
| `Airport_fee` | 3,154,852 |

The fact that all five are null on the *same* rows suggests this is a vendor reporting gap — certain vendors don't populate these fields.

**Action:**
- `congestion_surcharge` → coalesce to `0.0` (no fee charged)
- `Airport_fee` → coalesce to `0.0` (not at airport)
- `store_and_fwd_flag` → coalesce to `'N'` (not stored-and-forwarded)
- `passenger_count` → **keep null**, flag as `has_passenger_count = false`
- `RatecodeID` → **keep null**, or coalesce to `99` (Unknown)

---

### 3. Invalid Data

| Issue | Rows | % of Total |
|---|---|---|
| Dropoff ≤ pickup (invalid duration) | 167,624 | 1.30% |
| Passenger count ≤ 0 | 69,859 | 0.54% |
| Trip distance ≤ 0 | 367,456 | 2.85% |
| Fare amount ≤ 0 | 793,073 | 6.16% |
| Total amount ≤ 0 | 261,174 | 2.03% |

**Actions:**
- Invalid dates → **filter out** (physically impossible)
- Trip distance ≤ 0 → **filter out** (physically impossible)
- Fare amount < 0 → **filter out** (negative revenue impossible)
- Total amount < 0 → **filter out**
- Passenger count = 0 → **keep, flag** with `has_valid_passenger_count = false` (may be legit — e.g. package delivery)

---

### 4. Duplicates

Repeated rows appear on `(VendorID, pickup_datetime, dropoff_datetime, PULocationID, DOLocationID, fare_amount)`. Sample inspection shows:
- All from `VendorID = 2`
- All with `fare_amount = 0`
- Likely cancelled or disputed trips logged twice

**Action:** Deduplicate using `ROW_NUMBER() OVER (PARTITION BY ... ORDER BY pickup_datetime)`, keep first.

---

## Silver Cleaning Strategy

Ordered steps applied in the Silver notebook:

1. **Filter to target window** — pickup between `2025-04-01` and `2025-07-01`
2. **Remove impossible records** — invalid dates, zero/negative distance, negative fares
3. **Deduplicate** — `ROW_NUMBER()` partition by trip identity
4. **Fill nulls with defaults** — fees to 0.0, flags to 'N'
5. **Flag rather than drop ambiguous records** — `passenger_count = 0` kept with a flag
6. **Add derived columns** — trip duration, tip %, pickup hour, day of week
7. **Add quality flag** — `is_valid_trip = true` (reserved for future use)

## Expected Silver Row Count

~11.5M rows after cleaning (approximately 1.4M filtered out).

## Audit Reproducibility

To reproduce this audit:
1. Open `notebooks/02_silver_audit.ipynb`
2. Run all cells
3. Compare results with the tables above