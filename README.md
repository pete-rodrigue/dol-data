# DOL Apprenticeship Data Analysis

This notebook analyzes data from the Department of Labor's
[Apprenticeship Participant Characteristics and Training Outcomes](https://data.dol.gov/datasets/10264)
dataset, which tracks registered apprentices across the United States using the
RAPIDS (Registered Apprenticeship Partners Information Data System) database.

## Data

The dataset contains **8,745,442 rows and 34 columns**, covering apprentices from
fiscal years 2014–2024. Each row represents an individual apprentice record and
can appear in one of three tables:

| Table | Count | Description |
|-------|-------|-------------|
| `V_OA_APPRENTICE_ACTIVE_V1` | 5,601,774 | Currently active apprentices |
| `V_OA_APPRENTICE_NEW_V1` | 2,341,225 | Newly registered apprentices |
| `V_OA_APPRENTICE_EXITER_V1` | 802,443 | Apprentices who have exited |

The data can be obtained two ways:

- **Bulk download:** A ~3GB zip file from the [dataset page](https://data.dol.gov/datasets/10264),
  containing 175 CSV chunks named `Apprenticeship (active, new, exiter)_chunk_1.csv` through `_chunk_175.csv`
- **API:** Query the DOL API at `https://apiprod.dol.gov/v4/get/ETA/apprenticeship_data/json`
  (requires a free API key)

## Repository Contents

| File | Description |
|------|-------------|
| `analysis.ipynb` | Main analysis notebook (see below for details) |
| `index.html` | Webpage to view & filter data |
| `requirements.txt` | Python dependencies |

## Setup

```bash
pip install -r requirements.txt
```

Key dependencies: `polars`, `pandas`, `pyarrow`, `requests`

To run the bulk download analysis, place `ETA_apprenticeship_data.zip` in the
repo root directory. To use the API sections, save your DOL API key in a file
called `api_key.txt` in the repo root.

## Notebook Overview

### Bulk Download Analysis

The notebook loads all 175 CSV chunks from the zip file using Polars for
efficient memory handling, then casts all 34 columns to their correct types:

- **Categorical:** fiscal year, demographics, geography, industry, occupation status
- **Numeric (Float64):** wages, apprentice ID
- **Boolean:** adjustment flags (note: `UNION_Y_N` and all adjust flags are 100% missing in this extract)
- **Date:** start date (`MM DD YYYY HH:MM:SS` format), exit date (`M/DD/YYYY` format)

#### Key findings from the notebook output

**Missingness:**
- `Exit Date`, `UNION_Y_N`, `ACTIVE_ADJUST`, `COMPLETER_ADJUST`, and `NEW_ADJUST` are 100% missing
- `EXIT_WAGE_ADJUST` is 91% missing; `Exit_Wage` is 46% missing; `STARTING_WAGE` is 31% missing
- 16 columns have no missing data, including all demographic fields and `Apprentice Status Code`

**Top occupations** (share of all records):
1. Not Provided — 33.7%
2. Electrician — 11.1%
3. Carpenter — 5.1%
4. Plumber — 3.7%
5. Construction Craft Laborer — 3.5%

**Completion rates** (completed ÷ completed + cancelled, n = 6,122,877):

By starting wage quartile:
| Quartile | Range | Completion Rate |
|----------|-------|----------------|
| Q1 | ≤ $12.50/hr | 45.6% |
| Q2 | $12.50–$15.73/hr | 54.2% |
| Q3 | $15.73–$19.50/hr | 53.0% |
| Q4 | > $19.50/hr | 61.0% |

By fiscal year: ranged from ~54% (2014–2015) to ~57% (2017), dipping to ~53% in 2021–2022.
Note: 2024 shows an inflated rate (89%) as most 2024 apprentices are still active.

**Wage growth** (among completed/cancelled apprentices with valid wages, n = 3,802,904):
- Median starting wage: **$15.78/hr**
- Median exit wage: **$22.10/hr**
- Median wage gain: **$6.20/hr**

Wage gains are largest for younger apprentices (Ages 24 and Under) in earlier years,
and have declined in more recent cohorts, likely reflecting that recent apprentices
have had less time to progress.

### API Analysis

The notebook also includes reference code for pulling data via the DOL API,
including examples of filtering by state, union status, industry, and other fields.
This section is provided for reference — the main analysis uses the bulk download.

## Notes

- `STARTING_WAGE` and `Exit_Wage` contain some extreme outliers (max values in the
  millions), likely data entry errors. The wage adjustment flag columns were intended
  to filter these out, but are fully missing in this extract.
- The `State` field reflects program registration state, which may differ from where
  the apprentice is located (`APPR_STATE`).
- Fiscal year runs October 1 – September 30.
