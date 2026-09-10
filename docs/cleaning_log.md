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