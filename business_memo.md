# Business Memo
## To: Product, Marketing & Customer Support Leadership
## From: Data Analytics Team
## Re: Key Findings Before Launching Retention Campaign
## Snapshot: 2025-09-30

---

### Executive Summary

We analysed 2,400 customers ahead of the planned retention campaign. The overall churn rate is **47.0%** — almost half the base did not purchase in the 60-day window, so this is a serious retention problem, not an edge case. Below are the five strongest patterns from the data and what they mean for campaign design. Several common assumptions (e.g. "discounts retain customers", "loyalty enrolment prevents churn") are **not** supported by our data and would waste budget if acted on blindly.

---

### 1. Inactivity Is by Far the Strongest Churn Signal

Churn rises steeply with days since last order:

| Days since last order | Churn rate | Customers |
|---|---|---|
| 0–30 days | 11.9% | 671 |
| 31–60 days | 30.4% | 441 |
| 61–90 days | 45.7% | 361 |
| 91–180 days | 78.7% | 597 |
| 180+ days | 91.4% | 302 |

This is the clearest, most actionable pattern in the entire dataset. The 61–90 day window is the inflection point — churn jumps from 46% to 79% right after it.

**Recommendation:** Prioritise customers in the **61–90 day** inactivity window for urgent outreach. They are still recoverable; once they pass 90 days, the odds drop sharply. Customers already past 180 days (91% churn) are largely unrecoverable — minimal spend there.

---

### 2. Loyalty Enrolment Does NOT Prevent Churn — Only High Tiers Help

A common assumption is that enrolling customers in the loyalty programme reduces churn. **The data does not support this:**

| Loyalty status | Churn rate | Customers |
|---|---|---|
| Silver | 48.8% | 590 |
| Not Enrolled | 48.3% | 1,386 |
| Gold | 40.8% | 319 |
| Platinum | 37.1% | 105 |

Silver members churn at essentially the same rate as non-enrolled customers. Only Gold and Platinum members churn meaningfully less — and that is likely because high-tier customers are already loyal, not because the tier caused loyalty.

**Recommendation:** Do not invest in a blanket "enrol in loyalty" push expecting it to reduce churn. Instead, investigate what drives customers to Gold/Platinum (frequency, spend) and encourage those behaviours directly.

---

### 3. Web / App Disengagement Strongly Predicts Churn

Web activity in the 30 days before snapshot tracks churn closely:

| Sessions in last 30 days | Churn rate | Customers |
|---|---|---|
| 0 sessions | 66.3% | 190 |
| 1–2 | 63.9% | 592 |
| 3–5 | 50.9% | 599 |
| 6–10 | 36.5% | 674 |
| 10+ | 20.9% | 345 |

Median churned customer had 3 sessions and last visited 26 days ago; median retained customer had 6 sessions and last visited 7 days ago.

**Recommendation:** Connect web activity to the CRM in real time. Any customer with 0–2 sessions in 30 days and no recent visit should enter an automated re-engagement flow immediately — this is a live early-warning signal, not a quarterly batch metric.

---

### 4. The CRM Team's Manual Priority Already Works — Use It

The existing `manual_priority_bucket` set by the CRM team is a remarkably strong predictor:

| Manual priority | Churn rate |
|---|---|
| high | 74.7% |
| medium | 27.9% |
| low | 10.0% |

The team's existing intuition is already capturing real risk. This is valuable institutional knowledge that should be combined with the model, not replaced by it.

**Recommendation:** Treat "high" priority customers as a primary target list. The fact that they churn at 75% confirms the team is identifying genuine risk.

---

### 5. Discounts Don't Retain — and Heavy Discounting Correlates with MORE Churn

Average discount usage shows almost no protective effect, and the heaviest-discount group churns the most:

| Avg discount bracket | Churn rate | Customers |
|---|---|---|
| <5% | 42.3% | 26 |
| 5–15% | 47.6% | 168 |
| 15–30% | 45.1% | 1,223 |
| 30–50% | 48.6% | 947 |
| >50% | 66.7% | 36 |

Churn is essentially flat (~45–49%) across most discount levels, then spikes to 67% for the heaviest discounters. This strongly suggests heavy-discount customers are deal-seekers who leave when the deals stop.

**Recommendation:** Do not default to discounts in the retention campaign. For heavy-discount customers, shift to non-discount value (loyalty points, free shipping, early access). Reserve discounts for high-value customers who have recently gone quiet.

---

### What to Investigate Before Launch

| Priority | Investigation |
|---|---|
| 1 | Identify all customers currently in the 61–90 day inactivity window — this is the recoverable cohort |
| 2 | Cross-reference the campaign target list with `manual_priority_bucket = high` |
| 3 | Set up real-time web-activity alerts for customers dropping below 3 sessions/30d |
| 4 | Audit which "high-value" customers have open or reopened support tickets before sending offers |
| 5 | Stop assuming discounts retain — pilot non-discount interventions on the heavy-discount group |

---

### Closing Note

Churn here is driven primarily by **disengagement** (low recency, low web activity), not by demographics or discount levels. The campaign will be most effective if it targets the intersection of high recency + low web activity + high manual-priority, rather than relying on loyalty enrolment or discounts, which our data shows do not move the needle.

*Based on 2,400 customers, 8,128 pre-snapshot orders, snapshot date 2025-09-30.*
