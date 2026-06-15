# SMART Project — Presentation Guide
## EDA Findings & Meeting Preparation

---

# BEFORE YOU START: Understand the project in one sentence

**This project uses ICU monitoring data from 175 Subarachnoid Hemorrhage (SAH) patients
to predict their functional outcome (mRS score) after hospital discharge.**

Every clinical measurement, every vital sign, every lab result you see in the data is a
potential predictor. The target variable — `mRS score` (modified Rankin Scale) — measures
how disabled a patient is at discharge: 0 = fully recovered, 6 = dead.

---

# PRESENTATION STRUCTURE (recommended order)

---

## SLIDE 1 — Project Context (2 min)
*What are we studying and why does it matter?*

### What to say:
"Subarachnoid Hemorrhage (SAH) is a life-threatening stroke caused by bleeding around
the brain. It carries a 30-day mortality of ~30% and leaves many survivors with severe
disabilities. The SMART project aims to use real ICU monitoring data to predict, early in
the admission, how well a patient will recover — measured by the modified Rankin Scale (mRS)."

### Key clinical facts to state:
- mRS 0–2 = good outcome (independent, no significant disability)
- mRS 3–5 = poor outcome (dependent, severe disability)
- mRS 6 = death
- **Being able to predict this early would allow clinicians to adjust treatment intensity,
  better counsel families, and prioritize resources.**

### Why this matters to the hospital team:
"You collect all of this data already. We are asking: can the pattern in the first 24–72
hours of ICU monitoring tell us what the outcome will be weeks later?"

---

## SLIDE 2 — Dataset Overview (2 min)
*What data do we have?*

### Facts from the data (cite these exactly):
| Metric | Value |
|---|---|
| Total patients | **175** |
| Total individual measurements | **1,945,892** (~1.9 million rows) |
| Monitoring period covered | **2013–2024** (11 years) |
| Unique clinical features measured | **88** |
| Data format | Long-format time series |
| Target variable | `mRS score` — present in **100% of patients** |

### What to say:
"Each patient has their own CSV file containing every single measurement taken during their
ICU stay — vitals every 15 minutes, blood tests, intracranial pressure, ventilator settings.
In total, nearly 2 million rows of data across 175 patients. This is genuine real-world
clinical data, not a curated benchmark dataset, which means it contains real-world messiness
we need to understand before modelling."

### Admission year breakdown:
| Year | Patients |
|---|---|
| 2013 | 1 |
| 2019 | 30 |
| 2020 | 35 |
| 2021 | 51 |
| 2022 | 26 |
| 2023 | 23 |
| 2024 | 9 |

**Important point:** Different years mean different patients admitted at different times.
When we model, we will use time-since-admission (relative hours), NOT calendar dates.

---

## SLIDE 3 — Data Quality: Patients (3 min)
*How usable is each patient's data?*

**Show: `fig_01_rows_per_patient.png` + `fig_02_monitoring_duration.png`**

### Key finding 1 — Highly variable data volume per patient:
- Median patient: **9,295 measurement rows**
- Minimum: **9 rows** (ID_100) — almost no data at all
- Maximum: **77,357 rows** (ID_096) — extremely intensive monitoring

**Why this matters:** A model trained on all patients equally will be dominated by heavily
monitored patients. We need a minimum data threshold.

### Key finding 2 — Patients with too little data (flag for exclusion discussion):

**Patients with fewer than 100 rows — consider excluding:**
| Patient ID | Rows | Duration |
|---|---|---|
| ID_100 | 9 | 1.8 days |
| ID_090 | 30 | 0 days |
| ID_116 | 30 | 0 days |
| ID_152 | 34 | 4 days |
| ID_097 | 38 | 0 days |
| ID_037 | 48 | 0.9 days |
| ID_003 | 61 | 0 days |
| ID_155 | 63 | 0 days |
| ID_154 | 74 | 0 days |
| ID_119 | 75 | 0.5 days |
| ID_031 | 85 | 0 days |
| ID_137 | 90 | 27.8 days |
| ID_012 | 98 | 0 days |

**What to say:** "13 patients have fewer than 100 measurements. This does not mean they
were not sick — they may have been transferred or died quickly. But from a modelling
perspective, 9 or 30 data points cannot reliably represent a patient's physiological
trajectory. We recommend discussing a minimum threshold with the clinical team."

### Key finding 3 — Patients with < 1 day of monitoring:

**Patients with all data within a single 24-hour window:**
ID_090 (0 min), ID_097 (0 min), ID_116 (0 min), ID_155 (32 min), ID_031 (33 min),
ID_003 (45 min), ID_154 (47 min), ID_012 (62 min), ID_056 (89 min), ID_119 (11 h),
ID_091 (20 h), ID_037 (20 h), ID_080 (23 h)

**Why this matters clinically:** There is no time-series pattern to learn from a single
day. For sequential models (LSTM, Transformer), a minimum of several days is needed.

### Key finding 4 — Data export errors:

**Patients with suspiciously long durations (medically impossible ICU stays):**
| Patient ID | Duration | Rows |
|---|---|---|
| ID_146 | **3,311 days (9 years!)** | 28,748 |
| ID_143 | **1,120 days (3 years)** | 10,038 |
| ID_133 | **330 days** | 8,867 |
| ID_038 | **308 days** | 3,801 |

**What to say:** "No ICU patient stays for 9 years. These are almost certainly timestamp
errors in the data export system — perhaps a patient's records were exported with incorrect
year values. We recommend the clinical team verify these four cases before they are
included in any model. They are shown in red in our scatter plot."

**Show: `fig_05_duration_scatter.png`**

---

## SLIDE 4 — Data Quality: Features (3 min)
*Which measurements are reliable enough to use?*

**Show: `fig_06_feature_coverage_histogram.png` + `fig_07_top30_features.png`**

### The 88 features split into three tiers:

| Tier | Coverage | Count | Use in modelling |
|---|---|---|---|
| Core features | > 90% of patients | **34** | Safe to use directly |
| Supplementary | 50–90% of patients | **49** | Use with imputation strategy |
| Sparse | < 50% of patients | **5** | Likely exclude or treat separately |

### Core features (>90% coverage) — these are your reliable signals:

**Vitals (continuous monitoring, every 15 min):**
- Herzfrequenz (Heart Rate) — 96% coverage, 234,071 measurements
- Mit/Sys/Dia art Druck (Arterial Blood Pressure — mean/systolic/diastolic)
- SaO2 (Oxygen Saturation) — 92.6%
- Temperatur (Body Temperature) — 92.6%
- ICP/Hirndruck (Intracranial Pressure) — 58.9% ← critical for SAH

**Lab values (daily/twice daily blood tests):**
- Calcium, Natrium, Kalium, Chlorid (electrolytes) — **100%** all patients
- Hämatokrit, Hämoglobin, Erythrozytenzahl (red blood cell markers)
- Leukozytenzahl (white blood cells — infection marker)
- CRP (C-reactive protein — inflammation)
- Creatinin, eGFR (kidney function)
- Glucose
- Thrombozytenzahl (platelets — clotting)
- INR, aPTT, TPZ/Quick (coagulation — critical post-SAH)
- ALAT, ASAT, GGT (liver enzymes)
- Albumin

**What to say:** "The good news is that 34 features appear in more than 90% of patients.
These are the standard ICU monitoring protocol — every patient gets them. This gives us
a solid core feature set. Among them, ICP (intracranial pressure) is particularly
important for SAH specifically, as raised ICP is a direct marker of secondary brain injury."

---

## SLIDE 5 — Target Variable: mRS Score (2 min)
*What are we predicting?*

### Critical finding — mRS is present in 100% of patients:
"The good news from our EDA: the target variable `mRS score` is present in all 175
patients. This means we have labels for every single case, which is unusual for
clinical datasets of this size."

### What is mRS?
| Score | Meaning |
|---|---|
| 0 | No symptoms |
| 1 | No significant disability |
| 2 | Slight disability, independent |
| 3 | Moderate disability, some help needed |
| 4 | Moderately severe, cannot walk unassisted |
| 5 | Severe disability, bedridden |
| 6 | Dead |

### How to frame the prediction task:
**Option A — Binary classification:** Good outcome (mRS 0–2) vs. Poor outcome (mRS 3–6)  
**Option B — Ordinal classification:** Predict the exact mRS score (0–6)  
**Option C — Regression:** Treat mRS as a continuous variable

*Note: The full dataset with mRS labels is still incoming. The preliminary data already
contains the mRS column, which gives us a head start.*

---

## SLIDE 6 — Value Distributions: What Do the Vitals Look Like? (3 min)
*Are the values clinically plausible?*

**Show: fig_08 through fig_19 — one per feature**

### Feature-by-feature clinical interpretation:

| Feature | Median | Range | Clinical note |
|---|---|---|---|
| Heart rate | 80 bpm | 0–206 | Normal. Values of 0 = monitoring artifact |
| Mean arterial pressure | 92 mmHg | 0–360 | Normal. >360 = clear sensor error |
| Systolic BP | 140 mmHg | 12–360 | Slightly elevated — expected in SAH |
| Diastolic BP | 67 mmHg | 11–321 | Normal range |
| SaO2 (O2 saturation) | 96% | 0–100% | Values of 0 = sensor off/detached |
| Temperature | 37.5°C | -0.44–45°C | Median normal; extremes = errors |
| ICP (Hirndruck) | 12 mmHg | -41–361 mmHg | Normal <20; values >361 = sensor error |
| End-tidal CO2 | 44 mmHg | 0–99 | Normal range |

**What to say:** "Looking at the actual value ranges, we see a pattern common in raw
clinical data: the medians are clinically plausible, but the extremes include values that
are physically impossible. A heart rate of 206 is extreme but possible; a systolic BP of
360 mmHg or ICP of 361 mmHg are sensor artifacts. Before modelling, we need to either
clip these values or flag them for removal."

---

## SLIDE 7 — Outliers: Where Are the Extreme Values? (2 min)
*Which features need the most cleaning?*

**Show: `fig_20_outlier_rates.png`**

### Top features by extreme outlier rate (IQR × 3):
| Feature | Outlier % | Note |
|---|---|---|
| GLDH (liver enzyme) | **10.3%** | Likely acute liver stress episodes |
| Zellzahl kernhaltig (CSF cell count) | 7.5% | Lumbar puncture results vary widely |
| Leukozyten (CSF white cells) | 7.5% | Normal in SAH (blood in CSF) |
| Erythrozyten (CSF red cells) | 6.6% | Expected high in SAH — blood enters CSF |
| Calcium ionisiert | 5.4% | Hypercalcemia episodes |
| aPTT (coagulation) | 4.7% | Coagulopathy events — clinically significant |

**What to say:** "The outliers are not random noise. Several of them — like elevated CSF
cell counts and red cells — are actually expected in SAH patients because the hemorrhage
bleeds into the cerebrospinal fluid. These 'outliers' may carry diagnostic signal and
should NOT be blindly removed without clinical validation. We recommend reviewing these
with the clinical team before deciding on a cleaning strategy."

---

## SLIDE 8 — Feature Presence Heatmap (2 min)
*Who was measured for what?*

**Show: `fig_21_feature_presence_heatmap.png`**

### What to say:
"This heatmap shows which of the top 40 features each patient has. Blue = measured,
white = not measured. The mostly-blue pattern tells us that the core clinical protocol
is quite consistent across patients. The white gaps in some rows represent features that
are only ordered when clinically indicated — for example, CSF analysis (Liquor) is only
done after lumbar puncture, which not every patient receives."

---

## SLIDE 9 — What Can We Model? (4 min)
*Possible modelling approaches — present all options to the team*

### The prediction task:
**Input:** ICU time-series measurements from hours 0 to T after admission  
**Output:** mRS score at discharge (or at 3 months, depending on when it is recorded)

### Modelling approaches (from simple to complex):

**Approach 1 — Tabular baseline (first model to build):**
- Aggregate each patient's time series into summary statistics per feature
  (mean, min, max, std over the first 24/48/72 hours)
- Train: Logistic Regression, Random Forest, XGBoost
- **Why start here:** Fast to implement, highly interpretable, gives the clinical team
  a baseline to judge AI against. If this already works well, complex models may not
  be needed.
- Typical outcome: AUC 0.70–0.80 with good feature engineering

**Approach 2 — Classical time-series features:**
- Extract temporal features: trend slopes, variability, time above/below threshold
  (e.g., % of time with ICP > 20 mmHg)
- Still use tabular model (Random Forest/XGBoost)
- **Why this matters:** Clinicians know that *sustained* high ICP is worse than brief
  spikes — duration matters, not just peak values

**Approach 3 — Sequence model (LSTM / GRU):**
- Treat each patient as a sequence of timesteps
- Model learns temporal dependencies automatically
- **Challenge:** Irregular timesteps (different patients measured at different intervals)
  require resampling to a fixed grid (e.g., hourly averages)
- **Challenge:** Requires sufficient sequence length — patients with <2 days excluded

**Approach 4 — Transformer-based model:**
- Self-attention over the time series
- State-of-the-art for irregular clinical time series (e.g., STraTS, Raindrop models)
- Most powerful but requires the most data and computation

### Recommendation to present:
"We recommend starting with Approach 1 as a baseline, then Approach 2 to add temporal
context. Sequence models (Approach 3/4) will be evaluated once we understand how many
patients have sufficient monitoring duration. Based on our EDA, approximately
**130 patients have >7 days of data**, which is a reasonable starting point."

### Key challenge to flag:
**Class imbalance:** The mRS distribution in the full dataset is unknown until labels are
confirmed — but SAH outcomes are typically 40-50% poor outcome. If the split is imbalanced,
models need to be trained with class weights or oversampling (SMOTE).

---

## SLIDE 10 — Open Questions for the Clinical Team (3 min)
*The most important part of the meeting*

### Questions you MUST ask:

**About data quality:**
1. "Can you verify patients ID_146, ID_143, ID_133, and ID_038? Their monitoring duration
   spans years — we believe this is a timestamp export error, but we need clinical
   confirmation before excluding them."

2. "Patients ID_090, ID_097, and ID_116 have zero minutes of data — all measurements are
   at a single timestamp. Was there a recording error, or were these emergency admissions
   where data was entered retrospectively?"

3. "Should the mRS score in the data reflect discharge outcome or 3-month follow-up?
   This matters significantly for how we define the prediction target."

**About feature engineering:**
4. "For features like arterial blood pressure and heart rate, we see values like 0 and
   360. Can you confirm: are zeroes recorded when the monitor is disconnected, or do they
   represent a true physiological event (e.g., cardiac arrest)?"

5. "CSF (cerebrospinal fluid) values — Leukozyten, Erythrozyten, Protein — are only
   available in 63% of patients. Is this because lumbar puncture is not performed in all
   cases, or is there a documentation gap?"

6. "What does `Chlorid/Vitalstatus` mean as a feature — is this the chloride value at
   the time of a vital status check?"

**About the project scope:**
7. "When you say you want to predict outcome — do you mean predicting at the time of
   admission (using only first 24 hours), or can we use the full ICU stay?"

8. "What is the plan for the complete dataset — how many additional patients are expected,
   and when will labels be confirmed?"

9. "Are there additional clinical scores we should consider incorporating? For example:
   WFNS grade (SAH severity at admission), Fisher scale (CT bleeding pattern),
   or Hunt & Hess score — these are standard SAH grading tools and may be very strong
   predictors."

---

## SLIDE 11 — Summary & Next Steps (1 min)

### What we found (say this):
- Dataset: 175 patients, ~2 million measurements, 88 features
- Data quality issues identified: 4 likely export errors, 13 sparse patients, 13 with <1 day
- Core feature set: **34 features available in >90% of patients** — solid foundation
- Target variable confirmed present in all patients: `mRS score`
- Value ranges are mostly clinically plausible; outliers in specific features need
  clinical validation before removal

### What we propose to do next:
1. Agree on exclusion criteria with clinical team (minimum rows / duration)
2. Confirm the 4 export-error patients with clinical records
3. Resolve handling of non-numeric values and outliers
4. Build a baseline tabular model (XGBoost on 24h aggregates)
5. Evaluate temporal features based on the full dataset once available

---

# TIPS FOR DELIVERING THIS TO A MIXED AUDIENCE

**For clinicians (hospital team):**
- Lead with patient outcomes, not data science jargon
- Say "monitoring duration" not "time-series length"
- Say "we found 4 patients with impossible data" not "export errors"
- Always tie every finding back to: *what does this mean for the patient?*

**For your supervisor (Hafez):**
- Show you understand the data limitations — this shows scientific rigor
- Propose concrete decisions (not just "we need to discuss")
- Mention the 3 possible modelling directions clearly

**General tips:**
- Start every slide by stating the question you are answering
- End every slide with one clear takeaway sentence
- Do NOT skip the data quality slides — they are your most important contribution
  at this stage. Modelling on bad data produces bad results.

---

# QUICK REFERENCE: ALL KEY NUMBERS

| Metric | Value |
|---|---|
| Patients | 175 |
| Total rows | 1,945,892 |
| Unique features | 88 |
| Features >90% coverage | 34 |
| Features <50% coverage | 5 |
| Patients with <100 rows | 13 |
| Patients with <1 day data | 13 |
| Suspected export errors | 4 (ID_146, ID_143, ID_133, ID_038) |
| Median monitoring duration | 15 days |
| Median rows per patient | 9,295 |
| Target variable coverage | 100% (mRS score) |
| Top measured feature | Herzfrequenz (Heart Rate) — 234,071 rows |
| Highest outlier feature | GLDH liver enzyme — 10.3% extreme outliers |
| Admission years | 2013–2024 |
