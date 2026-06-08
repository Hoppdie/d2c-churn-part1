# Part 1 — Data Audit, EDA & Business Understanding

## Overview
This repository contains the complete Part 1 submission for the D2C Customer Churn Intelligence Capstone.

**Snapshot date:** `2025-09-30`  
**Target:** `churn_next_60d` — 1 if customer made no purchase between 2025-10-01 and 2025-11-29

---

## Repository Structure

```
├── README.md                    ← This file
├── eda_audit.ipynb              ← Main analysis notebook (run in Google Colab)
├── data_quality_report.md       ← Data quality issues and treatment recommendations
├── business_memo.md             ← Business-facing memo for leadership
├── requirements.txt             ← Python dependencies
```

---

## How to Run

### Option A — Google Colab (Recommended)

1. Open [Google Colab](https://colab.research.google.com)
2. Go to **File → Upload notebook** and upload `eda_audit.ipynb`
3. Download the dataset from the [Google Drive link](https://drive.google.com/drive/folders/1PmLapJI1VSDgvl_AxARNKwM1MCd3WFX0?usp=sharing)
4. Upload the CSV files to your own Google Drive folder (e.g., `My Drive/d2c_churn_data/`)
5. In the notebook's second cell, update `DATA_DIR` to match your folder path:
   ```python
   DATA_DIR = '/content/drive/MyDrive/d2c_churn_data/'
   ```
6. Run all cells top to bottom (**Runtime → Run all**)

### Option B — Local Jupyter

```bash
pip install -r requirements.txt
jupyter notebook eda_audit.ipynb
```

Update `DATA_DIR` in cell 2 to point to your local data folder.

---

## Dataset

All 7 CSV files are required:
- `customers.csv`
- `orders.csv`
- `support_tickets.csv`
- `web_events_snapshot.csv`
- `churn_labels.csv`
- `rfm_modeling_snapshot.csv`
- `intervention_history.csv`

Download from: https://drive.google.com/drive/folders/1PmLapJI1VSDgvl_AxARNKwM1MCd3WFX0?usp=sharing

---

## Key Outputs

The notebook produces:
- **8+ charts** saved as PNG files
- Full data quality audit with per-column analysis
- 5 evidenced churn-risk hypotheses
- Schema inspection and join coverage checks

---

## Churn-Risk Hypotheses Summary

| # | Hypothesis | Signal Used |
|---|---|---|
| H1 | High recency (91+ days inactive) → higher churn | `orders.order_date` |
| H2 | Not enrolled in loyalty programme → higher churn | `customers.loyalty_tier` |
| H3 | Low web sessions in last 30 days → higher churn | `web_events_snapshot.sessions_30d` |
| H4 | Negative / reopened support tickets → higher churn | `support_tickets.sentiment_score`, `reopened` |
| H5 | Very high discount usage → above-average churn risk | `orders.discount_pct` |

---

## Notes
- All analysis uses **only pre-snapshot data** (`order_date <= 2025-09-30`)
- `_DUP` order records are removed before all analysis
- `loyalty_tier` nulls are treated as "Not Enrolled", not imputed
