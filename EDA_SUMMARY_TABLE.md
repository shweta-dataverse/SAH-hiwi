# SMART (SAH) Dataset — EDA Summary Tables

Data source: `SMART_timeSeriesData_preliminary` · 175 patient CSV files · long format (`Feature | Value | Unit | Time`)

---

## TABLE 1 — Dataset at a Glance

| Metric | Value |
|---|---|
| Total patients | 175 |
| Total measurements (rows) | 1,945,892 |
| Unique features | 88 |
| Admission years covered | 2013–2024 |
| Median rows per patient | 9,295 |
| Min / Max rows per patient | 9 (ID_100) / 77,357 (ID_096) |
| Median monitoring duration | 15.0 days |
| Mean monitoring duration | 44.5 days |
| Target variable | `mRS score` |
| Target coverage | 100% (all 175 patients) |
| Non-numeric value rate | see notebook (`Value` coercion) |
| File encoding | latin-1 (German medical units, e.g. µmol) |

---

## TABLE 2 — Patient Data Quality Verdict

| Verdict | Count | Meaning |
|---|---|---|
| OK to use | 155 | Sufficient rows and duration |
| REVIEW — too few rows (<100) | 13 | Too sparse for reliable modelling |
| REVIEW — <1 day data | 3 (additional) | No time-series signal |
| EXCLUDE — export error (>180 days) | 4 | Medically impossible duration |

*Note: some patients carry more than one flag; the verdict shows the most severe.*

---

## TABLE 3 — Export-Error Patients (EXCLUDE / verify with clinical records)

| Patient ID | Duration | Rows | Why flagged |
|---|---|---|---|
| ID_146 | 3,311 days (≈9 years) | 28,748 | Impossible ICU stay — timestamp bug |
| ID_143 | 1,120 days (≈3 years) | 10,038 | Impossible ICU stay — timestamp bug |
| ID_133 | 330 days | 8,867 | Implausible — verify |
| ID_038 | 308 days | 3,801 | Implausible — verify |

---

## TABLE 4 — Patients with < 100 Rows (REVIEW — too little data)

| Patient ID | Rows | Duration (days) |
|---|---|---|
| ID_100 | 9 | 1.81 |
| ID_090 | 30 | 0.00 |
| ID_116 | 30 | 0.00 |
| ID_152 | 34 | 4.04 |
| ID_097 | 38 | 0.00 |
| ID_037 | 48 | 0.87 |
| ID_003 | 61 | 0.03 |
| ID_155 | 63 | 0.02 |
| ID_154 | 74 | 0.03 |
| ID_119 | 75 | 0.46 |
| ID_031 | 85 | 0.02 |
| ID_137 | 90 | 27.83 |
| ID_012 | 98 | 0.04 |

---

## TABLE 5 — Patients with < 1 Day of Data (REVIEW — no time-series value)

| Patient ID | Duration (days) | Rows |
|---|---|---|
| ID_090 | 0.000 | 30 |
| ID_097 | 0.000 | 38 |
| ID_116 | 0.000 | 30 |
| ID_155 | 0.022 | 63 |
| ID_031 | 0.023 | 85 |
| ID_003 | 0.031 | 61 |
| ID_154 | 0.033 | 74 |
| ID_012 | 0.043 | 98 |
| ID_056 | 0.062 | 101 |
| ID_119 | 0.458 | 75 |
| ID_091 | 0.865 | 637 |
| ID_037 | 0.870 | 48 |
| ID_080 | 0.975 | 810 |

---

## TABLE 6 — Feature Coverage Tiers

| Tier | Coverage | # Features | Modelling action |
|---|---|---|---|
| Core | > 90% of patients | 34 | Use directly |
| Supplementary | 50–90% of patients | 49 | Use with imputation |
| Sparse | < 50% of patients | 5 | Likely exclude / handle separately |

---

## TABLE 7 — Core Features (>90% coverage) — the reliable signal set

| Category | Features | Coverage |
|---|---|---|
| Target | mRS score | 100% |
| Electrolytes | Calcium, Natrium, Kalium, Chlorid | 100% |
| Red blood cells | Hämatokrit, Hämoglobin, Erythrozytenzahl, MCH, MCHC, MCV, Eryth.-Verteilungsbr., Mittl. Thromb.-Vol. | 99.4% |
| Kidney | Creatinin, eGFR (CKD-EPI), Harnstoff | 99.4% |
| Inflammation/infection | C-reaktives Protein, Leukozytenzahl | 99.4% |
| Metabolic | Glucose | 99.4% |
| Coagulation | Thrombozytenzahl, INR, aPTT, TPZ (Quick), Anteil großer Thromboz. | 99.4% |
| Liver | ALAT (96.6%), ASAT (96.0%), GGT (96.0%), Alk. Phosphatase (94.9%), Albumin (94.9%) | 94–97% |
| Vitals | Herzfrequenz (96%), SaO2 (92.6%), Temperatur (92.6%), Sys art Druck (91.4%), Mit art Druck (90.3%) | 90–96% |

*Clinical note: ICP (Hirndruck) is at 58.9% but is critically important for SAH (marker of secondary brain injury).*

---

## TABLE 8 — Top Vitals: Value Ranges & Plausibility

| Feature | Unit | Median | Range (min–max) | Count | Note |
|---|---|---|---|---|---|
| Herzfrequenz (HR) | bpm | 80 | 0–206 | 234,071 | 0 = monitor artifact |
| Mit art Druck (MAP) | mmHg | 92 | 0–360 | 215,596 | >360 = sensor error |
| Sys art Druck (SBP) | mmHg | 140 | 12–360 | 210,243 | Slightly high (expected in SAH) |
| Dia art Druck (DBP) | mmHg | 67 | 11–321 | 209,621 | Normal |
| SaO2 (O2 sat) | % | 96 | 0–100 | 206,008 | 0 = probe off |
| Temperatur | °C | 37.5 | −0.44–45 | 204,639 | Extremes = errors |
| O2 inspiratorisch | % | 30 | 20–174 | 109,959 | >100 implausible |
| CO2 Endtidal | mmHg | 44 | 0–99 | 107,559 | Normal |
| ICP (Hirndruck) | mmHg | 12 | −41–361 | 92,432 | >361 = sensor error; <20 normal |

---

## TABLE 9 — Top Outlier Features (IQR × 3)

| Feature | Outlier % | n values | Min–Max | Interpretation |
|---|---|---|---|---|
| GLDH (liver enzyme) | 10.3% | 184 | 15–162,600 | Acute liver stress |
| Zellzahl kernhaltig (CSF) | 7.5% | 653 | 2–15,967 | LP results vary widely |
| Leukozyten (CSF) | 7.5% | 654 | 0–17,640 | **Expected high in SAH** |
| Erythrozyten (CSF) | 6.6% | 654 | 0–2,079,000 | **Blood in CSF — SAH signal** |
| Immature Granul. | 5.8% | 794 | 0.01–2.30 | Infection marker |
| Calcium, ionisiert | 5.4% | 8,479 | 0.69–2.22 | Hypercalcemia episodes |
| Lipase | 4.9% | 163 | 0.14–24.82 | Pancreatic |
| LDH | 4.8% | 187 | 1.73–315 | Tissue damage |
| aPTT (coagulation) | 4.7% | 2,862 | 21–172 | Coagulopathy — clinically relevant |
| Pank.-Amylase | 4.5% | 134 | 0.05–10.50 | Pancreatic |

**Key insight:** CSF outliers (Leukozyten, Erythrozyten) are NOT noise — they are expected in SAH because the hemorrhage bleeds into the cerebrospinal fluid. Do not remove without clinical validation.

---

## TABLE 10 — Patients by Admission Year

| Year | Patients |
|---|---|
| 2013 | 1 |
| 2019 | 30 |
| 2020 | 35 |
| 2021 | 51 |
| 2022 | 26 |
| 2023 | 23 |
| 2024 | 9 |

---

## TABLE 11 — Key Findings & Insights (master summary)

| # | Finding | Evidence | Why it matters / Action |
|---|---|---|---|
| 1 | This is an SAH outcome-prediction task | `mRS score` in 100% of patients | Defines the target: predict mRS at discharge |
| 2 | Large but uneven dataset | 1.95M rows, 9–77,357 per patient | Need minimum-data threshold; weight patients |
| 3 | 4 patients have impossible durations | ID_146 (9y), ID_143, ID_133, ID_038 | Verify with clinical records; likely exclude |
| 4 | 13 patients have <100 rows | See Table 4 | Too sparse — agree exclusion rule |
| 5 | 13 patients have <1 day data | See Table 5 | No temporal signal for sequence models |
| 6 | 34 core features (>90% coverage) | Coverage analysis | Solid feature set for first models |
| 7 | 5 features <50% coverage | Horovitz-Quotient (12%), O2Hb (5%) | Likely exclude — too many missing |
| 8 | Vitals medians clinically normal | Table 8 | Data is broadly trustworthy |
| 9 | Sensor artifacts present (0s, >360 mmHg) | HR=0, BP=360, ICP=361 | Clip or flag before modelling |
| 10 | CSF outliers are SAH signal | Erythrozyten in CSF up to 2.08M | Do NOT blindly remove outliers |
| 11 | Multi-year data (2013–2024) | Table 10 | Use relative time (hours since admission) |
| 12 | Median ICU stay ≈ 15 days | Duration analysis | ~130 patients have >7 days → enough for sequence models |

---

## TABLE 12 — Recommended Modelling Roadmap

| Step | Approach | Output | When |
|---|---|---|---|
| 1 | Tabular baseline (XGBoost on 24–72h aggregates) | Interpretable benchmark | First |
| 2 | + Temporal features (trends, time-above-threshold) | Captures clinical dynamics | Next |
| 3 | Sequence model (LSTM/GRU) on resampled grid | Learns temporal patterns | After full dataset |
| 4 | Transformer (irregular time-series) | State-of-the-art | If data supports it |

---

## TABLE 13 — Open Questions for Clinical Team

| # | Question | Reason |
|---|---|---|
| 1 | Verify ID_146/143/133/038 durations | Suspected export errors |
| 2 | ID_090/097/116 single-timestamp data — error or retrospective? | 0-minute durations |
| 3 | Is mRS at discharge or 3-month follow-up? | Defines prediction target |
| 4 | Are BP/HR zeros disconnections or real events? | Imputation strategy |
| 5 | Why CSF values in only 63% — protocol or documentation? | Feature inclusion |
| 6 | Meaning of `Chlorid/Vitalstatus` feature | Feature understanding |
| 7 | Predict at admission (24h) or full stay? | Scope of model |
| 8 | Size/timeline of complete labelled dataset? | Planning |
| 9 | Add WFNS / Fisher / Hunt & Hess scores? | Strong known SAH predictors |
