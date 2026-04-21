# PM04-PS06 — Bursary Application Processor

> Organize projects for efficiency and easy maintenance.

A UiPath RPA automation that reads bursary applicant data from a CSV file, validates and filters eligible candidates based on academic and financial criteria, and writes the results to output files.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Workflows](#workflows)
- [Data](#data)
- [Variables](#variables)
- [Dependencies](#dependencies)
- [How to Run](#how-to-run)
- [Output](#output)

---

## Overview

This automation processes student bursary applications by:

1. Reading applicant records from a CSV input file.
2. Validating the data for completeness and correctness.
3. Filtering applicants who meet the eligibility criteria (minimum average mark and maximum household income).
4. Writing the eligible applicants to an output CSV file and logging summary statistics to a text log file.

| Property          | Value                          |
|-------------------|-------------------------------|
| Project Name      | PM04-PS06                     |
| Version           | 1.0.0                         |
| Type              | Process (Unattended)          |
| Target Framework  | Windows                       |
| Expression Language | Visual Basic                |
| UiPath Studio     | 26.0.191.0                    |

---

## Project Structure

```
PM04-PS06/
│
├── Main.xaml                        # Entry point — orchestrates all sub-workflows
│
├── Workflows/
│   ├── ReadCSVData.xaml             # Reads bursary applications from CSV
│   ├── ValidateData.xaml            # Validates the loaded DataTable
│   ├── FilterApplicants.xaml        # Filters eligible applicants
│   └── WriteResults.xaml            # Writes output CSV and results log
│
├── Data/
│   ├── Input/
│   │   └── bursary_applications.csv # Source applicant data
│   └── Output/
│       ├── Eligible_Applicants.csv  # Filtered results output
│       └── ResultsLog.txt           # Processing summary log
│
├── project.json                     # UiPath project configuration
└── README.md                        # This file
```

---

## Workflows

### `Main.xaml` _(Entry Point)_
Orchestrates the entire automation. Initialises configuration variables (file paths, eligibility thresholds), then invokes the four sub-workflows in sequence. Passes data between workflows via shared variables.

### `Workflows/ReadCSVData.xaml`
Reads the input CSV file located at `Data/Input/bursary_applications.csv` and loads it into a `DataTable` (`dt_Applications`) for downstream processing.

### `Workflows/ValidateData.xaml`
Validates the loaded `DataTable` to ensure all required columns are present and that records contain expected data types and non-empty values. Sets the `isValidData` flag accordingly.

### `Workflows/FilterApplicants.xaml`
Iterates through each applicant row and applies the eligibility criteria:
- **Minimum Average Mark** — controlled by `intMinMark`
- **Maximum Household Income** — controlled by `dblMaxIncome`

Eligible records are collected into `dt_Eligible` and the count is tracked via `intEligibleCount`.

### `Workflows/WriteResults.xaml`
Writes the eligible applicants from `dt_Eligible` to `Data/Output/Eligible_Applicants.csv`. Appends a timestamped summary entry to `Data/Output/ResultsLog.txt` in the format:

```
YYYY-MM-DD HH:mm:ss | Processed: X | Eligible: Y
```

---

## Data

### Input — `Data/Input/bursary_applications.csv`

| Column           | Type    | Description                          |
|------------------|---------|--------------------------------------|
| `StudentID`      | Integer | Unique student identifier            |
| `Name`           | String  | Full name of the applicant           |
| `Age`            | Integer | Age of the applicant                 |
| `Province`       | String  | Province of residence                |
| `AverageMark`    | Integer | Academic average mark (percentage)   |
| `HouseholdIncome`| Double  | Annual household income (ZAR)        |

**Example rows:**

```csv
StudentID,Name,Age,Province,AverageMark,HouseholdIncome
1001,Lerato Mokoena,19,Gauteng,78,120000
1002,Thabo Dlamini,21,KZN,62,90000
1003,Ayanda Khumalo,20,Eastern Cape,85,50000
```

### Output — `Data/Output/Eligible_Applicants.csv`
Contains only the applicant rows that passed the eligibility filter, in the same column format as the input file.

### Output — `Data/Output/ResultsLog.txt`
A running log of each execution run, e.g.:
```
2026-04-21 21:02:26 | Processed: 5 | Eligible: 3
```

---

## Variables

The following variables are defined in `Main.xaml`:

| Variable Name      | Type      | Description                                      |
|--------------------|-----------|--------------------------------------------------|
| `strCSVPath`       | String    | File path to the input CSV file                  |
| `strOutputPath`    | String    | File path for the eligible applicants output CSV |
| `strLogPath`       | String    | File path for the results log text file          |
| `intMinMark`       | Int32     | Minimum average mark threshold for eligibility   |
| `dblMaxIncome`     | Double    | Maximum household income threshold               |
| `dt_Applications`  | DataTable | All applicant records loaded from CSV            |
| `dt_Eligible`      | DataTable | Filtered eligible applicant records              |
| `isValidData`      | Boolean   | Flag set after data validation                   |
| `intEligibleCount` | Int32     | Count of eligible applicants found               |

---

## Dependencies

| Package                       | Version         |
|-------------------------------|-----------------|
| `UiPath.Excel.Activities`     | 3.5.0-preview   |
| `UiPath.System.Activities`    | 26.2.4          |

---

## How to Run

1. **Open the project** in UiPath Studio (v26.0+, Windows target framework).
2. **Verify input data** — ensure `Data/Input/bursary_applications.csv` exists and is correctly formatted.
3. **Adjust eligibility thresholds** (optional) — update `intMinMark` and `dblMaxIncome` in `Main.xaml` as needed.
4. **Run** `Main.xaml` via the Studio **Run** button or deploy to UiPath Orchestrator as an unattended process.

> ⚠️ The automation is configured as **unattended** (`isAttended: false`). No human interaction is required during execution.

---

## Output

After a successful run:

- ✅ `Data/Output/Eligible_Applicants.csv` — updated with eligible applicants.
- ✅ `Data/Output/ResultsLog.txt` — appended with a new summary line.

---

*Generated for PM04-PS06 · UiPath Studio 26.0.191.0 · Windows Framework*
