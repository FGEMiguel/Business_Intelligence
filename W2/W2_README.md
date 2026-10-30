# Workshop 2: from exploratory analysis to credible validation

Business Intelligence (IIB423T-1), Universidad del Desarrollo · Group 03 · Miguel Figueroa, Sebastián Bunzli · Due 29 October 2026

This folder holds the Workshop 2 (W2) submission. It continues the project of `../W1/` and `../C1/` with the same problem, KPI and unit of observation. Those folders are unchanged.

## Project in brief
- **Problem:** among patients with Isapre or self-pay (Particular) coverage who are hospitalized in Chile's public network, is the severity and admission profile different from FONASA beneficiaries, and is the difference stable over 2020–2024?
- **Unit of observation:** one hospital discharge (*egreso*). It is not a patient: the same person can appear several times.
- **Main KPI:** % of valid discharges with IR-GRD severity level 3, by insurance group and year. Supporting: % urgent admissions (`TIPO_INGRESO = URGENCIA`) and mean relative weight (`IR_29301_PESO`, a case-mix weight, not a cost).
- **Intended decision:** whether a Servicio de Salud or hospital management analyst should monitor Isapre/Particular discharges as a distinct group.
- The notebook labels the group `PRIVATE`. Particular means self-pay, so the group is not made only of insured patients.

## Folder contents
```
W2/
  README.md                        this file
  notebooks/W2_project.ipynb       W2 notebook (the graded deliverable)
  report/GROUP_03_W2_Report.docx   written report (supporting document)
  data/processed/                  aggregate tables and figures written by the notebook
```

`data/processed/` also holds `grd_<year>_extended.csv.gz` when the notebook runs. Those files are **not committed**: they contain record-level data and a hashed beneficiary identifier (`*.csv.gz` is in `.gitignore`).

## Data provenance
| Source | Files | Notes |
|---|---|---|
| FONASA open GRD discharge files, via the course repository `BI_T-main` (folder `GRD/`) | `GRD_PUBLICO_2020.txt`, `GRD_PUBLICO_2021.txt`, `GRD_PUBLICO_EXTERNO_2022.txt`, `GRD_PUBLICO_2023.txt`, `GRD_PUBLICO_2024.txt` | pipe-delimited, about 129 columns, one row per discharge; course layout label `SA_21_V1.2`; encodings latin1 (2020, 2021, 2024) and UTF-16 (2022, 2023) |
| DEIS/MINSAL Base de Establecimientos, via `BI_T-main` (folder `Establecimientos/<year>/`) | `Base_Establecimientos_<year>.xlsx` | establishment name, complexity level, region, commune, emergency type; joined by establishment code |

- **Coverage:** the GRD files describe public hospitals that register discharges in FONASA's GRD tool (FONASA reports 86 establishments with GRD registry and 68 financed through Programa 05 in 2023; FONASA, *Cuenta Pública 2023*). Private-provider GRD data are not in these files.
- **Columns read:** `PREVISION`, `IR_29301_PESO`, `IR_29301_SEVERIDAD`, `IR_29301_MORTALIDAD`, `TIPO_INGRESO`, `IR_29301_COD_GRD`, `SEXO`, `COD_HOSPITAL`, birth, admission and discharge dates, and `ID_BENEFICIARIO`, which is stored only as a deterministic hash (`patient_hash`).
- **Access:** the raw files are not redistributed here. Obtain them from the course repository `BI_T-main` or from FONASA's open data portal (datosabiertos.fonasa.cl).

## How to run
1. **Environment:** Python 3.10 or later with `pandas`, `numpy`, `matplotlib`, `scikit-learn`, `openpyxl` (reads the `.xlsx` files) and `jupyter`.
   ```
   pip install pandas numpy matplotlib scikit-learn openpyxl jupyter
   ```
2. **Raw data:** point the notebook to the folder that contains `GRD/` and `Establecimientos/`.
   - Windows PowerShell: `$env:RAW_DATA_ROOT = "C:\path\to\BI_T-main"`
   - macOS/Linux: `export RAW_DATA_ROOT=/path/to/BI_T-main`
3. **Run** from inside `W2/`, so the notebook writes to `W2/data/processed/`:
   ```
   cd W2
   jupyter nbconvert --to notebook --execute --inplace notebooks/W2_project.ipynb
   ```
   or open the notebook and use *Kernel → Restart & Run All*.
4. **First run vs. later runs:** if the processed extracts are missing, incomplete or have an empty `patient_hash`, Section 1 rebuilds them from the raw files (the slowest step). Otherwise they are reused. Set `FORCE_REGENERATE = True` in the first cell to rebuild anyway, or set `W2_ROOT` to choose another output root.
5. **Reproducibility:** fixed seed `SEED = 42` for the 25% stratified sample, the only random step. About 16 GB of RAM is comfortable for the 4.66 million rows.

## Where each rubric criterion is answered
| Rubric | Notebook section | Output files (`data/processed/`) |
|---|---|---|
| A. Continuity and feedback | header, §2 (10-row feedback-to-action table) | |
| B. Revised EDA | §5: descriptive table and three figures | `descriptive_by_year_insurance.csv`, `fig1_peso_distribution.png`, `fig2_severity_trend.png`, `fig3_urgency_trend.png` |
| C. Data validation | §3 data dictionary; §4 ten checks with row counts | `validation_checks.csv` |
| D. Sampling | §6: populations, biases, 25% stratified sample | `kpi_full.csv`, `kpi_sample25.csv` |
| E. Leakage audit | §7: ten context-specific risks; §8 decision-time table | |
| F. Holdout and validation | §8: chronological split, GroupKFold(5) inside training, train-only preprocessing | `split_summary.csv`, `groupkfold_folds.csv` |
| G. Communication | §10 management summary; AI-use disclosure | |
| (extension) | §9 fiscal-cost groundwork, formula only | |

## Key results
- **Dataset:** 4,657,061 raw rows, 3,889,686 valid discharges (83.5%). No rows are deleted; validity is a flag.
- **Severity:** in 2022–2024 Isapre/Particular discharges have a 4.9–5.5 point lower share of high-severity cases than FONASA. There is no gap in 2020 and a reversal in 2021.
- **Admission route:** the Isapre/Particular urgent-admission share is higher in all five years, by 5.5 to 8.9 points.
- **Age:** Isapre/Particular discharges are 7.7–10.5 years younger, so the severity gap is not yet adjusted for age or diagnosis mix.
- **Split:** training is 2020–2023 (3,015,975 rows, 77.5%); the holdout is 2024 (873,711 rows, 22.5%), with no episode overlap.

## Known limitations
- The establishment join matches 82.5% of rows.
- In the training rows the number of patient groups equals the number of rows, so the patient grouping in GroupKFold has no effect until the identifier is diagnosed. The folds also mix years rather than following time order.
- The 2024 holdout was already explored in W1 and C1. It is an independent robustness check, not a pristine test or a measure of predictive accuracy.
- No model is trained and no cost is estimated: the official MAI base price was not found in an open source.

## AI-use disclosure
Claude (Anthropic) was used to troubleshoot Python, PowerShell and encoding errors, to draft and debug code, to structure the notebook, and to draft the report and this README from the executed notebook's outputs. All reported values come from code the team executed against the real files. The team is responsible for the content and for any remaining errors.
