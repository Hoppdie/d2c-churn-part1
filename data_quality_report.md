# Data Quality Report
## D2C Customer Churn Capstone — Part 1

**Prepared by:** [Your Name]  
**Snapshot Date:** 2025-09-30  
**Datasets Audited:** customers, orders, support_tickets, web_events_snapshot, churn_labels, rfm_modeling_snapshot, intervention_history

---

## 1. Missing Values

| Dataset | Column | Missing Count | Missing % | Impact | Recommended Treatment |
|---|---|---|---|---|---|
| `customers.csv` | `loyalty_tier` | ~1,386 | ~57.8% | High — signals non-enrolment, strongly correlated with churn | **Do not impute.** Create a binary flag `loyalty_enrolled` (0/1) and keep nulls as a distinct category "Not Enrolled" |
| `customers.csv` | `skin_type` | ~401 | ~16.7% | Medium — useful for product personalisation but not a direct churn driver | Impute with mode or treat null as a separate category "Not Provided" |
| `orders.csv` | `rating` | ~80 | ~0.8% | Low — small proportion; affects average rating calculations | Exclude from rating averages using `dropna()`; do not impute artificially |

**Key insight:** The `loyalty_tier` nulls are not random — they represent customers who were never enrolled in the loyalty programme. Analysis shows these customers have a higher churn rate than enrolled customers. Treating these nulls as "missing" rather than "not enrolled" would introduce a modelling error.

---

## 2. Duplicate / Duplicate-Like Records

| Dataset | Issue | Count | Detail | Recommended Treatment |
|---|---|---|---|---|
| `orders.csv` | `_DUP` suffix order IDs | Present | Intentional simulation of real-world deduplication challenges | **Remove before analysis and modelling.** Filter: `orders[~orders['order_id'].str.endswith('_DUP')]` |
| All datasets | Exact row duplicates | 0 | No exact duplicate rows found across any dataset | No action needed |

---

## 3. Outlier Values

| Dataset | Column | Issue | Detail | Recommended Treatment |
|---|---|---|---|---|
| `orders.csv` | `gross_amount` | Extreme high values | Max value ₹24,789 vs median ~₹500–800; values above 99th percentile are likely data entry errors or bulk orders | Cap at 99th percentile for feature engineering; flag for business review |
| `orders.csv` | `discount_pct` | Up to 70% discount | Values of 0.7 (70%) are unusually high for a personal-care brand | Investigate whether these are clearance sales or data errors; keep but flag |
| `support_tickets.csv` | `resolution_hours` | Up to 74.6 hours | Near 3-day resolution times indicate either complex issues or SLA breaches | Retain as-is; useful signal for churn risk |

---

## 4. Post-Snapshot Leakage Risk

| Dataset | Column | Issue | Detail |
|---|---|---|---|
| `orders.csv` | `order_date` | Contains post-snapshot rows | Orders dated after `2025-09-30` exist **only** to construct churn labels. These rows **must not** be used as model features. Filter strictly: `orders[orders['order_date'] <= '2025-09-30']` |
| `rfm_modeling_snapshot.csv` | `churn_next_60d` | Target variable included in feature table | This column must be excluded from all model inputs. It is the label, not a feature. |

> ⚠️ **Critical:** Any feature derived from post-snapshot order data (e.g., order counts after 2025-09-30) would constitute target leakage and lead to artificially inflated model performance and major mark deductions.

---

## 5. Join / Key Issues

| Join | Expected Universe | Actual Match | Issue |
|---|---|---|---|
| `customers` → `orders` | 2,400 customers | Not all customers have orders | Expected — new customers may have no order history |
| `customers` → `support_tickets` | 2,400 customers | ~1,921 unique tickets; not all customers have tickets | Expected — not every customer raises a support ticket |
| `customers` → `web_events_snapshot` | 2,400 customers | 2,400 rows | Perfect 1:1 join ✓ |
| `customers` → `churn_labels` | 2,400 customers | 2,400 rows | Perfect 1:1 join ✓ |
| `customers` → `rfm_modeling_snapshot` | 2,400 customers | 2,400 rows | Perfect 1:1 join ✓ |
| `customers` → `intervention_history` | 2,400 customers | 2,400 rows | Perfect 1:1 join ✓ |

**Recommendation:** Always use a **left join from `customers`** as the base. This ensures all 2,400 customers are retained. Customers with no orders or tickets will have `NaN` in joined columns — fill these with 0 for count/rate features.

---

## 6. Date Consistency

| Check | Result | Detail |
|---|---|---|
| `customers.signup_date` > snapshot | 0 violations | All signups are on or before 2025-09-30 ✓ |
| `support_tickets.ticket_date` > snapshot | 0 violations | All tickets are on or before 2025-09-30 ✓ |
| `orders.order_date` range | 2024-01-09 to 2025-11-29 | Post-snapshot rows intentionally present for label construction |
| `web_events_snapshot.snapshot_date` | All = 2025-09-30 | Consistent ✓ |

---

## 7. Columns That May Cause Leakage If Used Incorrectly

| Column | File | Risk | Safe to Use? |
|---|---|---|---|
| `churn_next_60d` | `rfm_modeling_snapshot.csv`, `churn_labels.csv` | **Direct target leakage** — this IS the label | ❌ Never as a feature |
| `order_date > 2025-09-30` | `orders.csv` | **Temporal leakage** — future purchases used to predict churn | ❌ Filter out before feature creation |
| `split` | `churn_labels.csv`, `rfm_modeling_snapshot.csv` | No leakage risk but must not be used as a feature | ✅ Use only for train/val/test splitting |
| All columns in `rfm_modeling_snapshot.csv` (except target/split) | `rfm_modeling_snapshot.csv` | Pre-built, leakage-free — all derived from pre-snapshot data | ✅ Safe to use |

---

## 8. Summary & Recommendations

| Priority | Action |
|---|---|
| 🔴 Critical | Filter `orders.csv` to `order_date <= 2025-09-30` before any feature engineering |
| 🔴 Critical | Remove `_DUP` order records before analysis |
| 🔴 Critical | Never use `churn_next_60d` as a model feature |
| 🟡 High | Treat `loyalty_tier` nulls as "Not Enrolled" — do not impute with mode |
| 🟡 High | Cap `gross_amount` outliers at 99th percentile for robust feature computation |
| 🟢 Medium | Fill missing `rating` values with `NaN` exclusion in aggregations |
| 🟢 Medium | Use left join from `customers` as the base for all merges; fill missing ticket/order counts with 0 |
