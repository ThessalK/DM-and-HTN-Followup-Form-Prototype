# DM & HTN Follow-up Form — Bahmni HTML Form Entry Prototype

A standalone, mobile-responsive clinical follow-up form prototype for Diabetes Mellitus (DM) and Hypertension (HTN) patients, built as a plain HTML/CSS/JS single-page application. Designed for eventual conversion to a **Bahmni HTML Form Entry** template with OpenMRS concept mappings.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Form Sections & Field Reference](#form-sections--field-reference)
   - [Diagnosis Categories](#diagnosis-categories)
   - [Treatment Given (Hidden / Copy-Only)](#treatment-given-hidden--copy-only)
   - [Drug Adherence](#drug-adherence)
   - [Subjective Section](#subjective-section)
   - [Objective Section](#objective-section)
   - [Pertinent Lab Ix](#pertinent-lab-ix)
   - [Disease Outcome](#disease-outcome)
   - [Treatment (Active Plan)](#treatment-active-plan)
   - [Appointment](#appointment)
3. [Behavioral Rules](#behavioral-rules)
    - [Auto-Calculations](#auto-calculations)
    - [Validation Constraints](#validation-constraints)
    - [Conditional Visibility](#conditional-visibility)
   - [Risk-Linked Lifestyle Buttons](#risk-linked-lifestyle-buttons)
   - [MMAS-4 Adherence Modal](#mmas-4-adherence-modal)
4. [Save / Snapshot System](#save--snapshot-system)
5. [Copy Last Observation](#copy-last-observation)
6. [Reset (Add New)](#reset-add-new)
7. [Integer-Only Input Enforcement](#integer-only-input-enforcement)
8. [Ethiopian Date Picker](#ethiopian-date-picker)
9. [Pediatric BP Classification (AAP 2017)](#pediatric-bp-classification-aap-2017)
10. [Hypotensive Detection](#hypotensive-detection)
11. [Mobile Responsiveness](#mobile-responsiveness)
12. [Integration Guide for Bahmni EMR](#integration-guide-for-bahmni-emr)
11. [Developer Notes](#developer-notes)

---

## Architecture Overview

| Aspect | Detail |
|---|---|
| **Stack** | HTML5 + CSS3 + Vanilla ES6 JavaScript (no frameworks, no build tools) |
| **File** | Single file: `DM_HTN_Followup.html` (~3222 lines) |
| **State** | Client-side only (no persistence layer; save creates in-memory DOM snapshot columns) |
| **Calendar** | Ethiopian calendar (E.C.) with Gregorian conversion for the appointment picker |
| **Mock Data** | Hardcoded `dictConcepts` array simulates OpenMRS concept search |
| **Demographics** | Mock `patientDemographics` object (`{ age, gender }`) drives conditional rules. Editable via in-form **Test Demographics** section (Age input + Sex toggle buttons) for testing pediatric vs adult pathways |

---

## Form Sections & Field Reference

### Diagnosis Categories

| Field ID | Type | Behavior |
|---|---|---|---|
| `search-dm` | Autocomplete input | Single-select from `dictConcepts.dm`. Triggers `updateLifestyleButtons()` on change |
| `search-htn` | Autocomplete input | Single-select from `dictConcepts.htn` |
| `obs-htn-grade` | Read-only input | Auto-calculated from SBP/DBP values. Hidden row (`row-htn-grade`) shown only when BP entered |
| `search-complic` | Autocomplete + pills | Multi-select from `dictConcepts.complic` (DM/HTN complications). Pills in `#pill-box-complic` |
| `search-comorb` | Autocomplete + pills | Multi-select from `dictConcepts.comorb` (other comorbidities). Pills in `#pill-box-comorb` |

### Treatment Given (Hidden / Copy-Only)

Wrapped in `<tbody id="treatment-given-section">` — hidden by default. Shown only when **Copy Last Observation** is clicked. Never included in Save snapshots.

#### Non-Pharmacologic
- **Field**: `button-group` with 11 lifestyle buttons in `#treatment-life-style-btns` (inside the Treatment Given section — note: there is a separate set under the active Treatment section below)
- Each button has optional `data-*` attributes controlling conditional visibility (see [Risk-Linked Lifestyle Buttons](#risk-linked-lifestyle-buttons))

#### Pharmacologic Rows

| Field ID | Drug Class | 💊 Icon |
|---|---|---|
| `obs-oral-bg-drugs` | Oral Hypoglycemic Drugs | Yes |
| `obs-dm-insulin` | DM (Insulin) | Yes |
| `obs-htn-treatment` | HTN-Treatment | Yes |
| `obs-antiplatelet` | Antiplatelet | Yes |
| `obs-dyslipidemia` | Dyslipidemia | Yes |
| `obs-other-meds` | Other Medication | Yes |

Each row and its 💊 icon toggle visibility via `checkPharm()` — hidden when empty, shown when content exists.

Copy operation maps these to `*-copy` IDs (e.g. `obs-oral-bg-drugs` → `obs-oral-bg-drugs-copy`) in the active Treatment section.

### Drug Adherence

| Field ID | Type | Behavior |
|---|---|---|
| `obs-overall-adherence` | Read-only input | Auto-evaluated: **Good** if all MMAS-4 responses = "No"; **Poor** if any = "Yes". Color-coded green/red |

Row `row-overall-adherence` is outside the hidden `#treatment-given-section` tbody so it saves independently.

### Subjective Section

| Field ID | Type | Notes |
|---|---|---|
| `obs-symptoms` | Textarea | Recent Complaint |
| `search-risks` | Autocomplete + pills | Multi-select risk factors from `dictConcepts.risks`. Pills in `#pill-box-risks`. Drives lifestyle button visibility |
| Pregnancy buttons | Button group | Shown only for females aged 15–45 (`row-pregnant`). Selecting "No" reveals conceive planning row (`row-conceive`) |

### Objective Section

#### Vitals

| Field ID | Type | Auto-Calculation |
|---|---|---|---|
| `obs-sbp` | Integer input (text mode) | HTN Grade + HTN Status + Pediatric BP |
| `obs-dbp` | Integer input (text mode) | HTN Grade + HTN Status + Pediatric BP |
| `obs-pulse` | Integer input (text mode) | Pulse volume/rhythm row visibility |
| `obs-weight` | Integer input (text mode) | BMI + WHO Grading |
| `obs-height` | Integer input (text mode) | BMI + WHO Grading + Pediatric BP (when age < 13) |
| `obs-bmi` | Read-only (hidden row) | `weight(kg) / (height(m))²` |
| `obs-bmi-grading` | Read-only (hidden row) | WHO classification |
| `obs-waist` | Integer input (hidden row, age ≥ 20) | Metabolic Risk |
| `obs-metabolic-risk` | Read-only (hidden row) | Waist-based risk by gender |

#### Pertinent Physical Exam

| Trigger | `btn-group` Yes/No | Reveals exam rows on "Yes" |
|---|---|---|
| Exam fields | Textareas | Oral/Dental, Heart, Peripheral Arteries, MSK-Foot, Mental Status, Motor, Sensory |

### Pertinent Lab Ix

| Field ID | Type | Auto-Calculation |
|---|---|---|
| `obs-fbs` | Integer input | DM Status + Ketone trigger (≥250 mg/dL) |
| `obs-rbs` | Integer input | DM Status + Ketone trigger (≥350 mg/dL) |
| `obs-hga1c` | Integer input | DM Status |
| `obs-ketone` | Input (hidden row) | Shown when FBS ≥ 250 or RBS ≥ 350 |
| `obs-albumin` | Select (Nil/ Trace/ 1+– 4+) | 24-hr Urine Protein row shown on 2+/3+/4+ |
| `obs-micro` | Textarea | Microscopic findings |
| `obs-urine-prof` | Input (hidden row) | 24-hr Urine Protein |
| `obs-creatinine` | Integer input | GFR + KDIGO Category |
| `obs-gfr` | Read-only (hidden row) | 2021 CKD-EPI equation |
| `obs-gfr-kdigo` | Read-only (hidden row) | G1–G5 with color coding |
| `obs-na` | Integer input | — |
| `obs-k` | Integer input | — |
| `obs-triglyceride` | Integer input | — |
| `obs-ldl` | Integer input | — |
| `obs-ecg` | Textarea | — |
| `obs-echo` | Textarea | — |
| `obs-fundoscopic` | Textarea | — |
| `obs-other-investigation` | Textarea | — |
| `obs-overall-assessment` | Textarea | — |

### Disease Outcome

| Field ID | Type | Visibility |
|---|---|---|---|
| `dm-status` | Select (Controlled/Uncontrolled) | Hidden row, shown when FBS/RBS/HgA1c has data |
| `htn-status` | Select (Controlled/Uncontrolled) | Hidden row, shown when SBP/DBP has data |
| `obs-complic-status` | Select (Same/Corrected/Controlled/Uncontrolled/Unknown) | Shown only when complication pills exist |
| `obs-comorb-status` | Select (Same/Corrected/Controlled/Uncontrolled/Unknown) | Shown only when comorbidity pills exist |

### Treatment (Active Plan)

This section contains the **current encounter treatment plan**. It is always visible.

#### Non-Pharmacologic
- Buttons in `#treatment-life-style-btns` with conditional visibility via `data-risks`, `data-min-bmi`, `data-show-on-adherence`, `data-show-on-field`, `data-show-on-dm`, `data-show-on-comorb` attributes

#### Pharmacologic (Copy fields)
| Field ID | Source |
|---|---|
| `obs-oral-bg-drugs-copy` | Copied from `obs-oral-bg-drugs` |
| `obs-dm-insulin-copy` | Copied from `obs-dm-insulin` |
| `obs-htn-treatment-copy` | Copied from `obs-htn-treatment` |
| `obs-antiplatelet-copy` | Copied from `obs-antiplatelet` |
| `obs-dyslipidemia-copy` | Copied from `obs-dyslipidemia` |
| `obs-other-meds-copy` | Copied from `obs-other-meds` |

#### Linked Entries (conditional)
| Parent Field | Child Row | Child Field |
|---|---|---|
| `obs-linked-to` | `row-linkage-note` | `obs-linkage-note` |
| `obs-consultation-to` | `row-consultation-note` | `obs-consultation-note` |
| `obs-referral-to` | `row-referral-reason` | `obs-referral-reason` |

| Field ID | Type |
|---|---|
| `obs-remark` | Textarea |

### Appointment

| Field ID | Type | Behavior |
|---|---|---|
| `obs-appointment` | Read-only (Ethiopian calendar) | Displays selected date in E.C. format |
| `obs-appointment-gregorian` | Hidden | Stores Gregorian equivalent for Bahmni |
| Picker | Custom calendar widget | Restricts to today + future dates only. Saturdays, Sundays, and Ethiopian full holidays are disabled |

---

## Behavioral Rules

### Auto-Calculations

| Function | Trigger | Output(s) |
|---|---|---|
| `calculateHTNGrade()` | SBP or DBP or Height input | `obs-htn-grade`: **Adult (≥13):** Normal / Grade-1 / Grade-2 / Grade-3 / Hypotensive. **Pediatric (<13):** Pediatric: Normal / Elevated / Stage 1 HTN / Stage 2 HTN / Hypotensive (with height-adjusted percentile thresholds) |
| `autoCalculateHTNStatus()` | SBP or DBP input | `htn-status`: Controlled / Uncontrolled (pediatric: Normal+Elevated → Controlled, Hypotensive+Stage1+Stage2 → Uncontrolled) |
| `calculateHeightZScore()` | Height, age, sex | Height-for-age z-score using CDC LMS reference data |
| `calculatePediatricBPThresholds()` | Age, sex, height | 90th and 95th percentile SBP/DBP thresholds interpolated from AAP 2017 normative tables |
| `classifyPediatricBP()` | SBP, DBP, thresholds | Pediatric BP category: Normal / Elevated / Stage 1 HTN / Stage 2 HTN / Hypotensive |
| `autoCalculateDMStatus()` | FBS, RBS, or HgA1c input | `dm-status`: Controlled or Uncontrolled |
| `autoCalculateBMI()` | Weight or Height input | `obs-bmi` + `obs-bmi-grading` (WHO 7-class) |
| `autoCalculateWaistRisk()` | Waist input | `obs-metabolic-risk`: Normal / Increased / Greatly Increased (by gender) |
| `autoCalculateGFR()` | Creatinine input | `obs-gfr` (2021 CKD-EPI) + `obs-gfr-kdigo` (G1–G5) |
| `calculateOverallAdherence()` | MMAS-4 modal save | `obs-overall-adherence`: Good / Poor |

### Validation Constraints

Numeric fields validate on blur. Out-of-range values trigger a red border + inline error message.

| Field | Range | Notes |
|---|---|---|
| Systolic BP | 50 – 300 | `obs-sbp` |
| Diastolic BP | 30 – 180 | `obs-dbp` |
| Pulse Rate | 30 – 200 | `obs-pulse` |
| Weight | 5 – 250 | `obs-weight` |
| Height | 50 – 250 | `obs-height` |
| Waist Circumference | 50 – 180 | `obs-waist` |
| FBS | 30 – 600 | `obs-fbs` |
| RBS | 30 – 600 | `obs-rbs` |
| HgA1c | 4 – 20 | `obs-hga1c` |
| Creatinine | 0.05 – 15 | `obs-creatinine` (decimal) |
| Na+ | 70 – 200 | `obs-na` |
| K+ | 1 – 10 | `obs-k` |

### Conditional Visibility

| Element | Condition |
|---|---|
| `row-htn-grade` | SBP or DBP has a value |
| `row-peds-bp-result` | Age < 13 + SBP + DBP + Height all have values (shows classification badge + 90th/95th thresholds + height z-score) |
| `row-pulse-volume`, `row-pulse-rythm` | Pulse rate has a positive integer |
| `row-bmi`, `row-bmi-zscore` | Both weight and height have values |
| `row-waist-entry` | Patient age ≥ 20 (evaluated on DOM load) |
| `row-metabolic-risk` | Waist has a value |
| `row-urine-ketone` | FBS ≥ 250 or RBS ≥ 350 |
| `row-urine-protein` | Albumin = 2+, 3+, or 4+ |
| `row-gfr`, `row-gfr-kdigo` | Creatinine has a value |
| `row-dm-status` | FBS, RBS, or HgA1c has a value |
| `row-htn-status` | SBP or DBP has a value |
| `row-complic-status` | Any complication pills exist in `#pill-box-complic` |
| `row-comorb-status` | Any comorbidity pills exist in `#pill-box-comorb` |
| `row-pregnant` | Gender = F and age 15–45 |
| `row-conceive` | Pregnant answer = "No" |
| `row-overall-adherence` | Any pharmacologic field has content |
| `row-linked-to`, `row-linkage-note` | `obs-linked-to` has content |
| `row-consultation-to`, `row-consultation-note` | `obs-consultation-to` has content |
| `row-referral-to`, `row-referral-reason` | `obs-referral-to` has content |
| `pertinent-exam-row` (7 rows) | P/E finding = "Yes" |
| Pharmacologic rows + 💊 icons | Respective textarea has content (via `checkPharm()`) |
| `#treatment-given-section` | Hidden by default; shown on Copy Last Observation |

### Risk-Linked Lifestyle Buttons

In the active Treatment section (`#treatment-life-style-btns`), buttons with class `risk-linked` conditionally show/hide based on:

| Data Attribute | Logic |
|---|---|
| `data-risks` | Show if any listed risk factor pill exists in `#pill-box-risks` |
| `data-min-bmi` | Show if BMI ≥ threshold |
| `data-show-on-adherence` | Show if adherence value matches (e.g. "Poor") |
| `data-show-on-field` | Show if specified field ID has content |
| `data-show-on-dm` | Show if DM type matches |
| `data-show-on-comorb` | Show if any listed condition pill exists in `#pill-box-comorb` or `#pill-box-complic` |

Buttons hide automatically when conditions are no longer met.

### MMAS-4 Adherence Modal

Opened by clicking any 💊 icon next to a pharmacologic textarea.

**Workflow:**
1. Select drug name (auto-populated from the row)
2. Answer "Missed Dose?" Yes/No
3. If Yes → answer 4 MMAS-4 questions (Forgetfulness, Carelessness, Feeling Better, Feeling Worse)
4. Save → icon turns green ✅, `adherenceState` updated
5. `calculateOverallAdherence()` sets Drug Adherence to **Good** (all No) or **Poor** (any Yes)

---

## Save / Snapshot System

`saveFormSnapshot()` creates a **column-based snapshot** in the DOM.

| Aspect | Detail |
|---|---|
| **Session ID** | `save-session-{timestamp}` — reused within 12-hour window for updates |
| **Header** | Appends `<td class="saved-cell">` with formatted timestamp to the header row |
| **Per-row value** | Reads `.input-cell` content: pills → button selections → input/textarea/select values |
| **Exclusion** | Rows inside `#treatment-given-section` are skipped |
| **Status** | "Form saved successfully at …" message shown for 4 seconds |

Multiple snapshots create multiple saved columns side-by-side.

---

## Copy Last Observation

`copyLastObservation()` restores the **most recent saved snapshot** into the Now column.

### Skip List

The following fields are **never overwritten** by Copy (identified by field ID):

- **Vitals**: `obs-sbp`, `obs-dbp`, `obs-weight`
- **Subjective**: `obs-symptoms`
- **Labs**: `obs-fbs`, `obs-rbs`, `obs-hga1c`, `obs-ketone`, `obs-albumin`, `obs-micro`, `obs-urine-prof`, `obs-creatinine`, `obs-gfr`, `obs-gfr-kdigo`, `obs-na`, `obs-k`, `obs-triglyceride`, `obs-ldl`, `obs-ecg`, `obs-echo`, `obs-fundoscopic`, `obs-other-investigation`, `obs-overall-assessment`
- **Admin**: `obs-appointment`, `obs-overall-adherence`

### Treatment Given population

On Copy:
1. Hidden `#treatment-given-section` is shown (`display: ''`)
2. Medications are copied from the snapshot into the Treatment Given pharmacologic textareas
3. Lifestyle buttons in Treatment Given are selected and hidden buttons are set to `display: none`
4. `checkPharm()` runs to sync pharmacologic row/icon visibility

---

## Reset (Add New)

`resetNowColumn()` clears the entire Now column:

- All inputs, textareas, selects reset to empty
- All pills removed
- All button selections cleared
- `#treatment-given-section` hidden
- Appointment field and note reset
- `updateLifestyleButtons()` re-evaluated

---

## Integer-Only Input Enforcement

14 numeric fields enforce integer-only input via `allowOnlyInteger()`:

| Field ID |
|---|
| `obs-sbp` |
| `obs-dbp` |
| `obs-pulse` |
| `obs-weight` |
| `obs-height` |
| `obs-waist` |
| `obs-fbs` |
| `obs-rbs` |
| `obs-hga1c` |
| `obs-creatinine` |
| `obs-na` |
| `obs-k` |
| `obs-triglyceride` |
| `obs-ldl` |

**Implementation**: `oninput` handler strips all non-digit characters using `input.value.replace(/[^0-9]/g, '')`. Uses `type="text"` (not `type="number"`) to avoid cross-browser spinner inconsistencies.

---

## Ethiopian Date Picker

| Feature | Implementation |
|---|---|
| **Months** | 13 Ethiopian months (Meskerem–Pagume) |
| **Year range** | Defaults to current Ethiopian year; navigable forward only |
| **Past dates** | Disabled — cannot select before today |
| **Weekends** | Saturday and Sunday disabled |
| **Full holidays** | 8 fixed Ethiopian holidays + 5 configured movable Gregorian holidays; disabled with red styling |
| **Output** | `obs-appointment`: formatted string (e.g. "Tikimt 12, 2017 E.C.") |
| **Hidden output** | `obs-appointment-gregorian`: ISO date string for Bahmni |

---

## Pediatric BP Classification (AAP 2017)

When `patientDemographics.age < 13`, the form switches from adult HTN grading to the **2017 AAP pediatric BP classification system**, which requires the child's sex, exact age, height, and blood pressure to compute percentile-based thresholds.

### CDC Stature-for-Age LMS Data

Embedded `CDC_STATURE_LMS` object provides L, M, S parameters for stature-for-age at yearly intervals (ages 1–13) for both sexes, sourced from the CDC 2000 growth charts. The `calculateHeightZScore(age, sex, height_cm)` function interpolates between yearly entries and applies the LMS formula:

```
Z = ((X / M)^L - 1) / (L × S)
```

Handles the edge case where L ≈ 0 using the logarithmic form.

### AAP 2017 Normative BP Tables

Embedded `AAP_BP` lookup tables provide systolic and diastolic BP values at the 50th, 90th, and 95th percentiles for ages 1–12, both sexes, across 7 height percentile buckets (5th–95th). The `getPediatricBPThresholds()` function:

1. Converts the height Z-score → nearest height percentile bucket
2. Looks up BP thresholds from the embedded AAP tables
3. Interpolates between adjacent height percentile entries for accuracy

### Classification Categories

| Category | Criteria | Badge Color |
|---|---|---|
| **Normal** | SBP and DBP both < 90th percentile | Green (`#5cb85c`) |
| **Elevated** | SBP or DBP ≥ 90th but < 95th percentile, or ≥ 120/80 but still < 95th percentile | Yellow (`#f0ad4e`) |
| **Stage 1 HTN** | SBP or DBP ≥ 95th percentile but < 95th + 12 mmHg, or 130/80–139/89 | Orange (`#e67e22`) |
| **Stage 2 HTN** | SBP or DBP ≥ 95th + 12 mmHg, or ≥ 140/90 | Red (`#d9534f`) |
| **Hypotensive** | SBP or DBP below estimated 5th percentile (2×p50 − p95) or age-based absolute minimum (70 + age×2, capped at 90) | Cyan (`#5bc0de`) |

The higher severity between SBP and DBP determines the final category. A **Pediatric BP Classification** row (`row-peds-bp-result`) appears below the DBP input showing the color-coded badge, computed 90th/95th percentile thresholds, and the height Z-score.

### Age Cutoff

| Age | System | Threshold Source |
|---|---|---|
| < 13 | AAP 2017 pediatric classification | Percentile tables (age/sex/height adjusted) |
| ≥ 13 | Adult HTN grading | Fixed cutoffs (140/90, 160/100, 180/110) |

### Test Demographics

An in-form **Test Demographics** section (above Diagnosis Categories) provides Age and Sex inputs for easy testing of both pathways without using the browser console. Changing the age to < 13 automatically switches to pediatric BP logic.

---

## Hypotensive Detection

Hypotensive classification is applied to **both** age groups:

| Group | Criteria |
|---|---|
| **Pediatric (< 13 yr)** | SBP < estimated 5th percentile (`2 × p50 − p95`) **or** SBP < age-based minimum (`70 + age × 2`, capped at 90 for age > 10) **or** DBP < estimated 5th percentile **or** DBP < 50 |
| **Adult (≥ 13 yr)** | SBP < 90 **or** DBP < 60 |

In `calculateHTNGrade()`, hypotensive is shown as a distinct label in the `obs-htn-grade` field. In `autoCalculateHTNStatus()`, hypotension maps to **Uncontrolled**.

---

## Mobile Responsiveness

| Breakpoint | Layout |
|---|---|
| > 768 px | Standard table layout with 200px label + 160px saved + auto input columns |
| ≤ 768 px | Reduced padding, narrower cells, smaller inputs |
| ≤ 480 px | **Stacked card layout**: each table row becomes a bordered block with label as full-width header, input and saved cells below as block elements. Buttons, pills, and inputs expand to 100% width |

---

## Integration Guide for Bahmni EMR

### Concept Mapping

Each field ID (`obs-*`) maps to an OpenMRS concept. Below is the recommended mapping:

| Field ID | OpenMRS Concept (Suggested UUID) | Datatype |
|---|---|---|
| `search-dm` | Type of DM | Coded |
| `search-htn` | Type of HTN | Coded |
| `obs-htn-grade` | HTN Grade | Text |
| `search-complic` | Complication(s) | Multi-select Coded |
| `search-comorb` | Comorbidity | Multi-select Coded |
| `obs-oral-bg-drugs` | Oral Hypoglycemic Drugs | Text |
| `obs-dm-insulin` | DM (Insulin) | Text |
| `obs-htn-treatment` | HTN Treatment | Text |
| `obs-antiplatelet` | Antiplatelet | Text |
| `obs-dyslipidemia` | Dyslipidemia | Text |
| `obs-other-meds` | Other Medication | Text |
| `obs-overall-adherence` | Drug Adherence | Text |
| `obs-symptoms` | Recent Complaint | Text |
| `search-risks` | Recent Risk Factors | Multi-select Coded |
| `obs-sbp` | Systolic Blood Pressure | Numeric |
| `obs-dbp` | Diastolic Blood Pressure | Numeric |
| `obs-pulse` | Pulse Rate | Numeric |
| `obs-weight` | Weight | Numeric |
| `obs-height` | Height | Numeric |
| `obs-bmi` | BMI | Calculated |
| `obs-bmi-grading` | BMI Grading (WHO) | Text |
| `obs-waist` | Waist Circumference | Numeric |
| `obs-metabolic-risk` | Metabolic Risk | Text |
| `obs-fbs` | FBS | Numeric |
| `obs-rbs` | RBS | Numeric |
| `obs-hga1c` | HgA1c | Numeric |
| `obs-ketone` | Urine Ketone | Text |
| `obs-albumin` | Urine Albumin (Protein) | Coded |
| `obs-micro` | Urine Microscopic Findings | Text |
| `obs-urine-prof` | 24-hr Urine Protein | Numeric |
| `obs-creatinine` | Creatinine | Numeric |
| `obs-gfr` | GFR | Calculated |
| `obs-gfr-kdigo` | KDIGO Category | Text |
| `obs-na` | Sodium | Numeric |
| `obs-k` | Potassium | Numeric |
| `obs-triglyceride` | Triglyceride | Numeric |
| `obs-ldl` | LDL | Numeric |
| `obs-ecg` | ECG | Text |
| `obs-echo` | ECHO | Text |
| `obs-fundoscopic` | Fundoscopic Finding | Text |
| `obs-other-investigation` | Other Investigation | Text |
| `obs-overall-assessment` | Overall Assessment | Text |
| `dm-status` | DM Status | Coded |
| `htn-status` | HTN Status | Coded |
| `obs-complic-status` | Complication Status | Coded |
| `obs-comorb-status` | Comorbidity Status | Coded |
| `obs-linked-to` | Linked To | Text |
| `obs-linkage-note` | Linkage Note | Text |
| `obs-consultation-to` | Consultation To | Text |
| `obs-consultation-note` | Consultation Note | Text |
| `obs-referral-to` | Referral To | Text |
| `obs-referral-reason` | Referral Reason | Text |
| `obs-remark` | Remark | Text |
| `obs-appointment` | Appointment Date | Date (Ethiopian) |

### Conversion Workflow

1. Replace all field `id` and `name` attributes with Bahmni's `concept` naming convention
2. Replace `dictConcepts` mock with actual OpenMRS concept search API calls (REST or `obs.concept`)
3. Replace `patientDemographics` mock with actual Bahmni patient context (`$patient` in velocity template)
4. Remove inline JS and bundle as external resource if needed
5. Convert the Ethiopian date picker to use Bahmni's `datepicker` directive with Ethiopian calendar support
6. Wrap the form in a velocity template (`htmlFormEntry` or `ampath` format)
7. Map pill-based multi-select to Bahmni's multi-obs group pattern
8. Re-implement the Save snapshot system as Bahmni form submission (obs save via REST)
9. The MMAS-4 adherence data is not stored as structured obs — consider adding concept mappings for the 4 questions if structured reporting is needed

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

---

*Maintained by ThessalK. Prototype for Bahmni HTML Form Entry conversion.*
