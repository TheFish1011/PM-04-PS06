# PM04-PS06 — Bursary Application Processor

> Organize projects for efficiency and easy maintenance.

A UiPath RPA automation that reads bursary applicant data from a CSV file, validates and filters eligible candidates based on academic and financial criteria, and writes the results to output files.

> 🌿 **Branch:** `feature/reframework-integration`
> This branch refactors the original flat sequence into a **REFramework-adapted architecture** (Init → Get Transaction Data → Process Transaction → End Process). All paths and thresholds are centralised in a single `dict_Config` dictionary — nothing is hard-coded in downstream workflows.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Workflows](#workflows)
- [Configuration — dict_Config](#configuration--dict_config)
- [Variables](#variables)
- [Data](#data)
- [Dependencies](#dependencies)
- [How to Run](#how-to-run)
- [Output](#output)
- [Changelog](#changelog)

---

## Overview

This automation processes student bursary applications by:

1. **Init** — Loading all settings and thresholds into a central config dictionary.
2. **Get Transaction Data** — Reading applicant records from the CSV input file and validating them.
3. **Process Transaction** — Filtering applicants against the eligibility criteria.
4. **End Process** — Writing eligible applicants to an output CSV and appending a summary to the audit log.

| Property            | Value                          |
|---------------------|-------------------------------|
| Project Name        | PM04-PS06                     |
| Version             | 1.0.0                         |
| Type                | Process (Unattended)          |
| Target Framework    | Windows                       |
| Expression Language | Visual Basic                  |
| UiPath Studio       | 26.0.191.0                    |
| Active Branch       | `feature/reframework-integration` |

---

## Architecture

This branch follows a **REFramework-adapted** pattern:

```
┌─────────────────────────────────────────────────────────┐
│                        Main.xaml                        │
│                                                         │
│  ① INIT              → InitAllSettings.xaml             │
│       └─ Builds dict_Config (paths + thresholds)        │
│                                                         │
│  ② GET TRANSACTION   → ReadCSVData.xaml                 │
│       └─ Loads dt_Applications from CSV                 │
│                                                         │
│  ③ VALIDATE          → ValidateData.xaml                │
│       └─ Sets isValidData flag                          │
│                                                         │
│  ④ PROCESS           → FilterApplicants.xaml            │  ← only if isValidData = True
│       └─ Populates dt_Eligible + intEligibleCount       │
│                                                         │
│  ⑤ END PROCESS       → WriteResults.xaml                │  ← only if isValidData = True
│       └─ Writes output CSV + appends audit log          │
│                                                         │
│  ✗ INVALID PATH      → Log Fatal + Abort               │  ← if isValidData = False
└─────────────────────────────────────────────────────────┘
```

> All activities carry **Source / Reason / Outcome** annotations for full traceability.

---

## Project Structure

```
PM04-PS06/
│
├── Main.xaml                          # Entry point — REFramework orchestrator
│
├── Workflows/
│   ├── InitAllSettings.xaml           # ① INIT — builds dict_Config  ← NEW
│   ├── ReadCSVData.xaml               # ② GET — reads CSV into dt_Applications
│   ├── ValidateData.xaml              # ③ VALIDATE — sets isValidData flag
│   ├── FilterApplicants.xaml          # ④ PROCESS — filters eligible applicants
│   └── WriteResults.xaml              # ⑤ END — writes output CSV + audit log
│
├── Data/
│   ├── Input/
│   │   └── bursary_applications.csv   # Source applicant data
│   └── Output/
│       ├── Eligible_Applicants.csv    # Filtered results output
│       └── ResultsLog.txt             # Processing summary audit log
│
├── project.json                       # UiPath project configuration
└── README.md                          # This file
```

---

## Workflows

### `Main.xaml` _(Entry Point)_
REFramework orchestrator. Invokes all sub-workflows in the correct order and passes data between them via shared variables. The central `dict_Config` dictionary (populated by `InitAllSettings`) is used for every path and threshold lookup — nothing is hard-coded here.

---

### `Workflows/InitAllSettings.xaml` ⭐ _New in this branch_
**REFramework Init state.** Builds the `dict_Config` dictionary from scratch and returns it to `Main`. This is the **only file that needs to be edited** if a path or threshold changes.

| Key | Value | Description |
|---|---|---|
| `CSVPath` | `Data\Input\bursary_applications.csv` | Input file path |
| `OutputPath` | `Data\Output\Eligible_Applicants.csv` | Output file path |
| `LogPath` | `Data\Output\ResultsLog.txt` | Audit log path |
| `MinMark` | `70` | Minimum average mark (%) |
| `MaxIncome` | `150000.0` | Maximum household income (ZAR) |

Logs `"Config loaded. 5 settings ready."` on completion.

---

### `Workflows/ReadCSVData.xaml`
**REFramework Get Transaction Data state.** Reads the input CSV file using the `CSVPath` from `dict_Config` and loads all records into `dt_Applications`.

| Argument | Direction | Type | Description |
|---|---|---|---|
| `in_strCSVPath` | In | String | Path to the input CSV |
| `out_dt_Applications` | Out | DataTable | Loaded applicant records |

---

### `Workflows/ValidateData.xaml`
**REFramework validation step.** Checks `dt_Applications` for required columns and non-empty rows. Sets `isValidData` to `True` or `False`. If `False`, `Main` logs a **Fatal** message and aborts without writing any output.

| Argument | Direction | Type | Description |
|---|---|---|---|
| `in_dt_Applications` | In | DataTable | Loaded applicant records |
| `out_isValidData` | Out | Boolean | Validation result flag |

---

### `Workflows/FilterApplicants.xaml`
**REFramework Process Transaction state.** Iterates through each applicant row and applies both eligibility rules sourced from `dict_Config`:
- ✅ `AverageMark` ≥ `MinMark` (70)
- ✅ `HouseholdIncome` ≤ `MaxIncome` (150 000)

| Argument | Direction | Type | Description |
|---|---|---|---|
| `in_dt_Applications` | In | DataTable | All loaded applicants |
| `in_intMinMark` | In | Int32 | Minimum mark threshold |
| `in_dblMaxIncome` | In | Double | Maximum income threshold |
| `out_dt_Eligible` | Out | DataTable | Applicants who passed both rules |
| `out_intEligibleCount` | Out | Int32 | Count of eligible applicants |

---

### `Workflows/WriteResults.xaml`
**REFramework End Process state.** Writes `dt_Eligible` to the output CSV and appends a timestamped summary line to the audit log.

| Argument | Direction | Type | Description |
|---|---|---|---|
| `in_dt_Eligible` | In | DataTable | Eligible applicant records |
| `in_strOutputPath` | In | String | Output CSV path |
| `in_strLogPath` | In | String | Audit log path |
| `in_intEligibleCount` | In | Int32 | Count of eligible applicants |
| `in_intTotalCount` | In | Int32 | Total applicants processed |

---

## Configuration — `dict_Config`

All settings are managed in `Workflows/InitAllSettings.xaml`. **No other workflow contains hard-coded values.**

| Key | Type | Default Value | Description |
|---|---|---|---|
| `CSVPath` | String | `Data\Input\bursary_applications.csv` | Input CSV file location |
| `OutputPath` | String | `Data\Output\Eligible_Applicants.csv` | Eligible applicants output |
| `LogPath` | String | `Data\Output\ResultsLog.txt` | Audit log file location |
| `MinMark` | Int32 | `70` | Minimum average mark for eligibility |
| `MaxIncome` | Double | `150000.0` | Maximum household income for eligibility |

> 💡 To change a threshold or file path, edit **only** `InitAllSettings.xaml`.

---

## Variables

Defined in `Main.xaml`:

| Variable | Type | Description |
|---|---|---|
| `dict_Config` | `Dictionary(String, Object)` | Central config — all paths and thresholds |
| `dt_Applications` | DataTable | All applicant records loaded from CSV |
| `dt_Eligible` | DataTable | Filtered eligible applicant records |
| `isValidData` | Boolean | Set by ValidateData — gates filter + output steps |
| `intEligibleCount` | Int32 | Count of eligible applicants found |

> ℹ️ `strCSVPath`, `strOutputPath`, `strLogPath`, `intMinMark`, and `dblMaxIncome` have been **removed** from `Main.xaml` and consolidated into `dict_Config`.

---

## Data

### Input — `Data/Input/bursary_applications.csv`

| Column | Type | Description |
|---|---|---|
| `StudentID` | Integer | Unique student identifier |
| `Name` | String | Full name of the applicant |
| `Age` | Integer | Age of the applicant |
| `Province` | String | Province of residence |
| `AverageMark` | Integer | Academic average mark (%) |
| `HouseholdIncome` | Double | Annual household income (ZAR) |

**Example rows:**
```csv
StudentID,Name,Age,Province,AverageMark,HouseholdIncome
1001,Lerato Mokoena,19,Gauteng,78,120000
1002,Thabo Dlamini,21,KZN,62,90000
1003,Ayanda Khumalo,20,Eastern Cape,85,50000
```

### Output — `Data/Output/Eligible_Applicants.csv`
Contains only the applicant rows that passed both eligibility rules, in the same column format as the input.

### Output — `Data/Output/ResultsLog.txt`
Timestamped audit log appended on each run:
```
2026-04-21 21:02:26 | Processed: 5 | Eligible: 3
```

---

## Dependencies

| Package | Version |
|---|---|
| `UiPath.Excel.Activities` | 3.5.0-preview |
| `UiPath.System.Activities` | 26.2.4 |

---

## How to Run

1. **Open the project** in UiPath Studio (v26.0+, Windows target framework).
2. **Verify input data** — ensure `Data/Input/bursary_applications.csv` exists and is correctly formatted.
3. **Adjust settings** (optional) — edit `Workflows/InitAllSettings.xaml` to change any file paths or eligibility thresholds. **Do not edit** downstream workflows for config changes.
4. **Run** `Main.xaml` via the Studio **Run** button or deploy to UiPath Orchestrator as an unattended process.

> ⚠️ The automation is configured as **unattended** (`isAttended: false`). No human interaction is required during execution.

---

## Output

After a successful run:

- ✅ `Data/Output/Eligible_Applicants.csv` — updated with eligible applicants.
- ✅ `Data/Output/ResultsLog.txt` — appended with a new timestamped summary line.
- ✅ Execution log — contains `"PS06 bursary processing completed successfully."` at Info level.

If validation fails:
- ❌ Output files are **not written**.
- ❌ Execution log contains `"Validation failed. Aborting."` at **Fatal** level.

---

## Changelog

| Branch | Commit | Description |
|---|---|---|
| `feature/reframework-integration` | Latest | Refactor to REFramework-adapted architecture with Source/Reason/Outcome annotations |
| `main` | Previous | Add output CSV writer and audit log |
| `main` | Previous | Implement bursary filter logic with dynamic thresholds |
| `main` | Previous | Implemented ReadCSVData, ValidateData, and wired Main sequence |
| `main` | Initial | Initial commit |

---

*Generated for PM04-PS06 · Branch: `feature/reframework-integration` · UiPath Studio 26.0.191.0 · Windows Framework*
