# Data Cleaning Log — NYC Airbnb Open Data

## Dataset Overview
- Source: Kaggle — New York City Airbnb Open Data (AB_NYC_2019.csv)
- Rows: 48,895 | Columns: 16
- Initial inspection: df.info(), df.isnull().sum()

---

## Decision Log

### 1. `last_review` and `reviews_per_month` — Missing Values
**Observation:** 10,052 rows (~20.5%) missing in both columns simultaneously.

**Investigation:** Verified via `df[df['last_review'].isnull()]['number_of_reviews'].unique()` — confirmed nulls correspond exclusively to listings with 0 reviews.

**Decision:** Treat as "no reviews yet," not missing data.
- `reviews_per_month`: filled with `0`
- `last_review`: left as null after converting to datetime (no valid date exists for a listing with zero reviews — filling with a fake date would misrepresent the data)

**Rationale:** These nulls carry real business meaning (never reviewed), not data quality issues. Imputing a fake value would distort any downstream analysis on review recency or frequency.

---

### 2. `name` and `host_name` — Missing Values
**Observation:** `name` missing in 16 rows (0.03%), `host_name` missing in 21 rows (0.04%).

**Investigation:** Inspected affected rows directly — no pattern of broader data corruption; all other fields (id, host_id, coordinates, neighbourhood) remain intact. Appears to be isolated cases where hosts left the listing title or their name blank.

**Decision:** Filled both with the placeholder `'Unknown'` rather than dropping rows.

**Rationale:** Affected percentage is negligible, but dropping would still discard otherwise complete, valid rows. `name` and `host_name` are identifier/text fields not used in numeric analysis, so a placeholder has no downstream statistical impact.

---

### 3. Duplicate Rows
**Observation:** Checked via `df.duplicated().sum()` → returned 0.

**Decision:** No action needed.

**Rationale:** Dataset contains no exact duplicate rows across all columns.

---

### 4. `price` = 0 — Outliers
**Observation:** 11 rows (0.02%) had price = 0.

**Investigation:** Inspected affected rows — all had legitimate neighbourhoods, room types, and active review history (some with dozens of reviews and recent last_review dates). Not corrupted or test entries.

**Decision:** Dropped these 11 rows.

**Rationale:** A price of 0 is factually invalid (Airbnb listings cannot be free), and with no reliable basis to impute a correct price, dropping is safer than introducing a guessed value into price-based analysis. Negligible row loss (0.02%) makes this a low-risk decision.

---

### 5. `minimum_nights` — Extreme Outliers
**Observation:** 14 rows (0.03%) had minimum_nights > 365 (up to 1,250 nights).

**Investigation:** Cross-checked against availability_365 to test whether hosts were intentionally blocking bookings (a known Airbnb pattern). Found most affected listings had high availability_365 (300+ days), contradicting the "intentional delisting" theory — this pattern instead looks like unrealistic/erroneous minimum stay values rather than deliberate host behavior.

**Decision:** Dropped these 14 rows.

**Rationale:** No genuine short/mid-term rental requires 500+ night minimum stays. With no reliable way to determine a correct value, and given the negligible row loss (0.03%), dropping is the safer choice over imputing a guessed value.