# DM & HTN Follow-up Form

> Bahmni HTML Form Entry Prototype — Diabetes Mellitus & Hypertension Clinical Follow-up

<p align="center">
  <img alt="Static Badge" src="https://img.shields.io/badge/status-prototype-yellow?style=flat-square">
  <img alt="Static Badge" src="https://img.shields.io/badge/stack-vanilla_JS-blue?style=flat-square">
  <img alt="Static Badge" src="https://img.shields.io/badge/single_file-%7E4130_lines-blue?style=flat-square">
  <img alt="Static Badge" src="https://img.shields.io/badge/mobile_responsive-yes-brightgreen?style=flat-square">
  <img alt="Static Badge" src="https://img.shields.io/badge/license-MIT-green?style=flat-square">
  <img alt="Static Badge" src="https://img.shields.io/badge/calendar-Ethiopian_%2B_Gregorian-blue?style=flat-square">
  <img alt="Static Badge" src="https://img.shields.io/badge/clinical_guidelines-WHO_%2B_AAP_%2B_NKF-orange?style=flat-square">
</p>

A standalone, mobile-responsive clinical follow-up form for **Diabetes Mellitus (DM)** and **Hypertension (HTN)** patients, built as a pure HTML/CSS/JS single-page application. Designed for eventual conversion to a **Bahmni HTML Form Entry** template with OpenMRS concept mappings.

---

## Table of Contents

1. [Features at a Glance](#features-at-a-glance)
2. [Quick Start](#quick-start)
3. [Architecture Overview](#architecture-overview)
4. [Form Structure & Field Reference](#form-structure--field-reference)
    - [Test Demographics](#test-demographics)
    - [Diagnosis Categories](#diagnosis-categories)
    - [Treatment Given](#treatment-given)
    - [Drug Adherence](#drug-adherence)
    - [Subjective Section](#subjective-section)
    - [Objective Section](#objective-section)
    - [Pertinent Lab Ix](#pertinent-lab-ix)
    - [Disease Outcome](#disease-outcome)
    - [Treatment (Active Plan)](#treatment-active-plan)
    - [Appointment](#appointment)
5. [Clinical Logic Engine](#clinical-logic-engine)
    - [Auto-Calculations](#auto-calculations)
    - [Validation Constraints](#validation-constraints)
    - [Conditional Visibility](#conditional-visibility)
    - [Risk-Linked Lifestyle Buttons](#risk-linked-lifestyle-buttons)
    - [MMAS-4 Adherence Modal](#mmas-4-adherence-modal)
6. [Clinical Algorithms](#clinical-algorithms)
    - [Pediatric BP Classification (AAP 2017)](#pediatric-bp-classification-aap-2017)
    - [Hypotensive Detection](#hypotensive-detection)
    - [CVD Risk (WHO 2019 revised)](#cvd-risk-who-2019-revised)
    - [Estimated GFR](#estimated-gfr)
7. [Data Management](#data-management)
    - [Save / Snapshot System](#save--snapshot-system)
    - [Copy Last Observation](#copy-last-observation)
    - [Reset (Add New)](#reset-add-new)
8. [UI Features](#ui-features)
    - [Ethiopian Date Picker](#ethiopian-date-picker)
    - [Mobile Responsiveness](#mobile-responsiveness)
    - [Integer-Only Input Enforcement](#integer-only-input-enforcement)
9. [Bahmni EMR Integration](#bahmni-emr-integration)
    - [Concept Mapping](#concept-mapping)
    - [Simplified Integration Path](#simplified-integration-path)
    - [Professional Integration Guide](#professional-integration-guide)
10. [Developer Notes](#developer-notes)
11. [References](#references)
12. [License](#license)

---

## Features at a Glance

| Domain | Capabilities |
|--------|-------------|
| **Clinical Decision Support** | WHO 2019 CVD Risk (AFR-E), AAP 2017 Pediatric BP classification, CKiD U25 & 2021 CKD-EPI eGFR, Hypotensive detection |
| **Auto-Calculations** | HTN Grade, BMI + WHO Grading, DM/HTN/Dyslipidemia status, Waist Metabolic Risk, Drug Adherence (MMAS-4), GFR with automatic equation switching by age |
| **Conditional Logic** | Dynamic row visibility, diagnosis-gated outcome rows, risk-linked lifestyle buttons, per-item complication/comorbidity status rows |
| **Data Management** | Column-based in-memory snapshots, Copy Last Observation with skip list, complete Reset (Add New) |
| **Ethiopian Context** | 13-month Ethiopian calendar picker, Ethiopian holidays, Gregorian conversion for Bahmni, WHO CVD Risk for Eastern Sub-Saharan Africa |
| **EMR Integration** | Full OpenMRS concept mapping, Bahmni HTML Form Entry compatible, Quota-Based Appointment concept bridge |
| **Mobile Responsive** | Three breakpoints: tablet (≤768px) and stacked card layout (≤480px) |

---

## Quick Start

No build tools, no server, no dependencies required:

1. **Open** `DM_HTN_Followup.html` in any modern browser (Chrome, Firefox, Edge)
2. **Test** different patient profiles using the **Test Demographics** section at the top (Age + Sex toggle)
3. **Enter** vitals, labs, and diagnoses — all auto-calculations fire in real time
4. **Save** snapshots inline via the Save button; **Copy** the last observation or **Reset** to start fresh

> [!TIP]
> Set Age < 13 to activate pediatric BP classification. Set Age between 40–74 to test WHO CVD Risk calculation. The form adjusts all algorithms automatically based on demographics.

---

## Architecture Overview

| Aspect | Detail |
|--------|--------|
| **Stack** | HTML5 + CSS3 + Vanilla ES6 JavaScript (no frameworks, no build tools) |
| **File** | Single file: `DM_HTN_Followup.html` (~4130 lines) |
| **State** | Client-side only — save creates in-memory DOM snapshot columns |
| **Calendar** | Ethiopian calendar (E.C.) with Gregorian conversion for the appointment picker |
| **Mock Data** | Hardcoded `dictConcepts` array simulates OpenMRS concept search |
| **Demographics** | Mock `patientDemographics` object (`{ age, gender }`) drives all conditional rules. Editable via in-form **Test Demographics** section (Age input + Sex toggle buttons) for testing pediatric vs adult pathways |

---

## Form Structure & Field Reference

The form is a single HTML `<table>` organized into 9 sections with a Now column (current encounter) and optional saved snapshot columns.

---

### Test Demographics

Replaced by Bahmni patient demographic data in production. Used during prototyping to drive age- and sex-dependent algorithms.

| Field ID | Type | Purpose |
|----------|------|---------|
| `demo-age` | Integer input | Sets `patientDemographics.age`. Triggers `syncDemographics()` |
| Sex toggle | Button group (Male / Female) | Sets `patientDemographics.gender` via `setDemoSex()` |

Algorithms affected: HTN Grade, Pediatric BP, CVD Risk, GFR equation selection, BMI, Waist risk, Pregnancy row visibility.

---

### Diagnosis Categories

| Field ID | Type | Behavior |
|----------|------|----------|
| `search-dm` | Autocomplete text input | Single-select from `dictConcepts.dm`. Triggers `updateLifestyleButtons()`, `calculateCVDRisk()`, `autoCalculateDMStatus()` |
| `search-htn` | Autocomplete text input | Single-select from `dictConcepts.htn`. Triggers `autoCalculateHTNStatus()` |
| `obs-htn-grade` | Read-only text input | Auto-calculated from SBP/DBP values via `calculateHTNGrade()`. Hidden row (`row-htn-grade`) shown only when BP entered |
| `search-complic` | Autocomplete + pills | Multi-select from `dictConcepts.complic` (DM/HTN complications). Pills render in `#pill-box-complic` |
| `search-comorb` | Autocomplete + pills | Multi-select from `dictConcepts.comorb` (other comorbidities). Pills render in `#pill-box-comorb` |

---

### Treatment Given

Wrapped in `<tbody id="treatment-given-section">` — hidden by default. Shown **only** when **Copy Last Observation** is executed. Never included in Save snapshots.

#### Non-Pharmacologic

- Button group with lifestyle modification buttons in a `#life-style-btns` container
- Each button may carry `data-risks` attributes for conditional visibility

#### Pharmacologic Rows

| Field ID | Drug Class | 💊 MMAS-4 Icon |
|----------|-----------|:---:|
| `obs-oral-bg-drugs` | Oral Hypoglycemic Drugs | Yes |
| `obs-dm-insulin` | DM (Insulin) | Yes |
| `obs-htn-treatment` | HTN Treatment | Yes |
| `obs-antiplatelet` | Antiplatelet | Yes |
| `obs-dyslipidemia` | Dyslipidemia | Yes |
| `obs-other-meds` | Other Medication | Yes |

Each row and its 💊 icon toggle visibility via `checkPharm()` — hidden when empty, shown when content exists.

Copy operation maps these to `*-copy` IDs (e.g. `obs-oral-bg-drugs` → `obs-oral-bg-drugs-copy`) in the active Treatment section.

---

### Drug Adherence

| Field ID | Type | Behavior |
|----------|------|----------|
| `obs-overall-adherence` | Read-only text input | Auto-evaluated via `calculateOverallAdherence()`: **Good** if all MMAS-4 responses = "No"; **Poor** if any = "Yes". Color-coded green/red |

Row `row-overall-adherence` is positioned outside the hidden `#treatment-given-section` tbody so it saves independently.

---

### Subjective Section

| Field ID | Type | Notes |
|----------|------|-------|
| `obs-symptoms` | Textarea | Recent Complaint |
| `search-risks` | Autocomplete + pills | Multi-select risk factors from `dictConcepts.risks`. Pills in `#pill-box-risks`. Drives lifestyle button visibility and CVD Risk smoking detection |
| Pregnancy buttons | Button group (Yes / No) | Shown only for females aged 15–49 (`row-pregnant`). Selecting "No" reveals conceive planning row (`row-conceive`) |

Hidden fields for EMR mapping:

| Field ID | Type | Maps to |
|----------|------|---------|
| `obs-pregnant` | Hidden | Pregnancy Status concept |
| `obs-conceive` | Hidden | Planning to Conceive concept |

---

### Objective Section

#### Vitals

| Field ID | Type | Triggers |
|----------|------|----------|
| `obs-sbp` | Integer text input | HTN Grade + HTN Status + Pediatric BP + CVD Risk |
| `obs-dbp` | Integer text input | HTN Grade + HTN Status + Pediatric BP |
| `obs-pulse` | Integer text input | Pulse volume/rhythm row visibility |
| `obs-weight` | Integer text input | BMI + WHO Grading + CVD Risk (non-lab mode) |
| `obs-height` | Integer text input | BMI + WHO Grading + Pediatric BP (age < 13) + CKiD U25 eGFR (age ≤ 25) |
| `obs-bmi` | Read-only text (hidden) | `weight(kg) / (height(m))²` |
| `obs-bmi-grading` | Read-only text (hidden) | WHO 8-class BMI classification |
| `obs-waist` | Integer text (hidden, age ≥ 20) | Waist Metabolic Risk |
| `obs-metabolic-risk` | Read-only text (hidden) | Waist-based risk by gender |

#### Pulse Assessment

| Row | Trigger | Options |
|-----|---------|---------|
| `row-pulse-volume` | Pulse rate has a positive integer | Non-palpable / Feeble/Weak / Strong (button group) |
| `row-pulse-rythm` | Pulse rate has a positive integer | Regular / Regularly-Irregular / Irregularly-Irregular (button group) |

> [!NOTE]
> Pulse volume and rhythm use button groups without explicit `obs-` field IDs. Values are captured by the snapshot system reading the selected button text. For EMR integration, suggested concept IDs: `obs-pulse-volume`, `obs-pulse-rhythm`.

#### Physical Exam

A "Pertinent P/E Positive Finding?" button group (Yes/No) toggles 7 exam rows:

| Row ID | Field ID | Exam Area |
|--------|----------|-----------|
| `.pertinent-exam-row` | `obs-exam-oral` | Oral/Dental |
| `.pertinent-exam-row` | `obs-exam-heart` | Heart |
| `.pertinent-exam-row` | `obs-exam-arteries` | Peripheral Arteries |
| `.pertinent-exam-row` | `obs-exam-foot` | MSK-Foot |
| `.pertinent-exam-row` | `obs-exam-mental` | Mental Status |
| `.pertinent-exam-row` | `obs-exam-motor` | Motor |
| `.pertinent-exam-row` | `obs-exam-sensory` | Sensory |

---

### Pertinent Lab Ix

| Field ID | Type | Auto-Calculation |
|----------|------|------------------|
| `obs-fbs` | Integer text | DM Status + Ketone trigger (≥250 mg/dL) |
| `obs-rbs` | Integer text | DM Status + Ketone trigger (≥350 mg/dL) |
| `obs-hga1c` | Integer text | DM Status |
| `obs-ketone` | Text (hidden row) | Shown when FBS ≥ 250 or RBS ≥ 350 |
| `obs-albumin` | Select (Nil / Trace / 1+ / 2+ / 3+ / 4+) | 24-hr Urine Protein row shown on 2+/3+/4+ |
| `obs-micro` | Textarea | Microscopic findings |
| `obs-urine-prof` | Text (hidden row) | 24-hr Urine Protein |
| `obs-creatinine` | Decimal text | eGFR + Stage (G1–G5). Equation auto-selects: CKiD U25 (≤25 yr) or 2021 CKD-EPI (>25 yr) |
| `obs-gfr` | Read-only text (hidden) | **Age ≤ 25:** "Estimated GFR by CKiD U25 Creatinine" (requires height). **Age > 25:** "GFR(ml/min/1.73 m2)" via 2021 CKD-EPI |
| `obs-gfr-kdigo` | Read-only text (hidden) | G1–G5 with color coding. **Age ≤ 25:** "CKiD U25 Category". **Age > 25:** "KDIGO Category" |
| `obs-na` | Integer text | — |
| `obs-k` | Integer text | — |
| `obs-total-cholesterol` | Integer text | CVD Risk (lab mode) + Cholesterol category |
| `obs-triglyceride` | Integer text | — |
| `obs-ldl` | Integer text | Dyslipidemia Status (Controlled if LDL < 70) |
| `obs-cvd-risk` | Read-only badge (hidden) | WHO/ISH 2019 CVD Risk (AFR-E) — shown when dyslipidemia-copy empty and age 40–74 |
| `obs-ecg` | Textarea | — |
| `obs-echo` | Textarea | — |
| `obs-fundoscopic` | Textarea | — |
| `obs-other-investigation` | Textarea | — |
| `obs-overall-assessment` | Textarea | — |

---

### Disease Outcome

| Field ID | Type | Visibility Condition |
|----------|------|---------------------|
| `dm-status` | Select (Controlled / Uncontrolled) | **Type of DM** (`search-dm`) has a diagnosis **AND** FBS/RBS/HgA1c has data |
| `htn-status` | Select (Controlled / Uncontrolled) | **Type of HTN** (`search-htn`) has a diagnosis **AND** SBP/DBP has data |
| `obs-dyslipidemia-status` | Select (Controlled / Uncontrolled / Unknown) | Auto-computed from LDL when `obs-dyslipidemia-copy` has entry |
| `outcome-status-complic-*` (dynamic) | Select per complication | One row per pill in `#pill-box-complic`, created/removed dynamically |
| `outcome-status-comorb-*` (dynamic) | Select per comorbidity | One row per pill in `#pill-box-comorb`, created/removed dynamically |

Dynamic rows are rendered inside `<tbody id="disease-outcome-entries">` and are synchronized with pill state via `syncDiseaseOutcomeRows()`.

---

### Treatment (Active Plan)

This section contains the **current encounter treatment plan**, always visible. Medications are copied from Treatment Given via the Copy Last Observation workflow.

#### Non-Pharmacologic

Button group in `#treatment-life-style-btns` with 11 lifestyle modification buttons, each carrying `data-*` attributes for conditional visibility (see [Risk-Linked Lifestyle Buttons](#risk-linked-lifestyle-buttons)):

- Healthy Diet • Cessation of Smoking • Exercise • Weight Loss • Reduce Alcohol use • Cessation of Chewing Khat • Medication Adherence • Medication Side effects • Proper Insulin/Drug storage • Proper Insulin Injection • Daily Foot Inspection

> [!NOTE]
> The Treatment (Active Plan) section includes **Cessation of Chewing Khat** and uses `risk-linked` conditional visibility for 8 of 11 buttons. The Treatment Given section has a comparable but non-identical set without Khat cessation and without `risk-linked` data attributes.

#### Pharmacologic (Copy Fields)

| Field ID | Source Field |
|----------|-------------|
| `obs-oral-bg-drugs-copy` | `obs-oral-bg-drugs` |
| `obs-dm-insulin-copy` | `obs-dm-insulin` |
| `obs-htn-treatment-copy` | `obs-htn-treatment` |
| `obs-antiplatelet-copy` | `obs-antiplatelet` |
| `obs-dyslipidemia-copy` | `obs-dyslipidemia` |
| `obs-other-meds-copy` | `obs-other-meds` |

#### Linked Entries & Notes

| Parent Field | Child Row | Child Field | Visibility Logic |
|-------------|-----------|-------------|-----------------|
| `obs-linked-to` | `row-linkage-note` | `obs-linkage-note` | Parent has content |
| `obs-consultation-to` | `row-consultation-note` | `obs-consultation-note` | Parent has content |
| `obs-referral-to` | `row-referral-reason` | `obs-referral-reason` | Parent has content |

| Field ID | Type |
|----------|------|
| `obs-remark` | Textarea |

---

### Appointment

| Field ID | Type | Behavior |
|----------|------|----------|
| `obs-appointment` | Read-only text (Ethiopian calendar) | Displays selected follow-up date in E.C. format. **Must** be mapped to the pre-existing **Quota-Based Appointment** concept in Bahmni EMR |
| `obs-appointment-gregorian` | Hidden | Stores Gregorian equivalent for Bahmni interoperability |
| Picker | Custom calendar widget | Restricts to today + future dates only. Saturdays, Sundays, and Ethiopian full holidays are disabled |

> [!IMPORTANT]
> The `obs-appointment` field **must** be mapped to the pre-existing **Quota-Based Appointment** concept configured within the Bahmni EMR appointment scheduling module. This integration ensures that each scheduled follow-up date is validated against the facility's appointment quota limits, preventing overbooking and enabling streamlined patient flow management. The Ethiopian date picker feeds directly into this quota-enforced slot allocation system via the hidden Gregorian conversion field, which serves as the interoperability bridge between the form's calendar widget and Bahmni's downstream scheduling services.

---

## Clinical Logic Engine

### Auto-Calculations

Every auto-calculation function, its trigger events, and its outputs:

| Function | Trigger | Output(s) |
|----------|---------|-----------|
| `calculateHTNGrade()` | SBP or DBP or Height input | `obs-htn-grade`: **Adult (≥13):** Normal / Grade-1 / Grade-2 / Grade-3 / Hypotensive. **Pediatric (<13):** Normal / Elevated / Stage 1 HTN / Stage 2 HTN / Hypotensive (height-adjusted percentile thresholds) |
| `autoCalculateHTNStatus()` | SBP, DBP input, or `search-htn` change | `htn-status`: Controlled / Uncontrolled (pediatric: Normal+Elevated → Controlled, Hypotensive+Stage1+Stage2 → Uncontrolled). Row hidden until Type of HTN diagnosed |
| `calculateHeightZScore()` | Height, age, sex | Height-for-age z-score using CDC LMS reference data |
| `calculatePediatricBPThresholds()` | Age, sex, height | 90th and 95th percentile SBP/DBP thresholds interpolated from AAP 2017 normative tables |
| `classifyPediatricBP()` | SBP, DBP, thresholds | Pediatric BP category: Normal / Elevated / Stage 1 HTN / Stage 2 HTN / Hypotensive |
| `autoCalculateDMStatus()` | FBS, RBS, HgA1c input, or `search-dm` change | `dm-status`: Controlled / Uncontrolled. Row hidden until Type of DM diagnosed |
| `autoCalculateBMI()` | Weight or Height input | `obs-bmi` + `obs-bmi-grading` (WHO 8-class) |
| `autoCalculateWaistRisk()` | Waist input | `obs-metabolic-risk`: Normal / Increased / Greatly Increased (by gender) |
| `autoCalculateGFR()` | Creatinine, Height, or demographics change | `obs-gfr` (CKiD U25 ≤25 yr / 2021 CKD-EPI >25 yr) + G-stage via `updateGFRLabels()` |
| `calculateOverallAdherence()` | MMAS-4 modal save | `obs-overall-adherence`: Good / Poor |
| `calculateCVDRisk()` | SBP, TC, weight, height, DM, smoking status, dyslipidemia-copy, demographics | `row-cvd-risk` badge: <5% / 5–<10% / 10–<20% / 20–<30% / ≥30% |
| `autoCalculateDyslipidemiaStatus()` | `obs-dyslipidemia-copy`, LDL | `obs-dyslipidemia-status`: Controlled (LDL<70) / Uncontrolled (LDL≥70) / Unknown (no LDL) |
| `updateGFRLabels()` | Demographics age change | Swaps labels between CKiD U25 (≤25 yr) and 2021 CKD-EPI (>25 yr) for GFR and G-stage rows |
| `syncDiseaseOutcomeRows()` | Pill add/remove in `#pill-box-complic` or `#pill-box-comorb` | Creates/removes per-item status rows in `<tbody id="disease-outcome-entries">` |

---

### Validation Constraints

Numeric fields validate on blur via `validateNumericRange()`. Out-of-range values trigger a red border + inline error message.

| Field | Range | Notes |
|-------|-------|-------|
| Systolic BP | 50 – 300 | `obs-sbp` |
| Diastolic BP | 30 – 180 | `obs-dbp` |
| Pulse Rate | 30 – 200 | `obs-pulse` |
| Weight | 5 – 250 | `obs-weight` (kg) |
| Height | 50 – 250 | `obs-height` (cm) |
| Waist Circumference | 50 – 180 | `obs-waist` (cm) |
| FBS | 30 – 600 | `obs-fbs` (mg/dL) |
| RBS | 30 – 600 | `obs-rbs` (mg/dL) |
| HgA1c | 4 – 20 | `obs-hga1c` (%) |
| Creatinine | 0.05 – 15 | `obs-creatinine` (mg/dL, decimal) |
| Na+ | 70 – 200 | `obs-na` (mEq/L) |
| K+ | 1 – 10 | `obs-k` (mEq/L) |
| Total Cholesterol | 50 – 500 | `obs-total-cholesterol` (mg/dL) |

---

### Conditional Visibility

| Element | Condition |
|---------|-----------|
| `row-htn-grade` | SBP or DBP has a value |
| `row-peds-bp-result` | Age < 13 + SBP + DBP + Height all have values (shows classification badge + 90th/95th thresholds + height z-score) |
| `row-pulse-volume`, `row-pulse-rythm` | Pulse rate has a positive integer |
| `row-bmi`, `row-bmi-zscore` | Both weight and height have values |
| `row-waist-entry` | Patient age ≥ 20 (evaluated on DOM load) |
| `row-metabolic-risk` | Waist has a value |
| `row-urine-ketone` | FBS ≥ 250 or RBS ≥ 350 |
| `row-urine-protein` | Albumin = 2+, 3+, or 4+ |
| `row-gfr`, `row-gfr-kdigo` | Creatinine has a value. For ≤25 yr (CKiD U25), height also required |
| `row-dm-status` | Type of DM (`search-dm`) has a diagnosis **and** FBS/RBS/HgA1c has a value |
| `row-htn-status` | Type of HTN (`search-htn`) has a diagnosis **and** SBP/DBP has a value |
| `#disease-outcome-entries tr` | Created dynamically for each pill in `#pill-box-complic` and `#pill-box-comorb` |
| `row-dyslipidemia-status` | `obs-dyslipidemia-copy` has a value (treatment section dyslipidemia entry). Auto-hides when cleared |
| `row-pregnant` | Gender = F and age 15–49 |
| `row-conceive` | Pregnant answer = "No" |
| `row-overall-adherence` | Any pharmacologic field has content |
| `row-linked-to`, `row-linkage-note` | `obs-linked-to` has content |
| `row-consultation-to`, `row-consultation-note` | `obs-consultation-to` has content |
| `row-referral-to`, `row-referral-reason` | `obs-referral-to` has content |
| `.pertinent-exam-row` (7 rows) | P/E finding toggle = "Yes" |
| Pharmacologic rows + 💊 icons | Respective textarea has content (via `checkPharm()`) |
| `#treatment-given-section` | Hidden by default; shown on Copy Last Observation |

---

### Risk-Linked Lifestyle Buttons

In the active Treatment section (`#treatment-life-style-btns`), buttons with class `risk-linked` conditionally show/hide based on data attributes evaluated by `updateLifestyleButtons()`:

| Data Attribute | Logic |
|---------------|-------|
| `data-risks` | Show if any listed risk factor pill exists in `#pill-box-risks` |
| `data-min-bmi` | Show if calculated BMI ≥ threshold |
| `data-show-on-adherence` | Show if adherence value matches (e.g. "Poor") |
| `data-show-on-field` | Show if specified field ID has content |
| `data-show-on-dm` | Show if DM type matches |
| `data-show-on-comorb` | Show if any listed condition pill exists in `#pill-box-comorb` or `#pill-box-complic` |

Buttons hide automatically when conditions are no longer met. The function is triggered on pill add/remove, demographics change, DM type selection, and adherence update.

---

### MMAS-4 Adherence Modal

Opened by clicking any 💊 icon next to a pharmacologic textarea in the Treatment Given section.

**Workflow:**
1. Select drug name (auto-populated from the row's field label)
2. Answer **"Missed Dose?"** — Yes / No
3. If Yes → answer 4 MMAS-4 questions:
   - Forgetfulness — "Do you sometimes forget to take your medicine?" (Yes/No)
   - Carelessness — "Are you sometimes careless about taking your medicine?" (Yes/No)
   - Feeling Better — "When you feel better, do you sometimes stop taking your medicine?" (Yes/No)
   - Feeling Worse — "When you feel worse, do you sometimes stop taking your medicine?" (Yes/No)
4. Save → icon turns green ✅, `adherenceState` object updated in memory
5. `calculateOverallAdherence()` evaluates:
   - **Good** — all MMAS-4 responses = "No"
   - **Poor** — any MMAS-4 response = "Yes"

The adherence state is structured per-drug and aggregated to produce the overall adherence reading.

---

## Clinical Algorithms

### Pediatric BP Classification (AAP 2017)

When `patientDemographics.age < 13`, the form switches from adult HTN grading to the **2017 AAP pediatric BP classification system**, which requires the child's sex, exact age, height, and blood pressure to compute percentile-based thresholds.

#### CDC Stature-for-Age LMS Data

Embedded `CDC_STATURE_LMS` object provides L, M, S parameters for stature-for-age at yearly intervals (ages 1–13) for both sexes, sourced from the CDC 2000 growth charts. The `calculateHeightZScore(age, sex, height_cm)` function interpolates between yearly entries and applies the LMS formula:

```
Z = ((X / M)^L - 1) / (L × S)
```

Handles the edge case where L ≈ 0 using the logarithmic form.

#### AAP 2017 Normative BP Tables

Embedded `AAP_BP` lookup tables provide systolic and diastolic BP values at the 50th, 90th, and 95th percentiles for ages 1–12, both sexes, across 7 height percentile buckets (5th–95th). The `getPediatricBPThresholds()` function:

1. Converts the height Z-score → nearest height percentile bucket
2. Looks up BP thresholds from the embedded AAP tables
3. Interpolates between adjacent height percentile entries for accuracy

#### Classification Categories

| Category | Criteria | Badge Color |
|----------|----------|-------------|
| **Normal** | SBP and DBP both < 90th percentile | Green (`#5cb85c`) |
| **Elevated** | SBP or DBP ≥ 90th but < 95th percentile, or ≥ 120/80 but still < 95th percentile | Yellow (`#f0ad4e`) |
| **Stage 1 HTN** | SBP or DBP ≥ 95th percentile but < 95th + 12 mmHg, or 130/80–139/89 | Orange (`#e67e22`) |
| **Stage 2 HTN** | SBP or DBP ≥ 95th + 12 mmHg, or ≥ 140/90 | Red (`#d9534f`) |
| **Hypotensive** | SBP or DBP below estimated 5th percentile (2×p50 − p95) or age-based absolute minimum (70 + age×2, capped at 90) | Cyan (`#5bc0de`) |

The higher severity between SBP and DBP determines the final category. A **Pediatric BP Classification** row (`row-peds-bp-result`) appears below the DBP input showing the color-coded badge, computed 90th/95th percentile thresholds, and the height Z-score.

#### Age Cutoff

| Age | System | Threshold Source |
|-----|--------|-----------------|
| < 13 | AAP 2017 pediatric classification | Percentile tables (age/sex/height adjusted) |
| ≥ 13 | Adult HTN grading | Fixed cutoffs (140/90, 160/100, 180/110) |

---

### Hypotensive Detection

Hypotensive classification is applied to **both** age groups:

| Group | Criteria |
|-------|----------|
| **Pediatric (< 13 yr)** | SBP < estimated 5th percentile (`2 × p50 − p95`) **or** SBP < age-based minimum (`70 + age × 2`, capped at 90 for age > 10) **or** DBP < estimated 5th percentile **or** DBP < 50 |
| **Adult (≥ 13 yr)** | SBP < 90 **or** DBP < 60 |

In `calculateHTNGrade()`, hypotension is shown as a distinct label in the `obs-htn-grade` field. In `autoCalculateHTNStatus()`, hypotension maps to **Uncontrolled**.

---

### CVD Risk (WHO 2019 revised)

The form computes a 10-year cardiovascular disease risk estimate using the **2019 revised WHO cardiovascular disease risk charts** for **Eastern Sub-Saharan Africa (AFR-E)**. This uses a lookup-table-based approach for both Laboratory and Non-Laboratory models.

#### Display

A **CVD Risk** row (`row-cvd-risk`) appears below the LDL entry in the *Pertinent Lab Ix* section, showing a color-coded badge:

| Risk Level | Badge Color |
|------------|-------------|
| <5% | Green (`#4CAF50`) |
| 5% to <10% | Yellow (`#FFC107`) |
| 10% to <20% | Orange (`#FF9800`) |
| 20% to <30% | Red (`#F44336`) |
| ≥30% | Dark Red (`#B71C1C`) |

#### Visibility

The row is hidden by default and appears only when:
- The **Dyslipidemia** textarea (`obs-dyslipidemia-copy`) in the Treatment (Active Plan) section is **empty**.
- Patient age is **within 40–74 years** (otherwise the row is hidden and skipped entirely).

> [!TIP]
> Use the **Test Demographics** section (Age input + Sex toggle) at the top of the form to test adult (40–74) vs. out-of-range scenarios without reloading the page.

When dyslipidemia-copy has a value, or age is outside 40–74, the CVD Risk row is hidden and `autoCalculateDyslipidemiaStatus()` runs instead to show the **Dyslipidemia Disease Outcome** row (auto-computed from LDL).

#### Inputs and Modes

The calculator supports two modes:

| Mode | Condition | Additional Inputs |
|------|-----------|-------------------|
| **Lab-based** | Total Cholesterol (`obs-total-cholesterol`) has a value ≥ 1 | TC value mapped to one of 5 cholesterol categories for lookup |
| **Non-lab (BMI-based)** | Total Cholesterol is empty or 0 | Uses BMI (computed from weight & height) with the non-lab (chol=0) lookup column |

**Common inputs** (required for both modes):

| Input | Source | Notes |
|-------|--------|-------|
| Age | `patientDemographics.age` | Must be 40–74 (lookup table range). Use Test Demographics section |
| Sex | `patientDemographics.gender` | M = Male, F = Female. Use sex toggle |
| SBP | `obs-sbp` | Categorical: <120, 120–129, 130–149, 150–169, ≥170 |
| Diabetes | `search-dm` = "Type 2 Diabetes Mellitus" | dm=1 if exact match, else dm=0 |
| Smoking | `#pill-box-risks` pills | Smoker if any pill text contains "tobacco" or "smok" (case-insensitive) |

#### Lookup Tables

The form utilizes three JavaScript arrays acting as lookup tables:
- `WHO_2019_NONLAB_AFRE`: Non-laboratory model (uses BMI)
- `WHO_2019_LAB_NONDM_AFRE`: Laboratory model for people without diabetes
- `WHO_2019_LAB_DM_AFRE`: Laboratory model for people with diabetes

The structures are nested by **Age Group** → **SBP Category** → **Sex/Smoking/TC or BMI Index**.
Each derived cell contains an exact risk percentage which maps to a risk level (0–4) corresponding to the five band labels above.

**Cholesterol code mapping:**
| Internal Category Index | WHO Standard (`mmol/L`) | Form Input Approx (`mg/dL`) |
|------------------------|------------------------|----------------------------|
| **0** | `< 4.0` | `< 155` |
| **1** | `4.0 to < 5.0` | `155 to < 193` |
| **2** | `5.0 to < 6.0` | `194 to < 232` |
| **3** | `6.0 to < 7.0` | `232 to < 270` |
| **4** | `≥ 7.0` | `≥ 271` |

#### Trigger Events

`calculateCVDRisk()` runs whenever any of these inputs change:
- `obs-sbp` (SBP)
- `obs-total-cholesterol` (TC — switches lab/non-lab mode)
- `obs-weight` or `obs-height` (BMI for non-lab mode)
- `search-dm` (diabetes status)
- `pill-box-risks` pill add/remove (smoking status)
- `obs-dyslipidemia-copy` (visibility gate — show CVD Risk when empty, show Dyslipidemia Status when filled)
- Demographics age/sex change

Additionally, `autoCalculateDyslipidemiaStatus()` runs directly on `obs-ldl` changes to update the Dyslipidemia Disease Outcome status in real time.

---

### Estimated GFR

The form automatically selects the appropriate eGFR equation based on patient age — no manual switching required.

#### CKiD U25 Creatinine (Age ≤ 25)

For patients aged **25 years and under**, the form uses the **CKiD U25 creatinine eGFR equation** (Pierce et al., 2021).

> [!NOTE]
> CKiD U25 is recommended by the NKF for pediatric and young adult populations. The traditional CKD-EPI equation significantly overestimates GFR in this group.

**Equation:**
```
eGFR = k × (height(m) / sCr(mg/dL))
```

Where **k** is age- and sex-dependent:

| Age Range | Male (k) | Female (k) |
|-----------|----------|------------|
| 1 to <12 years | `39.0 × 1.008^(age−12)` | `36.1 × 1.008^(age−12)` |
| 12 to <18 years | `39.0 × 1.045^(age−12)` | `36.1 × 1.023^(age−12)` |
| 18 to 25 years | 50.8 | 41.4 |

**Inputs:**

| Input | Source |
|-------|--------|
| Scr (serum creatinine) | `obs-creatinine` (mg/dL) |
| Age | `patientDemographics.age` |
| Sex | `patientDemographics.gender` |
| Height | `obs-height` (cm) — **required** |

#### 2021 CKD-EPI Creatinine (Age > 25)

For patients aged **over 25 years**, the form uses the **2021 CKD-EPI Creatinine equation** (NKF/ASN Task Force), the current clinical standard for adults.

> [!NOTE]
> This is the **race-free** 2021 equation (no coefficient for Black / non-Black). It supercedes the 2009 MDRD and 2012 CKD-EPI equations.

**Equation:**
```
eGFR = 142 × (Scr / A)^B × 0.9938^age × (1.012 if female)
```

Where **A** and **B** are sex-dependent:

| Sex | Scr ≤ A | Scr > A |
|-----|---------|---------|
| **Female** | A = 0.7, B = −0.241 | A = 0.7, B = −1.200 |
| **Male**   | A = 0.9, B = −0.302 | A = 0.9, B = −1.200 |

If the patient is female, the result is multiplied by **1.012**.

**Inputs:**

| Input | Source |
|-------|--------|
| Scr (serum creatinine) | `obs-creatinine` (mg/dL) |
| Age | `patientDemographics.age` |
| Sex | `patientDemographics.gender` |

No height input needed — GFR is normalized to 1.73 m² BSA.

#### Staging (Both Equations)

The resulting eGFR is classified into KDIGO G-categories:

| Stage | eGFR (mL/min/1.73 m²) | Color |
|-------|------------------------|-------|
| **G1** | ≥ 90 | Green |
| **G2** | 60–89 | Light Green |
| **G3a** | 45–59 | Yellow |
| **G3b** | 30–44 | Orange |
| **G4** | 15–29 | Red |
| **G5** | < 15 | Dark Red |

#### Automatic Equation Switching

| Condition | Equation Used | GFR Label | Stage Label | Height Dependency |
|-----------|---------------|-----------|-------------|-------------------|
| Age ≤ 25 | CKiD U25 (Pierce 2021) | *Estimated GFR by CKiD U25 Creatinine* | *CKiD U25 Category* | Required |
| Age > 25 | 2021 CKD-EPI (Inker 2021) | *GFR(ml/min/1.73 m2)* | *KDIGO Category* | Not needed |

The switch is managed by `updateGFRLabels()` which dynamically rewrites field labels and placeholders. The equation selection occurs inside `autoCalculateGFR()` which reads the current age from `patientDemographics.age`.

---

## Data Management

### Save / Snapshot System

`saveFormSnapshot()` creates a **column-based snapshot** in the DOM.

| Aspect | Detail |
|--------|--------|
| **Session ID** | `save-session-{timestamp}` — reused within 12-hour window for updates |
| **Header** | Appends `<td class="saved-cell">` with formatted timestamp to the header row |
| **Per-row value** | Reads `.input-cell` content: pills → button selections → input/textarea/select values |
| **Exclusion** | Rows inside `#treatment-given-section` are skipped |
| **Status** | "Form saved successfully at …" message shown for 4 seconds |

Multiple snapshots create multiple saved columns side-by-side.

---

### Copy Last Observation

`copyLastObservation()` restores the **most recent saved snapshot** into the Now column.

#### Skip List

The following fields are **never overwritten** by Copy:

- **Vitals**: `obs-sbp`, `obs-dbp`, `obs-weight`
- **Subjective**: `obs-symptoms`
- **Labs**: `obs-fbs`, `obs-rbs`, `obs-hga1c`, `obs-ketone`, `obs-albumin`, `obs-micro`, `obs-urine-prof`, `obs-creatinine`, `obs-gfr`, `obs-gfr-kdigo`, `obs-na`, `obs-k`, `obs-total-cholesterol`, `obs-triglyceride`, `obs-ldl`, `obs-ecg`, `obs-echo`, `obs-fundoscopic`, `obs-other-investigation`, `obs-overall-assessment`
- **Admin**: `obs-appointment`, `obs-overall-adherence`

#### Treatment Given Population

On Copy:
1. Hidden `#treatment-given-section` is shown (`display: ''`)
2. Medications are copied from the snapshot into the Treatment Given pharmacologic textareas
3. Lifestyle buttons in Treatment Given are selected and hidden buttons are set to `display: none`
4. `checkPharm()` runs to sync pharmacologic row/icon visibility

---

### Reset (Add New)

`resetNowColumn()` clears the entire Now column:
- All inputs, textareas, selects reset to empty
- All pills removed from complication, comorbidity, and risk factor containers
- All button selections cleared
- `#treatment-given-section` hidden
- Appointment field and note reset
- `updateLifestyleButtons()` re-evaluated
- `syncDiseaseOutcomeRows()` runs to clear dynamic outcome rows

---

## UI Features

### Ethiopian Date Picker

| Feature | Implementation |
|---------|---------------|
| **Months** | 13 Ethiopian months (Meskerem–Pagume) |
| **Year range** | Defaults to current Ethiopian year; navigable forward only |
| **Past dates** | Disabled — cannot select before today |
| **Weekends** | Saturday and Sunday disabled |
| **Full holidays** | 8 fixed Ethiopian holidays + 5 configured movable Gregorian holidays; disabled with red styling |
| **Output** | `obs-appointment`: formatted string (e.g. "Tikimt 12, 2017 E.C.") |
| **Hidden output** | `obs-appointment-gregorian`: ISO date string for Bahmni |

---

### Mobile Responsiveness

| Breakpoint | Layout |
|------------|--------|
| > 768 px | Standard table layout with 200px label + 160px saved + auto input columns |
| ≤ 768 px | Reduced padding, narrower cells, smaller inputs |
| ≤ 480 px | **Stacked card layout**: each table row becomes a bordered block with label as full-width header, input and saved cells below as block elements. Buttons, pills, and inputs expand to 100% width |

---

### Integer-Only Input Enforcement

15 numeric fields enforce integer-only input via `allowOnlyInteger()`:

`obs-sbp` • `obs-dbp` • `obs-pulse` • `obs-weight` • `obs-height` • `obs-waist` • `obs-fbs` • `obs-rbs` • `obs-hga1c` • `obs-creatinine` • `obs-na` • `obs-k` • `obs-total-cholesterol` • `obs-triglyceride` • `obs-ldl`

**Implementation**: `oninput` handler strips all non-digit characters using `input.value.replace(/[^0-9]/g, '')`. Uses `type="text"` (not `type="number"`) to avoid cross-browser spinner inconsistencies.

> [!NOTE]
> `obs-creatinine` uses `allowDecimalInput()` instead to permit decimal values (range 0.05–15.0 mg/dL).

---

## Bahmni EMR Integration

### Concept Mapping

Each field ID maps to an OpenMRS concept. Fields are organized by form section. Copy fields (`*-copy`) represent the treatment plan copy of the same medication concept.

#### Diagnosis & Subjective

| Field ID | Concept | Datatype |
|----------|---------|----------|
| `search-dm` | Type of DM | Coded |
| `search-htn` | Type of HTN | Coded |
| `search-complic` | Complication(s) | Multi-select Coded |
| `search-comorb` | Comorbidity | Multi-select Coded |
| `search-risks` | Risk Factors | Multi-select Coded |
| `obs-symptoms` | Recent Complaint | Text |
| `obs-pregnant` | Pregnancy Status | Coded |
| `obs-conceive` | Planning to Conceive | Coded |
| `obs-htn-grade` | HTN Grade (auto-calculated) | Text |

#### Vitals

| Field ID | Concept | Datatype |
|----------|---------|----------|
| `obs-sbp` | Systolic Blood Pressure | Numeric |
| `obs-dbp` | Diastolic Blood Pressure | Numeric |
| `obs-pulse` | Pulse Rate | Numeric |
| `obs-weight` | Weight (kg) | Numeric |
| `obs-height` | Height (cm) | Numeric |
| `obs-bmi` | BMI (auto-calculated) | Calculated |
| `obs-bmi-grading` | BMI WHO Grading (auto-calculated) | Text |
| `obs-waist` | Waist Circumference (cm) | Numeric |
| `obs-metabolic-risk` | Waist Metabolic Risk (auto-calculated) | Text |

#### Physical Exam

| Field ID | Concept | Datatype |
|----------|---------|----------|
| `obs-exam-oral` | Oral/Dental Exam Finding | Text |
| `obs-exam-heart` | Heart Exam Finding | Text |
| `obs-exam-arteries` | Peripheral Arteries Exam Finding | Text |
| `obs-exam-foot` | MSK-Foot Exam Finding | Text |
| `obs-exam-mental` | Mental Status Exam Finding | Text |
| `obs-exam-motor` | Motor Exam Finding | Text |
| `obs-exam-sensory` | Sensory Exam Finding | Text |

#### Lab Investigations

| Field ID | Concept | Datatype |
|----------|---------|----------|
| `obs-fbs` | Fasting Blood Sugar (mg/dL) | Numeric |
| `obs-rbs` | Random Blood Sugar (mg/dL) | Numeric |
| `obs-hga1c` | Hemoglobin A1c (%) | Numeric |
| `obs-ketone` | Urine Ketone | Text |
| `obs-albumin` | Urine Albumin (Protein) | Coded |
| `obs-micro` | Urine Microscopic Findings | Text |
| `obs-urine-prof` | 24-hour Urine Protein | Numeric |
| `obs-creatinine` | Serum Creatinine (mg/dL) | Numeric |
| `obs-gfr` | eGFR — CKiD U25 (≤25 yr) / 2021 CKD-EPI (>25 yr) | Calculated |
| `obs-gfr-kdigo` | G-Stage — CKiD U25 Category / KDIGO Category | Text |
| `obs-na` | Sodium (mEq/L) | Numeric |
| `obs-k` | Potassium (mEq/L) | Numeric |
| `obs-total-cholesterol` | Total Cholesterol (mg/dL) | Numeric |
| `obs-triglyceride` | Triglyceride (mg/dL) | Numeric |
| `obs-ldl` | LDL (mg/dL) | Numeric |
| `obs-cvd-risk` | WHO/ISH CVD Risk (auto-calculated) | Calculated |
| `obs-ecg` | ECG Finding | Text |
| `obs-echo` | ECHO Finding | Text |
| `obs-fundoscopic` | Fundoscopic Finding | Text |
| `obs-other-investigation` | Other Investigation Finding | Text |
| `obs-overall-assessment` | Overall Assessment | Text |

#### Disease Outcome

| Field ID | Concept | Datatype |
|----------|---------|----------|
| `dm-status` | DM Status (Controlled / Uncontrolled) | Coded |
| `htn-status` | HTN Status (Controlled / Uncontrolled) | Coded |
| `obs-dyslipidemia-status` | Dyslipidemia Status (Controlled / Uncontrolled / Unknown) | Coded |
| `outcome-status-complic-*` | Per-complication status (Same / Corrected / Controlled / Uncontrolled / Unknown) | Coded |
| `outcome-status-comorb-*` | Per-comorbidity status (Same / Corrected / Controlled / Uncontrolled / Unknown) | Coded |

#### Pharmacologic — Baseline (Treatment Given)

| Field ID | Concept | Datatype |
|----------|---------|----------|
| `obs-oral-bg-drugs` | Oral Hypoglycemic Drugs | Text |
| `obs-dm-insulin` | DM Insulin | Text |
| `obs-htn-treatment` | HTN Treatment | Text |
| `obs-antiplatelet` | Antiplatelet | Text |
| `obs-dyslipidemia` | Dyslipidemia Treatment | Text |
| `obs-other-meds` | Other Medication | Text |
| `obs-overall-adherence` | Drug Adherence (Good / Poor) | Text |

#### Pharmacologic — Active Plan (Copy Fields)

| Field ID | Concept | Datatype |
|----------|---------|----------|
| `obs-oral-bg-drugs-copy` | Oral Hypoglycemic Drugs (Plan) | Text |
| `obs-dm-insulin-copy` | DM Insulin (Plan) | Text |
| `obs-htn-treatment-copy` | HTN Treatment (Plan) | Text |
| `obs-antiplatelet-copy` | Antiplatelet (Plan) | Text |
| `obs-dyslipidemia-copy` | Dyslipidemia Treatment (Plan) | Text |
| `obs-other-meds-copy` | Other Medication (Plan) | Text |

#### Linked Entries, Referral & Appointment

| Field ID | Concept | Datatype |
|----------|---------|----------|
| `obs-linked-to` | Linked To | Text |
| `obs-linkage-note` | Linkage Note | Text |
| `obs-consultation-to` | Consultation To | Text |
| `obs-consultation-note` | Consultation Note | Text |
| `obs-referral-to` | Referral To | Text |
| `obs-referral-reason` | Referral Reason | Text |
| `obs-remark` | Remark | Text |
| `obs-appointment` | Quota-Based Appointment Concept (Ethiopian Calendar) — maps to Bahmni's pre-configured appointment concept for quota-enforced scheduling | Date |
| `obs-appointment-gregorian` | Appointment Date (Gregorian, hidden) — interoperability bridge for Bahmni appointment module | Date |

---

### Simplified Integration Path (For Bahmni Beginners)

If you are new to Bahmni, you can still deploy these calculations without writing a full Angular app:

1. **Standard Fields**: Use the [Bahmni Form Builder (GUI)](https://bahmni.atlassian.net/wiki/spaces/BAH/pages/147128330/Form+Builder) to create a standard "DM/HTN Progress Note."
2. **Logic Injection**: Add a single "Custom Control" field in your form design specifically for the **CVD Risk Badge**.
3. **Copy-Paste Logic**: Copy the `calculateCVDRisk()` function and the `WHO_2019` arrays from this file into your Bahmni `form-conditions.js`.
4. **Field IDs**: Ensure the IDs you choose in the Form Builder match the IDs in our `dictConcepts` (e.g., `obs-sbp`, `obs-ldl`).

This "Low-Code" approach allows the EMR to perform the math automatically while using Bahmni's built-in form-saving features.

> [!TIP]
> See the [Concept Mapping](#concept-mapping) table above for the complete list of field IDs and their suggested OpenMRS concept mappings.

---

### Professional Integration Guide

As this form moves from a high-fidelity prototype to a production clinical environment, the following architectural strategy ensures 100% preservation of all ad-hoc clinical logic (CVD Risks, Pediatric BP, GFR) and UI features.

#### Architectural Strategy: The Bridge Pattern

Do not use the standard Bahmni Form Builder for the complex calculation blocks. Instead, use the **Bahmni Custom Display Control** methodology to preserve the high-fidelity UI and clinical logic.

| Layer | Implementation |
|-------|---------------|
| **Logic Layer (Angular Service)** | Move all math functions (`calculateCVDRisk`, `calculateHTNGrade`, `classifyPediatricBP`) into a standalone JS file: `/var/www/bahmni_config/openmrs/apps/clinical/customControl/js/cvdCalculationService.js` |
| **UI Layer (Custom Directive)** | Create an AngularJS directive that wraps this HTML form, binding UI elements directly to the `$scope.observations` object |

#### Implementation Steps

| Step | Action | Description |
|------|--------|-------------|
| **1** | **Concept Mapping** | Map all `obs-` IDs (including hidden ones like `obs-pregnant`, `obs-conceive`) to their OpenMRS Concept UUIDs |
| **2** | **Logic Migration** | Move the WHO 2019 Matrix Arrays and calculation logic into the `cvdCalculator` service |
| **3** | **CDS Integration** | Wire the Statin Recommendation Logic into Bahmni's `form-conditions.js` to trigger real-time expert advice |
| **4** | **UI Persistence** | Bind the dynamic risk badges to hidden input fields to ensure calculated results are saved to the OpenMRS database for reporting |

#### Preservation of Key Features

- **CVD Risk (WHO 2019)**: Logic stays in the Angular Service; Matrix arrays are kept in a separate constant
- **Pediatric BP**: Handled by the same service using the patient's birthdate and height-for-age z-score logic
- **Statin Suggestion**: Real-time calculation triggers an inline alert box (`cvd-risk-suggestion`) based on the latest 2019 guidelines
- **UI Styling**: All CSS styles from this prototype can be moved into Bahmni's `custom.css`

> [!IMPORTANT]
> **Data Integrity**: By mapping interactive UI fields to OpenMRS Concept UUIDs, you ensure that even though the UI is "custom," the clinical data remains structured, searchable, and interoperable across the EMR.

---

## Developer Notes

- **No external dependencies** — pure HTML/CSS/JS. Loads in any modern browser offline.
- **ES6 features** used throughout: arrow functions, `const`/`let`, template literals, `Array.from()`, `Array.includes()`.
- **File naming convention**: `DM_HTN_Followup.html` — use exact filename for GitHub Pages URL.
- **Testing**: load directly in browser, no server required.
- **Blood pressure thresholds**: Controlled = SBP < 140 AND DBP < 90 (per Ethiopian treatment guidelines).
- **DM thresholds**: Controlled = HgA1c < 7, or HgA1c 7–8 with FBS ≤ 125; if no HgA1c, FBS ≤ 125 or RBS < 200.
- **HTN Grade thresholds**:
  - **Adult (≥13 yr)**: Normal < 140/90, Grade-1 ≥ 140/90, Grade-2 ≥ 160/100, Grade-3 ≥ 180/110, Hypotensive < 90/60.
  - **Pediatric (<13 yr)**: Uses 2017 AAP percentile-based classification with CDC height-for-age z-score. Categories: Normal (both < 90th %ile), Elevated (≥ 90th but < 95th %ile, or ≥ 120/80), Stage 1 HTN (≥ 95th %ile but < 95th+12, or 130/80–139/89), Stage 2 HTN (≥ 95th+12, or ≥ 140/90), Hypotensive (< estimated 5th %ile or age-based absolute minimum). Controlled = Normal/Elevated, Uncontrolled = Hypotensive/Stage 1/Stage 2.
- **WAIST risk thresholds** (WHO): Male ≤ 94 cm normal, 94–102 increased, > 102 greatly increased. Female ≤ 80 cm normal, 80–88 increased, > 88 greatly increased.
- **BMI WHO Grading**: 8-class WHO classification: Severe Thinness (< 16.0), Moderate Thinness (16.0–16.9), Mild Thinness (17.0–18.4), Normal (18.5–24.9), Pre-Obesity/Overweight (25.0–29.9), Obesity Class I (30.0–34.9), Obesity Class II (35.0–39.9), Obesity Class III / Morbid (≥ 40.0).
- **eGFR equation switching**: `autoCalculateGFR()` uses `updateGFRLabels()` to dynamically swap labels and equation (see [Estimated GFR](#estimated-gfr)). **Age ≤ 25:** CKiD U25 equation (`eGFR = k × height/cr`) with age/sex-specific k values; height is required. **Age > 25:** 2021 CKD-EPI equation (`142 × (Scr/A)^B × 0.9938^age × gender`); height not needed.
- **CVD Risk age gate**: `calculateCVDRisk()` hides the CVD Risk row entirely when age is outside 40–74 (calls `autoCalculateDyslipidemiaStatus()` as fallback). The WHO 2019 lookup tables only support ages 40–74.
- **Dynamic disease outcome rows**: `syncDiseaseOutcomeRows()` creates one status row per complication/comorbidity pill, each with its own select (Same/Corrected/Controlled/Uncontrolled/Unknown). Rows are removed when the corresponding pill is removed. Uses `<tbody id="disease-outcome-entries">` container.
- **DM/HTN status diagnosis gating**: `autoCalculateDMStatus()` and `autoCalculateHTNStatus()` only show their respective outcome rows when the corresponding diagnosis has been entered in the Diagnosis Categories section (`search-dm` for DM, `search-htn` for HTN). This prevents disease outcome rows from appearing before a formal diagnosis is documented.

---

## References

1. **Federal Ministry of Health, Ethiopia.** *National Training on Screening and Comprehensive Management of Hypertension and Diabetes Mellitus at Primary Health Care Level.* Addis Ababa, Ethiopia; 2023.

2. **World Health Organization.** *HEARTS Technical Package for Cardiovascular Disease Management in Primary Health Care: Risk-Based CVD Management.* WHO, Geneva; 2019.

3. **World Health Organization.** *WHO Cardiovascular Disease Risk Charts: Revised Models to Estimate Risk in 21 Global Regions.* Lancet Glob Health 2019; 7(10): e1332–e1345.

4. **Pierce CB, et al.** *A New eGFR Equation for Children and Young Adults (CKiD U25).* Am J Kidney Dis 2021; 78(1): 99–109.

5. **Inker LA, et al. (NKF/ASN Task Force).** *New Creatinine- and Cystatin C–Based Equations to Estimate GFR without Race.* N Engl J Med 2021; 385: 1737–1749.

6. **Flynn JT, et al. (AAP).** *Clinical Practice Guideline for Screening and Management of High Blood Pressure in Children and Adolescents.* Pediatrics 2017; 140(3): e20171904.

---

## License

This project is provided as a reference prototype for Bahmni HTML Form Entry implementations. Licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details. Use, modify, and distribute freely for clinical and educational purposes.

---

<p align="center">
  <sub>Maintained by <strong>Dr. Teselonke K.</strong> — Prototype for Bahmni HTML Form Entry conversion.</sub>
  <br>
  <sub>Built with vanilla HTML, CSS & JavaScript — no frameworks, no build tools, no dependencies.</sub>
</p>
