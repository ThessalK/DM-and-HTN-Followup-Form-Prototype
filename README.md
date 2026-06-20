# DM & HTN Follow-up Form — Bahmni HTML Form Entry Prototype

<p align="center">
  <img src="https://img.shields.io/badge/status-prototype-yellow" alt="Status">
  <img src="https://img.shields.io/badge/license-MIT-blue" alt="License">
  <img src="https://img.shields.io/badge/stack-HTML%20%7C%20CSS%20%7C%20ES6-green" alt="Stack">
  <img src="https://img.shields.io/badge/EC-Ethiopian%20Calendar-orange" alt="Ethiopian Calendar">
  <img src="https://img.shields.io/badge/Bahmni-EMR%20Ready-brightgreen" alt="Bahmni Ready">
  <img src="https://img.shields.io/badge/WHO%202019-CVD%20Risk-important" alt="WHO CVD Risk">
</p>

A production-oriented, mobile-responsive clinical follow-up form for **Diabetes Mellitus (DM)** and **Hypertension (HTN)** patients — built as a standalone HTML/CSS/ES6 single-page application and engineered for direct conversion to a **Bahmni HTML Form Entry** template with full OpenMRS concept mappings.

> **Target audience**: Clinical teams deploying into Bahmni EMR environments, health informaticians, and developers working on Ethiopian-standard DM/HTN follow-up workflows.

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
11. [CVD Risk (WHO 2019 revised)](#cvd-risk-who-2019-revised)
12. [Mobile Responsiveness](#mobile-responsiveness)
13. [Integration Guide for Bahmni EMR](#integration-guide-for-bahmni-emr)
14. [Developer Notes](#developer-notes)

---

## Architecture Overview

| Aspect | Detail |
|---|---|
| **Stack** | HTML5 + CSS3 + Vanilla ES6 JavaScript (no frameworks, no build tools) |
| **File** | Single file: `DM_HTN_Followup.html` (~3948 lines) |
| **State** | Client-side only (no persistence layer; save creates in-memory DOM snapshot columns) |
| **Calendar** | Ethiopian calendar (E.C.) with Gregorian conversion for the appointment picker |
| **Mock Data** | Hardcoded `dictConcepts` array simulates OpenMRS concept search |
| **Demographics** | Mock `patientDemographics` object (`{ age, gender }`) drives conditional rules. Editable via in-form **Test Demographics** section (Age input + Sex toggle buttons) for testing pediatric vs adult pathways |

---

## Form Sections & Field Reference

### Diagnosis Categories

| Field ID | Type | Behavior |
|---|---|---|
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
| `obs-pregnant` | Hidden `<input type="hidden">` + Button group | Shown only for females aged 15–45 (`row-pregnant`). Selecting "No" reveals conceive planning row (`row-conceive`). Value synced to hidden field for OpenMRS concept mapping |

### Objective Section

#### Vitals

| Field ID | Type | Auto-Calculation |
|---|---|---|
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
| `obs-creatinine` | Decimal input (allowDecimalInput) | GFR + KDIGO Category |
| `obs-gfr` | Read-only (hidden row) | 2021 CKD-EPI equation |
| `obs-gfr-kdigo` | Read-only (hidden row) | G1–G5 with color coding |
| `obs-na` | Integer input | — |
| `obs-k` | Integer input | — |
| `obs-total-cholesterol` | Integer input | CVD Risk (lab mode) |
| `obs-triglyceride` | Integer input | — |
| `obs-ldl` | Integer input | — |
| `obs-cvd-risk` | Read-only badge (hidden row) | WHO 2019 revised CVD Risk (AFR-E) — shown when dyslipidemia-copy has value |
| `obs-ecg` | Textarea | — |
| `obs-echo` | Textarea | — |
| `obs-fundoscopic` | Textarea | — |
| `obs-other-investigation` | Textarea | — |
| `obs-overall-assessment` | Textarea | — |

### Disease Outcome

| Field ID | Type | Visibility |
|---|---|---|
| `dm-status` | Select (Controlled/Uncontrolled) | Hidden row, shown when FBS/RBS/HgA1c has data |
| `htn-status` | Select (Controlled/Uncontrolled) | Hidden row, shown when SBP/DBP has data |
| `obs-dyslipidemia-status` | Select (Controlled/Uncontrolled/Unknown) | Auto-computed from LDL when dyslipidemia-copy has entry. Hidden row, shown when dyslipidemia-copy has data |
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
| `obs-appointment-gregorian` | Hidden | Gregorian backing-store for Bahmni's existing appointment scheduling concept; maps directly to the OpenMRS appointment date-time obs |
| Picker | Custom calendar widget | Restricts to today + future dates only. Saturdays, Sundays, and Ethiopian full holidays are disabled |

**Bahmni Integration — Appointment Scheduling Concept Mapping**

This form integrates with Bahmni's pre-existing Appointment Scheduling module, which provides a dedicated concept and API for follow-up visit tracking out of the box. The `obs-appointment-gregorian` hidden field is the integration point: it stores the ISO-formatted Gregorian date that maps directly to the standard OpenMRS appointment obs concept, enabling seamless interoperability with Bahmni's scheduling infrastructure without custom extension.

| Capability | Bahmni Native Integration |
|---|---|
| **Follow-up visit creation** | On form save, submit `obs-appointment-gregorian` to the OpenMRS appointment API to create or update a scheduled appointment for the patient. The field already aligns with Bahmni's existing `Visit Appointment` concept for out-of-the-box mapping |
| **Automated reminders** | Bahmni's built-in SMS/notification service triggers automatically from the scheduled appointment date — no additional wiring required |
| **Slot availability & quota enforcement** | Reads the configured `maxAppointmentsPerDate` from the Appointment Service concept. On date selection, queries Bahmni's appointment REST endpoint (`/rest/v1/appointment/all?forDate=...&serviceUuid=...`) to retrieve the current booking count; displays real-time availability (e.g., "3 of 10 slots booked" with a color-coded indicator). Prevents booking when the daily quota is exhausted and surfaces an inline warning: *"This appointment service has reached its daily capacity. Please select an alternative date."* |
| **Bi-directional sync** | On form load, pre-populate `obs-appointment` from the patient's existing scheduled appointment (if any) via Bahmni's appointment REST endpoint; update on save keeps both sides in sync |
| **Conflict detection** | Bahmni's scheduling engine inherently cross-references provider availability and existing appointments before committing |
| **Calendar integration** | In production, replace the prototype's standalone Ethiopian picker with Bahmni's native `datepicker` directive, which already supports the Ethiopian calendar and the appointment scheduling context natively |

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
| `calculateCVDRisk()` | SBP, TC, weight, height, DM, smoking, dyslipidemia-copy, demographics | `row-cvd-risk` badge: <5% (green) / 5% to <10% (yellow) / 10% to <20% (orange) / 20% to <30% (red) / ≥30% (dark red) |
| `autoCalculateDyslipidemiaStatus()` | dyslipidemia-copy, LDL | `obs-dyslipidemia-status`: Controlled (LDL<70) / Uncontrolled (LDL≥70) / Unknown (no LDL) |

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
| Total Cholesterol | 50 – 500 | `obs-total-cholesterol` |

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
| `row-dyslipidemia-status` | `obs-dyslipidemia-copy` has a value (treatment section dyslipidemia entry). Auto-hides when cleared |
| `row-pregnant` | Gender = F and age 15–45 (re-evaluated dynamically when age or sex changes via `syncDemographics()` / `setDemoSex()`) |
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
- **Labs**: `obs-fbs`, `obs-rbs`, `obs-hga1c`, `obs-ketone`, `obs-albumin`, `obs-micro`, `obs-urine-prof`, `obs-creatinine`, `obs-gfr`, `obs-gfr-kdigo`, `obs-na`, `obs-k`, `obs-total-cholesterol`, `obs-triglyceride`, `obs-ldl`, `obs-ecg`, `obs-echo`, `obs-fundoscopic`, `obs-other-investigation`, `obs-overall-assessment`
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
| `obs-total-cholesterol` |
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

## CVD Risk (WHO 2019 revised)

The form computes a 10-year cardiovascular disease risk estimate using the **2019 revised WHO cardiovascular disease risk charts** for **Eastern Sub-Saharan Africa**. This uses a lookup-table-based approach for both Laboratory and Non-Laboratory models. 

### Display

A **CVD Risk** row (`row-cvd-risk`) appears below the LDL entry in the *Pertinent Lab Ix* section, showing a color-coded badge:

| Risk Level | Badge Color |
|---|---|
| <5% | Green (`#4CAF50`) |
| 5% to <10% | Yellow (`#FFC107`) |
| 10% to <20% | Orange (`#FF9800`) |
| 20% to <30% | Red (`#F44336`) |
| ≥30% | Dark Red (`#B71C1C`) |

### Visibility

The row is hidden by default and appears only when the **Dyslipidemia** textarea (`obs-dyslipidemia-copy`) in the Treatment (Active Plan) section is **empty**. When dyslipidemia-copy has a value, CVD Risk is hidden and the **Dyslipidemia** Disease Outcome row is shown instead (auto-computed from LDL).

### Inputs and Modes

The calculator supports two modes:

| Mode | Condition | Additional Inputs |
|---|---|---|
| **Lab-based** | Total Cholesterol (`obs-total-cholesterol`) has a value ≥ 1 | TC value mapped to one of 5 cholesterol categories for lookup |
| **Non-lab (BMI-based)** | Total Cholesterol is empty or 0 | Uses BMI (computed from weight & height) with the non-lab (chol=0) lookup column |

**Common inputs** (required for both modes):

| Input | Source | Notes |
|---|---|---|
| Age | `patientDemographics.age` | Must be 40–74 (lookup table range). Use Test Demographics section |
| Sex | `patientDemographics.gender` | M = Male, F = Female. Use sex toggle |
| SBP | `obs-sbp` | Categorical: <120, 120–129, 130–149, 150–169, ≥170 |
| Diabetes | `search-dm` = "Type 2 Diabetes Mellitus" | dm=1 if exact match, else dm=0 |
| Smoking | `#pill-box-risks` pills | Smoker if any pill text contains "tobacco" or "smok" (case-insensitive) |

### Lookup Tables

The form utilizes three JavaScript arrays acting as lookup tables:
- `WHO_2019_NONLAB_AFRE`: Non-laboratory model (uses BMI)
- `WHO_2019_LAB_NONDM_AFRE`: Laboratory model for people without diabetes
- `WHO_2019_LAB_DM_AFRE`: Laboratory model for people with diabetes

The structures are nested by **Age Group** → **SBP Category** → **Sex/Smoking/TC or BMI Index**.
Each derived cell contains an exact risk percentage which maps to a risk level (0–4) corresponding to the five band labels above.
**Cholesterol code mapping:**
| Internal Category Index | WHO Standard (`mmol/L`) | Form Input Approx (`mg/dL`) |
|---|---|---|
| **0** | `< 4.0` | `< 155` |
| **1** | `4.0 to < 5.0` | `155 to < 193` |
| **2** | `5.0 to < 6.0` | `194 to < 232` |
| **3** | `6.0 to < 7.0` | `232 to < 270` |
| **4** | `≥ 7.0` | `≥ 271` |

### Trigger Events

`calculateCVDRisk()` runs whenever any of these inputs change:
- `obs-sbp` (SBP)
- `obs-total-cholesterol` (TC — switches lab/non-lab mode)
- `obs-weight` or `obs-height` (BMI for non-lab mode)
- `search-dm` (diabetes status)
- `pill-box-risks` pill add/remove (smoking status)
- `obs-dyslipidemia-copy` (visibility gate — show CVD Risk when empty, show Dyslipidemia Status when filled)
- Demographics age/sex change
- `obs-ldl` (triggers both `autoCalculateDyslipidemiaStatus()` and `calculateCVDRisk()`)
- `obs-dyslipidemia` (clearing dyslipidemia re-shows CVD Risk row)

Additionally, `autoCalculateDyslipidemiaStatus()` runs directly on `obs-ldl` changes to update the Dyslipidemia Disease Outcome status in real time.

---

## Mobile Responsiveness

| Breakpoint | Layout |
|---|---|
| > 768 px | Standard table layout with 200px label + 160px saved + auto input columns |
| ≤ 768 px | Reduced padding, narrower cells, smaller inputs |
| ≤ 480 px | **Stacked card layout**: each table row becomes a bordered block with label as full-width header, input and saved cells below as block elements. Buttons, pills, and inputs expand to 100% width |

---

## Integration Guide for Bahmni EMR

This guide provides a comprehensive, step-by-step pathway for deploying this DM/HTN Follow-up form into a production Bahmni EMR environment without losing any functionality. It covers both the **full-custom** approach (Custom Display Control, maximum fidelity) and the **rapid-deployment** approach (Standard HTML Form Entry + `form-conditions.js`).

---

### Architecture Overview: Deployment Options

| Approach | Effort | Preserves UI/UX | Recommended For |
|---|---|---|---|
| **A. Custom Display Control** (Angular Directive) | Medium-High | Full — badges, pills, glassmorphism | Teams wanting pixel-perfect parity with the prototype |
| **B. Standard HTML Form Entry** (Velocity Template + `form-conditions.js`) | Low-Medium | Logic preserved, simplified UI | Teams needing rapid deployment with auto-calculations |
| **C. Bahmni Form Builder** (GUI) | Low | Minimal — only basic inputs | Quick prototyping, not recommended for production |

> **Recommendation**: Use **Option A** for the final production deployment. Use **Option B** as an interim step to validate clinical logic in Bahmni before committing to the full custom build.

---

### Phase 1: Concept Mapping and Metadata Setup

Before writing any code, every `obs-*` field must be registered in OpenMRS as a concept. Below is the complete mapping table with recommended datatypes and answer sets.

#### Core Observation Concepts

| Field ID | Recommended Concept Name | Datatype | Units / Answer Set |
|---|---|---|---|
| `obs-sbp` | Systolic Blood Pressure | Numeric | mmHg |
| `obs-dbp` | Diastolic Blood Pressure | Numeric | mmHg |
| `obs-pulse` | Pulse Rate | Numeric | bpm |
| `obs-weight` | Weight | Numeric | kg |
| `obs-height` | Height | Numeric | cm |
| `obs-bmi` | BMI | Numeric | Calculated |
| `obs-bmi-grading` | BMI Grading (WHO) | Text | — |
| `obs-waist` | Waist Circumference | Numeric | cm |
| `obs-metabolic-risk` | Metabolic Risk (Waist) | Text | — |
| `obs-fbs` | Fasting Blood Sugar | Numeric | mg/dL |
| `obs-rbs` | Random Blood Sugar | Numeric | mg/dL |
| `obs-hga1c` | Hemoglobin A1c | Numeric | % |
| `obs-ketone` | Urine Ketone | Text | — |
| `obs-albumin` | Urine Albumin | Coded | Nil, Trace, 1+, 2+, 3+, 4+ |
| `obs-micro` | Urine Microscopic Findings | Text | — |
| `obs-urine-prof` | 24hr Urine Protein | Numeric | mg/24h |
| `obs-creatinine` | Serum Creatinine | Numeric | mg/dL |
| `obs-gfr` | eGFR (CKD-EPI 2021) | Numeric | Calculated |
| `obs-gfr-kdigo` | KDIGO Category | Text | — |
| `obs-na` | Serum Sodium | Numeric | mEq/L |
| `obs-k` | Serum Potassium | Numeric | mEq/L |
| `obs-total-cholesterol` | Total Cholesterol | Numeric | mg/dL |
| `obs-triglyceride` | Triglyceride | Numeric | mg/dL |
| `obs-ldl` | LDL Cholesterol | Numeric | mg/dL |
| `obs-ecg` | ECG Finding | Text | — |
| `obs-echo` | ECHO Finding | Text | — |
| `obs-fundoscopic` | Fundoscopic Finding | Text | — |
| `obs-other-investigation` | Other Investigation | Text | — |
| `obs-overall-assessment` | Overall Assessment | Text | — |
| `obs-symptoms` | Recent Complaint | Text | — |
| `obs-htn-grade` | HTN Grade | Text | — |
| `obs-cvd-risk` | CVD Risk (WHO 2019) | Numeric | Calculated — percentage |
| `obs-cvd-risk-badge` | CVD Risk Level | Text | Calculated — "<5%", "5% to <10%", etc. |
| `obs-overall-adherence` | Drug Adherence | Text | Good / Poor |
| `obs-pregnant` | Pregnancy Status | Coded | Yes, No |
| `obs-conceive` | Planning to Conceive | Coded | Yes, No |

#### Coded / Multi-Select Concepts

| Field ID | Recommended Concept Name | Datatype | Answer Set / Concept Set |
|---|---|---|---|
| `search-dm` | Type of Diabetes Mellitus | Coded | Type 1 DM, Type 2 DM, Gestational, MODY, Other Secondaries |
| `search-htn` | Type of Hypertension | Coded | Essential, Secondary |
| `search-complic` | DM/HTN Complications | Multi-select Coded | Concept set: Diabetic Retinopathy, Diabetic Neuropathy, CKD, HHD, etc. |
| `search-comorb` | Comorbidities | Multi-select Coded | Concept set: BPH, Gout, Asthma, COPD, etc. |
| `search-risks` | Cardiovascular Risk Factors | Multi-select Coded | Concept set: Tobacco Use, Physical Inactivity, Excessive Alcohol, Poor Diet, Khat |
| `dm-status` | DM Status | Coded | Controlled, Uncontrolled |
| `htn-status` | HTN Status | Coded | Controlled, Uncontrolled |
| `obs-dyslipidemia-status` | Dyslipidemia Status | Coded | Controlled, Uncontrolled, Unknown |
| `obs-complic-status` | Complication Status | Coded | Same, Corrected, Controlled, Uncontrolled, Unknown |
| `obs-comorb-status` | Comorbidity Status | Coded | Same, Corrected, Controlled, Uncontrolled, Unknown |

#### Medication / Free-Text Concepts

| Field ID | Recommended Concept Name | Datatype | Notes |
|---|---|---|---|
| `obs-oral-bg-drugs` | Oral Hypoglycemic Drugs | Text | Same concept used for `*-copy` fields |
| `obs-dm-insulin` | Insulin Regimen | Text | Same concept used for `*-copy` fields |
| `obs-htn-treatment` | HTN Treatment | Text | Same concept used for `*-copy` fields |
| `obs-antiplatelet` | Antiplatelet Therapy | Text | Same concept used for `*-copy` fields |
| `obs-dyslipidemia` | Dyslipidemia Treatment | Text | Same concept used for `*-copy` fields |
| `obs-other-meds` | Other Medications | Text | Same concept used for `*-copy` fields |
| `obs-linked-to` | Linked To | Text | — |
| `obs-linkage-note` | Linkage Note | Text | — |
| `obs-consultation-to` | Consultation Referral | Text | — |
| `obs-consultation-note` | Consultation Note | Text | — |
| `obs-referral-to` | Referral To | Text | — |
| `obs-referral-reason` | Referral Reason | Text | — |
| `obs-remark` | Clinical Remark | Text | — |
| `obs-appointment` | Follow-up Appointment Date | Date | Ethiopian calendar with Gregorian backing-store |

> **Record the UUID** assigned by OpenMRS for each concept. These UUIDs are referenced in the form template and `form-conditions.js`.

> **Tip for `*-copy` fields**: Medication copy fields (e.g., `obs-oral-bg-drugs-copy`) share the **same concept UUID** as their source. Bahmni distinguishes them by section context (Treatment Given vs. Active Plan). No separate concept registration is needed.

---

### Phase 2: Code Extraction and Modularisation

The prototype is a single monolithic HTML file (~3948 lines). For Bahmni, split it into this file structure within `bahmni_config/openmrs/apps/clinical/`:

```
customDisplayControl/
├── js/
│   ├── dmHtnCalculationService.js      # All auto-calculation functions
│   ├── dmHtnCdsRules.js                # Clinical decision support (statin suggestion)
│   └── ethiopianDatePicker.js          # Ethiopian calendar (optional; prefer Bahmni native)
├── templates/
│   └── dmHtnFollowupTemplate.html      # The form HTML (AngularJS version)
├── css/
│   └── dmHtnFollowup.css               # All form-specific styles
└── dmHtnFollowup.json                  # Custom Display Control manifest

form-conditions.js                      # Condition-level CDS rules (global)
custom.css                              # Global style overrides
```

#### 2.1 Extract the Calculation Service

Create `dmHtnCalculationService.js` as an **AngularJS factory**. Move these functions into it:

| Prototype Function | Service Method | Purpose |
|---|---|---|
| `WHO_2019_NONLAB_AFRE`, `WHO_2019_LAB_DM_AFRE`, `WHO_2019_LAB_NONDM_AFRE` | `CVD_RISK_TABLES` constant | WHO 2019 lookup tables (preserve verbatim) |
| `getAgeGroup()`, `getSBPCategory()`, `getBMICategory()`, `getTCCategory()` | Used internally by `computeCvdRisk()` | Lookup helpers |
| `calculateCVDRisk()` | `computeCvdRisk(obs)` | Returns `{ risk, level, label, badgeClass }` |
| `getCVDRiskLabel()`, `getCVDBadgeClass()` | `formatCvdRisk(level)` | Returns label string + CSS class |
| `calculateHTNGrade()` | `computeHtnGrade(age, sbp, dbp, height, sex)` | Returns grade string |
| `autoCalculateBMI()` | `computeBmi(weightKg, heightCm)` | Returns `{ bmi, grading }` |
| `autoCalculateGFR()` | `computeGfr(creatinine, age, sex)` | Returns `{ gfr, kdigo, kdigoColor }` |
| `classifyPediatricBP()` | `computePediatricBp(age, sex, sbp, dbp, heightCm)` | Returns category with threshold details |
| `autoCalculateDyslipidemiaStatus()` | `computeDyslipidemiaStatus(ldl)` | Returns Controlled / Uncontrolled / Unknown |
| `autoCalculateWaistRisk()` | `computeWaistRisk(waistCm, sex)` | Returns Normal / Increased / Greatly Increased |
| `calculateHeightZScore()` | `computeHeightZScore(age, sex, heightCm)` | Returns z-score using CDC LMS |

**Example service skeleton:**

```javascript
// dmHtnCalculationService.js
angular.module('bahmni.clinical')
    .factory('dmHtnCalculationService', ['$http', function($http) {

        var CVD_RISK_TABLES = {
            NONLAB_AFRE: /* paste WHO_2019_NONLAB_AFRE from prototype */,
            LAB_DM_AFRE: /* paste WHO_2019_LAB_DM_AFRE from prototype */,
            LAB_NONDM_AFRE: /* paste WHO_2019_LAB_NONDM_AFRE from prototype */
        };

        function computeCvdRisk(obs) {
            var age = obs.patient.age;
            var sex = obs.patient.gender;
            var sbp = obs.get('obs-sbp');
            var tc = obs.get('obs-total-cholesterol');
            var dmType = obs.get('search-dm');
            var smoker = obs.hasRiskFactor('Tobacco');
            var weight = obs.get('obs-weight');
            var height = obs.get('obs-height');

            // Replicate the exact logic from prototype's calculateCVDRisk()
            // using the CVD_RISK_TABLES above.
            // ...

            return { risk: riskPct, level: level, label: label, badgeClass: badgeClass };
        }

        return {
            computeCvdRisk: computeCvdRisk,
            computeBmi: computeBmi,
            computeGfr: computeGfr,
            computeHtnGrade: computeHtnGrade,
            computePediatricBp: computePediatricBp,
            computeDyslipidemiaStatus: computeDyslipidemiaStatus,
            computeWaistRisk: computeWaistRisk
        };
    }]);
```

> **Critical**: The WHO 2019 lookup arrays must be copied **verbatim** from the prototype. These are the only source of truth. Any transcription error will produce incorrect risk scores.

#### 2.2 Extract Clinical Decision Support Rules

Move the statin suggestion and dyslipidemia auto-status logic into `form-conditions.js`:

```javascript
// form-conditions.js
// Deploy to: /var/www/bahmni_config/openmrs/apps/clinical/form-conditions.js
conditionFunctions = {

    'statin-suggestion': function(obs) {
        var age = obs.patient.age;
        var dmType = obs.get('search-dm');
        var dyslipidemia = obs.get('obs-dyslipidemia');
        var dyslipidemiaCopy = obs.get('obs-dyslipidemia-copy');
        var tc = obs.get('obs-total-cholesterol');
        var ldl = obs.get('obs-ldl');
        var cvdRisk = obs.get('obs-cvd-risk');

        var noDysTx = !dyslipidemia && !dyslipidemiaCopy;
        if (!noDysTx) return null;

        // Condition 1: Age >= 40 + Type 2 DM
        if (age >= 40 && dmType === 'Type 2 Diabetes Mellitus') {
            return 'Consider starting statin targeting LDL<70 after treatment followup if no complication associated; If complication LDL<55';
        }

        // Condition 2: Elevated CVD risk
        if (cvdRisk !== null) {
            var isLab = obs.get('obs-total-cholesterol') > 0;
            if ((isLab && cvdRisk >= 20) || (!isLab && cvdRisk >= 10)) {
                return 'Consider starting statin targeting LDL<70 after treatment followup if no complication associated; If complication LDL<55';
            }
        }

        // Condition 3: Elevated lipids
        if ((tc !== null && tc > 200) || (ldl !== null && ldl >= 130)) {
            return 'Consider starting statin targeting LDL<70 after treatment followup if no complication associated; If complication LDL<55';
        }

        return null;
    },

    'auto-dyslipidemia-status': function(obs) {
        var ldl = obs.get('obs-ldl');
        var dysTx = obs.get('obs-dyslipidemia-copy');
        if (!dysTx) return null;
        if (ldl === null || ldl === undefined) return 'Unknown';
        return ldl < 70 ? 'Controlled' : 'Uncontrolled';
    },

    'pregnancy-row-visibility': function(obs) {
        var age = obs.patient.age;
        var sex = obs.patient.gender;
        return (sex === 'F' && age >= 15 && age <= 45) ? 'visible' : 'hidden';
    }
};
```

> `form-conditions.js` runs client-side in Bahmni's observation model. Use `obs.get('field-id')` to read current values. Return values can populate read-only fields or trigger CDS alerts via `conditions.alert()`.

#### 2.3 Build the Angular Form Template

Convert the HTML `<table>` layout to an AngularJS template. Replace all inline event handlers with Angular directives:

| Prototype Pattern | AngularJS Replacement |
|---|---|
| `<input oninput="allowOnlyInteger(this); calculateCVDRisk()">` | `<input ng-change="vm.calculateCvdRisk()" ng-keyup="vm.allowOnlyInteger($event)">` |
| `<button onclick="toggleBtn(this)">` | `<button ng-click="vm.toggleBtn('groupName', $event)">` |
| `style="display:none"` + JS toggle | `<tr ng-show="vm.isConditionMet()">` |
| `document.getElementById('obs-sbp').value` | `vm.obs.sbp` (via `ng-model`) |
| `Array.from(...).forEach(...)` DOM manipulation | `ng-repeat` + `vm.pills` array |

**Structural patterns to follow:**

```html
<!-- Auto-calculated read-only field (stores to DB) -->
<input type="text" id="obs-bmi" ng-model="vm.obs.bmi" readonly
       class="calculated-field" concept="'BMI UUID'" />

<!-- Conditional row visibility -->
<tr ng-show="vm.obs.sbp || vm.obs.dbp">
    <td>HTN Grade</td>
    <td>
        <input type="text" id="obs-htn-grade" ng-model="vm.obs.htnGrade" readonly
               concept="'HTN Grade UUID'" />
    </td>
</tr>

<!-- Multi-select pill box as ng-repeat -->
<div class="pill-container">
    <div class="pill" ng-repeat="pill in vm.pills.complic track by $index">
        {{pill.label}}
        <span class="remove-pill" ng-click="vm.removePill('complic', $index)">&times;</span>
    </div>
</div>

<!-- CVD Risk badge with statin suggestion -->
<div ng-show="vm.cvdRisk.risk !== null">
    <span class="risk-badge {{vm.cvdRisk.badgeClass}}">{{vm.cvdRisk.label}}</span>
    <div ng-show="vm.statinSuggestion" class="statin-suggestion">
        {{vm.statinSuggestion}}
    </div>
</div>
```

> **Key rule**: Every `obs-*` field that needs to persist to the database must have a `concept="'uuid'"` attribute. Read-only calculated fields also need this — Bahmni will store the calculated value as an observation.

---

### Phase 3: Custom Display Control Implementation (Full Fidelity)

For pixel-perfect parity with the prototype, implement a **Custom Display Control**. This gives complete control over the HTML/CSS/JS rendered inside a Bahmni form section.

#### 3.1 Create the Manifest

```json
// dmHtnFollowup.json — deployed to customDisplayControl/
{
    "name": "DM/HTN Follow-up Form",
    "displayControl": "dmHtnFollowup",
    "template": "/customDisplayControl/templates/dmHtnFollowupTemplate.html",
    "controller": "/customDisplayControl/js/dmHtnFollowupController.js",
    "styles": [
        "/customDisplayControl/css/dmHtnFollowup.css"
    ],
    "requiredPrivilege": "app:clinical",
    "conceptSet": "DM/HTN Follow-up Set UUID"
}
```

#### 3.2 Write the Controller

```javascript
// dmHtnFollowupController.js
angular.module('bahmni.clinical')
    .controller('dmHtnFollowupController', ['$scope', 'dmHtnCalculationService',
        function($scope, calculationService) {

            var vm = this;
            vm.obs = $scope.observations || {};
            vm.pills = { complic: [], comorb: [], risks: [] };

            // Recalculate whenever observations change
            $scope.$watch('observations', function(newObs) {
                if (!newObs) return;
                vm.cvdRisk = calculationService.computeCvdRisk(newObs);
                vm.statinSuggestion = computeStatinSuggestion(newObs);
                vm.htnGrade = calculationService.computeHtnGrade(
                    newObs.patient.age,
                    newObs.get('obs-sbp'),
                    newObs.get('obs-dbp')
                );
                // ... bind all auto-calculations ...
            }, true);

            vm.toggleBtn = function(groupName, $event) {
                // Radio/checkbox toggle logic matching prototype
            };

            vm.addPill = function(type, value) {
                vm.pills[type].push({ label: value });
                // Also persist as child obs in the obs group
            };

            vm.removePill = function(type, index) {
                vm.pills[type].splice(index, 1);
                // Remove child obs from obs group
            };
        }
    ]);
```

#### 3.3 Register in Bahmni Configuration

Add to `bahmni_config/openmrs/apps/clinical/extension.json`:

```json
{
    "id": "dmHtnFollowupForm",
    "extensionPointId": "org.bahmni.patient.search",
    "type": "form",
    "extensionParams": {
        "formType": "DM/HTN Follow-up",
        "formUrl": "/customDisplayControl/dmHtnFollowup.json"
    }
}
```

---

### Phase 4: Rapid Deployment via Standard HTML Form Entry (Lower Fidelity)

If a Custom Display Control is not immediately feasible, deploy using **Velocity Template + `form-conditions.js`**. This preserves all auto-calculations but replaces the custom pill and badge UI with standard HTML inputs.

#### 4.1 Velocity Template Structure

```velocity
## /var/www/bahmni_config/openmrs/apps/clinical/forms/dmHtnFollowup.vm
<script src="/openmrs/scripts/form-conditions.js"></script>
<script src="/customDisplayControl/js/dmHtnCalculationService.js"></script>

<table>
    <tr>
        <td><label>Type of DM</label></td>
        <td>
            <select id="search-dm" concept="Type of DM UUID"
                    onchange="calculateCvdRisk()">
                <option value="">— Select —</option>
                <option value="Type 1 Diabetes Mellitus">Type 1</option>
                <option value="Type 2 Diabetes Mellitus">Type 2</option>
            </select>
        </td>
    </tr>
    <tr>
        <td><label>Systolic BP</label></td>
        <td>
            <input type="text" id="obs-sbp" concept="Systolic BP UUID"
                   oninput="allowOnlyInteger(this); calculateCvdRisk(); calculateHtnGrade();" />
        </td>
    </tr>
    <!-- Hidden calculated fields -->
    <tr style="display:none">
        <td><input type="hidden" id="obs-bmi" concept="BMI UUID" /></td>
    </tr>
    <tr style="display:none">
        <td><input type="hidden" id="obs-cvd-risk" concept="CVD Risk UUID" /></td>
    </tr>
</table>
```

#### 4.2 What Works and What Doesn't

| Feature | Standard Form Entry | Workaround |
|---|---|---|
| Auto-calculations (BMI, GFR, CVD risk) | Yes — via `form-conditions.js` | Store as hidden obs |
| Statin suggestion text | Partial | Render as read-only text field |
| Multi-select pills | No | Use `<select multiple>` or checkbox group |
| Color-coded risk badges | No | Use text labels ("High Risk") |
| Pregnancy row logic | Yes — via Velocity `#if` | `#if($patient.age >= 15 && $patient.age <= 45 && $patient.gender == 'F')` |
| Ethiopian date picker | No | Use Bahmni's built-in date picker |
| MMAS-4 adherence modal | Partial | Render as inline question set |
| Glassmorphism / custom CSS | No | Limited by Bahmni's iframe sandbox |

---

### Phase 5: Module Integration Points

#### 5.1 Appointment Scheduling Integration

Wire `obs-appointment-gregorian` to the Bahmni Appointment Scheduling API, including **slot quota** validation against the configured `maxAppointmentsPerDate` on the Appointment Service concept:

```javascript
// In your controller's save handler
function onFormSave() {
    var appointmentDate = $scope.observations.get('obs-appointment-gregorian');
    if (!appointmentDate) return;

    // --- Slot quota check ---
    var serviceUuid = 'DM/HTN Follow-up Service UUID';
    Bahmni.Appointments.getAllAppointments({
        forDate: appointmentDate,
        serviceUuid: serviceUuid
    }).then(function(existingAppointments) {
        var maxQuota = appointmentService.maxAppointmentsPerDate || 10;  // fallback
        var bookedCount = existingAppointments.results.length;
        if (bookedCount >= maxQuota) {
            $scope.showQuotaWarning = true;
            $scope.quotaMessage = 'Service fully booked (' + bookedCount +
                '/' + maxQuota + '). Please select another date.';
            return;
        }
        $scope.showQuotaWarning = false;

        // --- Proceed with booking ---
        Bahmni.Appointments.create({
            patientUuid: $scope.patient.uuid,
            providerUuid: $scope.session.currentProvider.uuid,
            serviceUuid: serviceUuid,
            startDateTime: appointmentDate,
            endDateTime: moment(appointmentDate).add(30, 'minutes').toISOString(),
            appointmentKind: 'FollowUp',
            notes: 'DM/HTN scheduled follow-up'
        }).then(function(response) {
            // Update obs-appointment with the appointment UUID for cross-reference
        });
    });
}
```

The prototype's hidden `obs-appointment-gregorian` field serves as the canonical date source — no additional user input required beyond the Ethiopian calendar picker.

#### 5.2 MMAS-4 Adherence as Structured Observations

Register four concepts for the MMAS-4 questions so responses are queryable:

| Field ID | Concept Name | Answer Set |
|---|---|---|
| `mmas4-q1` | Adherence: Forgetfulness | Yes / No |
| `mmas4-q2` | Adherence: Carelessness | Yes / No |
| `mmas4-q3` | Adherence: Feeling Better | Yes / No |
| `mmas4-q4` | Adherence: Feeling Worse | Yes / No |

The calculated composite score ("Good" / "Poor") is stored in `obs-overall-adherence`.

#### 5.3 Multi-Select Data via Obs Groups

Each pill selection should be stored as a child observation within an **Obs Group**:

```javascript
function addPillToObsGroup(groupConceptUuid, selectedConceptUuid) {
    var obsGroup = {
        concept: { uuid: groupConceptUuid },
        groupMembers: [{
            concept: { uuid: selectedConceptUuid },
            value: true
        }]
    };
    $scope.observations.push(obsGroup);
}
```

This ensures each complication, comorbidity, or risk factor is stored as structured, queryable data rather than a concatenated text string.

#### 5.4 CVD Risk Data for Analytics

Store both the numeric risk (`obs-cvd-risk`) and the categorical label (`obs-cvd-risk-badge`) to enable:

- Population-level CVD risk distribution reports
- Risk-stratified cohort queries
- Longitudinal risk tracking across encounters
- Automated alert rules based on risk thresholds

---

### Phase 6: Testing and Validation Protocol

Validate each feature by running identical inputs against both the prototype (open in browser) and the Bahmni-deployed form. Outputs must match exactly.

| Test | Input | Expected Output |
|---|---|---|
| CVD Risk (Lab) | TC=240, SBP=145, age=55, M, non-smoker, DM=Type 2 | Risk = 18.3% (Orange: "10% to <20%") |
| CVD Risk (Non-lab) | Clear TC, weight=75, height=170, same context | Risk from BMI-based lookup (Orange) |
| Statin — DM + age | DM=Type 2, age≥40, no dyslipidemia tx | Suggestion visible |
| Statin — Dyslipidemia tx entered | Add text to obs-dyslipidemia | Suggestion hidden |
| Statin — High TC | TC > 200, no DM, no tx | Suggestion visible |
| Statin — High LDL | LDL ≥ 130, no DM, no tx | Suggestion visible |
| Pregnancy row | age=30, sex=F | Row visible |
| Pregnancy hidden | age=50, sex=F | Row hidden |
| Pediatric BP | age=8, sex=M, height=130, SBP=115, DBP=75 | AAP classification badge |
| GFR | Creatinine=1.2, age=45, sex=F | eGFR + KDIGO grade |
| MMAS-4 | Yes + 4 answers | Adherence = "Poor" |
| Copy Last Observation | Save → modify → Copy | Fields restored (skip list respected) |
| Appointment | Select Ethiopian date | Gregorian hidden value synced |
| Appointment — quota | Book to capacity on a given date | Warning shown: "Service fully booked"; booking blocked |
| Appointment — quota | Select date with available slots | Date accepted; no quota warning |
| Reset | Click Add New | All fields cleared |

---

### Phase 7: Deployment Checklist

- [ ] All concepts registered in OpenMRS with correct datatypes and answer sets
- [ ] Concept UUIDs recorded and referenced in form configuration files
- [ ] `dmHtnCalculationService.js` deployed and registered in Angular module
- [ ] `form-conditions.js` deployed with statin + dyslipidemia rules
- [ ] Custom Display Control manifest (`.json`) and controller deployed
- [ ] Appointment scheduling API integration tested end-to-end
- [ ] Slot quota enforcement verified — `maxAppointmentsPerDate` read correctly from Appointment Service concept; quota-exceeded warning displays and blocks booking
- [ ] MMAS-4 hidden observation concepts created
- [ ] Obs group concepts created for multi-select fields
- [ ] All 14 test cases from Phase 6 pass against prototype
- [ ] Cross-browser verified: Chrome, Firefox, Safari, Edge
- [ ] Mobile-responsive layout confirmed on tablet and phone viewports
- [ ] Calculated values persist in OpenMRS database and appear in visit history

---

### Deployment File Map

| Artifact | Deployment Path |
|---|---|
| `dmHtnCalculationService.js` | `/var/www/bahmni_config/openmrs/apps/clinical/customDisplayControl/js/` |
| `form-conditions.js` | `/var/www/bahmni_config/openmrs/apps/clinical/` |
| `dmHtnFollowupTemplate.html` | `/var/www/bahmni_config/openmrs/apps/clinical/customDisplayControl/templates/` |
| `dmHtnFollowupController.js` | `/var/www/bahmni_config/openmrs/apps/clinical/customDisplayControl/js/` |
| `dmHtnFollowup.css` | `/var/www/bahmni_config/openmrs/apps/clinical/customDisplayControl/css/` |
| `dmHtnFollowup.json` | `/var/www/bahmni_config/openmrs/apps/clinical/customDisplayControl/` |
| `dmHtnFollowup.vm` (if using Option B) | `/var/www/bahmni_config/openmrs/apps/clinical/forms/` |
| `extension.json` (amendment) | `/var/www/bahmni_config/openmrs/apps/clinical/` |

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

*Maintained by Dr Teselonke K. Prototype for Bahmni HTML Form Entry conversion.*
