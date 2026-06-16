#  NOAA Storm Damage Prediction

### ISOM 631 – Predictive Analytics and Machine Learning | Suffolk University MSBA | Spring 2026

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)](https://python.org)
[![XGBoost](https://img.shields.io/badge/XGBoost-Best_Model-FF6600?logo=xgboost)](https://xgboost.readthedocs.io)
[![NOAA](https://img.shields.io/badge/Dataset-NOAA_Storm_Events-0052A5)](https://www.ncei.noaa.gov/pub/data/swdi/stormevents/csvfiles/)
[![Stage 1 AUC](https://img.shields.io/badge/Stage_1_AUC-0.9705-2ca25e)](#model-results)
[![Stage 2 R²](https://img.shields.io/badge/Stage_2_R²-0.653-2ca25e)](#model-results)

---

## Project Summary

This project builds a **two-stage machine learning pipeline** to predict property damage from severe weather events recorded in the NOAA Storm Events Database. Given a storm's characteristics — type, magnitude, duration, location, and timing — the model first predicts **whether** the storm will cause any property damage, then, for the storms that do, predicts **how much**.

**Stage 1 (Classification):** XGBoost · **Accuracy = 92.2%** · **AUC = 0.9705**
**Stage 2 (Regression):** XGBoost · **R² = 0.653** · **RMSE = 1.2194** (log-damage scale)

The analysis covers 283,155 storm events with recorded damage outcomes (out of 409,280 raw records), spanning 2013–2020 across all 50 U.S. states, and was built for a simulated regional insurance company use case.

---

## Project Objectives

1. **Predict** whether an individual storm event will cause property damage, using only information available the moment NOAA logs the event.
2. **Estimate** the dollar amount of damage for storms predicted to be damaging.
3. **Solve** the zero-inflation problem inherent to catastrophe data — 73.7% of storm events cause zero damage, which breaks standard single-stage regression.
4. **Translate** model findings into deployment-ready recommendations for an insurer's claims, risk, and finance teams.

---

## Why Two Stages?

A single regression model trained on all 283,155 events would fail, because nearly three-quarters of storms cause no damage at all. A model trained on mostly zeros learns to predict numbers close to zero for everything, which is useless for the events that matter.

This project solves it the way real catastrophe (CAT) models do — split the problem in two:

| Stage | Question | Data Used | Model |
|---|---|---|---|
| **1 — Classification** | Will this storm cause *any* damage? | All 283,155 events | Logistic Regression → Random Forest → **XGBoost** |
| **2 — Regression** | If damage occurs, *how much*? | 74,514 damage-only events | Linear Regression → Random Forest → **XGBoost** |

---

## Dataset

| Property | Value |
|---|---|
| **Source** | [NOAA Storm Events Database](https://www.ncei.noaa.gov/pub/data/swdi/stormevents/csvfiles/) (National Centers for Environmental Information) |
| **Raw records** | 409,280 storm events |
| **Working dataset** | 283,155 events with a recorded damage value |
| **Time span** | 2013–2020 (7 years; 2016 file unavailable) |
| **Coverage** | All 50 U.S. states, 55+ storm event types |
| **Stage 1 target** | `damage_occurred` (0/1) |
| **Stage 2 target** | `log_damage` — log-transformed `damage_property`, fit only on the 74,514 events with damage > $0 |

> **Note:** Raw CSV files are not included in this repository. Source files (`Storms_2013.csv` … `Storms_2020.csv`) are available from the [NOAA NCEI Storm Events portal](https://www.ncei.noaa.gov/pub/data/swdi/stormevents/csvfiles/).

---

## Repository Structure

```
NOAA-Storm-Damage-Prediction/
│
├── visualizations/
│   ├── Target Variable Distribution.png
│   ├── Storm Type Frequency vs Severity.png
│   ├── Seasonality and Coastal Impact.png
│   ├── Correlation Heatmap - Damage Amount.png
│   ├── Correlation Heatmap - Damage Occurrence.png
│   ├── Stage 1 ROC Curve and Confusion Matrix.png
│   ├── Stage 2 Predicted vs Actual.png
│   ├── XGBoost Feature Importance.png
│   └── Final Results Dashboard.png
│
├── NOAA_Storm_Damage_Analysis.ipynb   ← Main analysis notebook
├── NOAA_Presentation_Group_6.pptx     ← Final presentation deck
├── README.md                          ← You are here
└── .gitignore
```

> The notebook also generates **three interactive Folium maps** (national damage heatmap, top-500 catastrophic events, and a New England/Boston regional view). These render live inside the notebook but are interactive HTML, not static images, so they aren't duplicated in `visualizations/` — open the notebook to explore them.

---

## Notebook Structure

| Part | Content | Key Output |
|---|---|---|
| 🔵 **Part 1 – The Situation** | Data loading, quality audit, leakage removal, target analysis | 283,155 valid records, zero-inflation identified (73.7%) |
| 🟡 **Part 2 – The Discovery** | Feature engineering, log-transform, 8 EDA questions, 3 Folium maps | Frequency-vs-severity gap, seasonality, coastal premium, correlation structure |
| 🟠 **Part 3 – The Model** | Two-stage pipeline: classification → regression, feature importance | Stage 1 AUC = 0.9705 · Stage 2 R² = 0.653 |
| 🟢 **Part 4 – The Recommendation** | Business interpretation, 4 deployment-ready recommendations | Same-day damage flagging, reserve estimation, exposure roadmap |

---

## Model Results

### Stage 1 — Classification (Will damage occur?)

| Model | Accuracy | AUC | Notes |
|---|---|---|---|
| Logistic Regression | 82.0% | 0.8482 | Baseline |
| Random Forest | 89.1% | 0.9503 | Strong improvement |
| **XGBoost ✅** | **92.2%** | **0.9705** | Final model |

**Confusion matrix (56,631 test events):**

| | Predicted: No Damage | Predicted: Damage |
|---|---|---|
| **Actual: No Damage** | 40,274 (96.5%) | 1,454 (3.5%) |
| **Actual: Damage** | 2,954 (19.8%) | 11,949 (80.2%) |

The model correctly flags 80.2% of damaging storms while clearing 96.5% of harmless ones — meaning claims teams get an early warning list without being flooded with false alarms. The honest limitation: roughly 1 in 5 damaging storms is missed, so this model accelerates first response but doesn't replace final field confirmation.

### Stage 2 — Regression (How much damage?)

| Model | R² | RMSE (log scale) | Notes |
|---|---|---|---|
| Linear Regression | 0.341 | 1.6806 | Baseline |
| Random Forest | 0.612 | 1.2902 | +80% R² vs. baseline |
| **XGBoost ✅** | **0.653** | **1.2194** | Final model — 24 engineered features |

An R² of 0.65 using *only* hazard data (storm type, magnitude, location, timing — no building or exposure data) is consistent with academic benchmarks for hazard-only catastrophe models. An RMSE of 1.22 on the log scale means predictions typically land within roughly 3.4× of the actual dollar figure — useful for early reserving and triage, not final settlement.

---

## Top Feature Drivers (XGBoost)

| Stage | Rank 1 | Rank 2 | Rank 3 |
|---|---|---|---|
| **Stage 1 (Occurrence)** | Magnitude Type | Severity Score | `wfo_target_enc` (weather office) |
| **Stage 2 (Severity)** | `wfo_target_enc` | `mag_x_severity` (magnitude × severity interaction) | Magnitude Type |

`magnitude` (raw physical intensity) is the strongest signal for *whether* damage happens, while `event_severity_score` (a domain-knowledge ranking of storm type destructiveness) dominates *how much* damage happens — together they capture the difference between "is this storm dangerous" and "is this kind of storm dangerous."

---

## Key EDA Findings

- **Zero-inflation:** 73.7% of all recorded storms cause $0 in property damage — the core reason a two-stage model is necessary.
- **Frequency ≠ severity:** Thunderstorm wind is the most frequent event type, but hurricanes average **$151M per event** — over 80x more damage per event than wind, despite being rare.
- **Seasonality:** Damage peaks sharply in August–September (Atlantic hurricane season); coastal states see **2.7× higher average damage** than inland states across all seasons.
- **Conditional effects:** `is_coastal` and `is_hurricane_season` correlate *negatively* with damage severity but *positively* with damage occurrence — they predict whether damage happens, not how much, once it does.
- **Geography:** Engineered target-encoded features for state and weather forecast office (`wfo_target_enc`) became the single strongest Stage 2 predictor, replacing arbitrary location IDs with historical damage signal.

---

## Business Recommendations

1. **Same-day damage triage** — flag likely-damaged locations the moment NOAA logs a storm, instead of waiting days for manual field reports (80.2% damage-event recall).
2. **Loss accumulation monitoring** — map flagged damage by location in real time to catch regional claim clustering early, ahead of a flood of incoming claims.
3. **Same-day reserve estimation** — generate a working dollar estimate immediately for finance and reinsurance triggers, instead of waiting weeks for adjuster reports.
4. **Future enrichment** — the current model is hazard-only; joining it with property exposure data (building value, age, construction type) is expected to raise R² from ~0.65 toward 0.80–0.90 in a future phase.

---

## Tools & Libraries

| Category | Tools |
|---|---|
| Language | Python 3.10+ |
| Data | `pandas`, `numpy` |
| Visualization | `matplotlib`, `seaborn`, `folium` |
| Modeling | `scikit-learn`, `xgboost` |
| Platform | Google Colab / Jupyter |

---

## Ethical Considerations

This model uses geographic features (`state`, `is_coastal`, `is_new_england`, latitude/longitude) as predictors. Storm risk genuinely varies by geography — coastal exposure and hurricane paths are physical phenomena, not proxies for protected demographic characteristics. That said, any operational deployment should ensure damage estimates and reserve recommendations don't inadvertently slow claims response or resource allocation in lower-income or historically underserved regions, since disaster recovery speed already correlates with such factors independent of the model.

---

*ISOM 631 – Predictive Analytics and Machine Learning · Suffolk University MSBA · Spring 2026*
*Team: Shivani, Rebecca Otu, Amna Muqudas*
