# Data Quality Report
## D2C Customer Churn Capstone — Part 1

**Snapshot Date:** 2025-09-30  
**Datasets Audited:** customers, orders, support_tickets, web_events_snapshot, churn_labels, rfm_modeling_snapshot, intervention_history  
**Universe:** 2,400 customers

---

## 1. Missing Values

| Dataset | Column | Missing Count | Missing % | Impact | Recommended Treatment |
|---|---|---|---|---|---|
| `customers.csv` | `loyalty_tier` | 1,386 | 57.8% | High — signals non-enrolment | Do **not** impute. Create binary flag `loyalty_enrolled` and keep nulls as category "Not Enrolled" |
| `customers.csv` | `skin_type` | 401 | 16.7% | Medium — personalisation field | Treat null as separate "Not Provided" category |
| `orders.csv` | `rating` | 80 (raw) / 58 (pre-snapshot) | 0.8% / 0.7% | Low | Exclude from rating averages with `dropna()`; do not impute |
| `rfm_modeling_snapshot.csv` | `loyalty_tier` | 1,386 | 57.8% | Same as customers | Same treatment |

**Key insight:** The `loyalty_tier` nulls are not random — they mark customers never enrolled in the loyalty programme. However, our EDA found these non-enrolled customers churn at 48.3%, which is **not** meaningfully higher than enrolled Silver members (48.8%). Only Gold (40.8%) and Platinum (37.1%) members churn notably less. So loyalty enrolment alone is a weak churn signal — tier level matters more than enrolment status.

**Notable:** Unrated orders have a higher return rate (10.3%) than rated orders (6.5%), suggesting customers who don't leave ratings are more likely to have had a return — a small but real data-quality signal worth noting.

---

## 2. Duplicate / Duplicate-Like Records

| Dataset | Issue | Count | Detail | Treatment |
|---|---|---|---|---|
| `orders.csv` | `_DUP` suffix order IDs | 12 | Intentional duplicate-like records | **Removed before analysis.** Filter `~order_id.str.endswith('_DUP')` |
| All datasets | Exact duplicate rows | 0 | None found | No action needed |

Raw orders: 10,009 → Clean orders (after removing 12 `_DUP`): **9,997**

---

## 3. Outlier Values

| Dataset | Column | Detail | Treatment |
|---|---|---|---|
| `orders.csv` | `gross_amount` | Max ₹24,789 vs 99th percentile ₹2,343. 82 orders (1.0%) above p99 | Cap at 99th percentile for feature engineering; flag for business review |
| `orders.csv` | `discount_pct` | Values up to 0.70 (70% off) | Investigate clearance vs error; retain but flag |
| `support_tickets.csv` | `resolution_hours` | Up to ~74.6 hours | Retain — useful churn signal |

---

## 4. Post-Snapshot Leakage Risk (CRITICAL)

| Dataset | Column | Detail |
|---|---|---|
| `orders.csv` | `order_date` | 1,869 orders dated after 2025-09-30 (range 2025-10-01 → 2025-11-29) exist **only** for label construction. **Must not** be used as features. |
| `rfm_modeling_snapshot.csv` | `churn_next_60d` | This is the target. Must be excluded from model inputs. |

After filtering to `order_date <= 2025-09-30`: **8,128 pre-snapshot orders** are safe to use as features.

> ⚠️ Any feature derived from the 1,869 post-snapshot orders would be target leakage and lead to major mark deductions.

---

## 5. Join / Key Issues

All joins are left joins from `customers` (universe = 2,400). Join coverage verified:

| Join | Coverage | Unmatched | Note |
|---|---|---|---|
| customers → orders | 100.0% | 0 | All order customer_ids exist in customers |
| customers → support_tickets | 100.0% | 0 | 1,153 customers (48.0%) have zero tickets — expected |
| customers → web_events_snapshot | 100.0% | 0 | Perfect 1:1 |
| customers → churn_labels | 100.0% | 0 | Perfect 1:1 |
| customers → rfm_modeling_snapshot | 100.0% | 0 | Perfect 1:1 |
| customers → intervention_history | 100.0% | 0 | Perfect 1:1 |

**Recommendation:** Use left join from `customers`. Fill missing ticket/order counts with 0 for customers with no history.

---

## 6. Date Consistency

| Check | Result |
|---|---|
| `customers.signup_date` > snapshot | 0 violations ✓ |
| `support_tickets.ticket_date` > snapshot | 0 violations ✓ |
| `orders.order_date` range | 2024 → 2025-11-29 (post-snapshot rows intentional) |
| `web_events_snapshot.snapshot_date` | All = 2025-09-30 ✓ |

---

## 7. Columns That May Cause Leakage If Used Incorrectly

| Column | File | Risk | Safe? |
|---|---|---|---|
| `churn_next_60d` | rfm_modeling_snapshot, churn_labels | Direct target | ❌ Never a feature |
| Post-snapshot `order_date` rows | orders | Temporal leakage | ❌ Filter out |
| `split` | churn_labels, rfm_modeling_snapshot | Splitting only | ✅ Not as feature |
| All other rfm_modeling_snapshot columns | rfm_modeling_snapshot | Pre-built, leakage-free | ✅ Safe |

---

## 8. Summary & Recommendations

| Priority | Action |
|---|---|
| 🔴 Critical | Filter orders to `order_date <= 2025-09-30` (drops 1,869 rows) |
| 🔴 Critical | Remove 12 `_DUP` order records |
| 🔴 Critical | Never use `churn_next_60d` as a feature |
| 🟡 High | Treat 1,386 `loyalty_tier` nulls as "Not Enrolled" |
| 🟡 High | Cap `gross_amount` at p99 (₹2,343) for robust features |
| 🟢 Medium | Exclude 58 null ratings from averages |
| 🟢 Medium | Fill missing ticket/order counts with 0 after left join |

**Dataset headline:** 2,400 customers, 47.0% overall churn rate (balanced classes), 8,128 usable pre-snapshot orders, 1,247 customers (52%) with at least one support ticket.
