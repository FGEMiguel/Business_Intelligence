# Who reaches Chile's public hospitals without FONASA coverage?

Business Intelligence (IIB423T-1), Universidad del Desarrollo · Group 03 · Miguel Figueroa, Sebastián Bunzli

Severity and admission profile of Isapre/Particular discharges versus FONASA discharges in FONASA's GRD files, 2020–2024. This README covers the whole project and has a **Workshop 2 (W2)** section. W1 and C1 artifacts are preserved in their folders and in the Git history.

## Project summary
- **Problem:** among patients with Isapre or self-pay (Particular) coverage who are hospitalized in Chile's public network, is the severity and admission profile different from FONASA beneficiaries, and is the difference stable?
- **Unit of observation:** one hospital discharge (*egreso*). It is not a patient: the same person can appear several times.
- **Main KPI:** % of valid discharges with IR-GRD severity level 3, by insurance group and year. Supporting: % urgent admissions (`TIPO_INGRESO = URGENCIA`) and mean relative weight (`IR_29301_PESO`, a case-mix weight, not a cost).
- **Intended decision:** whether a Servicio de Salud or hospital management analyst should monitor Isapre/Particular discharges as a distinct group.
- The notebook labels the group `PRIVATE`. Particular means self-pay, so the group is not made only of insured patients.

## Repository layout
```
notebooks/W2_project.ipynb   W2 notebook (this submission; developed as W2_GRD.ipynb)
report/GROUP_03_W2_Report.docx   W2 report
analysis/, report/, presentation/   W1 and C1 artifacts (unchanged)
data/                        raw-derived data, not committed (see .gitignore)
data/processed/              extracts and outputs written by the notebook
README.md                    this file
```

## Data provenance
| Source | Content | Notes |
|---|---|---|
| FONASA open GRD discharge files, via the course repository `BI_T-main` (`GRD/`) | `GRD_PUBLICO_2020.txt`, `GRD_PUBLICO_2021.txt`, `GRD_PUBLICO_EXTERNO_2022.txt`, `GRD_PUBLICO_2023.txt`, `GRD_PUBLICO_2024.txt` | pipe-delimited, about 129 columns, one row per discharge; layout label of the course extract `SA_21_V1.2`; encodings: latin1 (2020, 2021, 2024) and UTF-16 (2022, 2023) |
| DEIS/MINSAL Base de Establecimientos, via `BI_T-main` (`Establecimientos/<year>/Base_Establecimientos_<year>.xlsx`) | establishment name, level of complexity, region, commune, emergency type | joined by establishment code; one file per year |

- **Coverage:** the GRD files describe public hospitals that register discharges in FONASA's GRD tool (FONASA reports 86 establishments with GRD registry and 68 financed through Programa 05 in 2023; source: FONASA, *Cuenta Pública 2023*). Private-provider GRD data are not in these files.
- **Columns read:** `PREVISION`, `IR_29301_PESO`, `IR_29301_SEVERIDAD`, `IR_29301_MORTALIDAD`, `TIPO_INGRESO`, plus `IR_29301_COD_GRD`, `SEXO`, `COD_HOSPITAL`, birth and admission/discharge dates, and `ID_BENEFICIARIO` (stored only as a deterministic hash, `patient_hash`).
- **Confidentiality:** raw files and `data/processed/*.csv.gz` are **not published**. The processed files contain a hashed beneficiary identifier and record-level data, so they stay out of the repository. The repository holds only code, aggregate tables and figures.

## How to run
1. **Environment:** Python 3.10 or later with `pandas`, `numpy`, `matplotlib`, `scikit-learn` and `openpyxl` (the last one reads the Establecimientos `.xlsx` files), plus `jupyter`.
2. **Raw data:** place the course folder so that it contains `GRD/` and `Establecimientos/`, and point the notebook to it:
   - Windows PowerShell: `$env:RAW_DATA_ROOT = "C:\path\to\BI_T-main"`
   - macOS/Linux: `export RAW_DATA_ROOT=/path/to/BI_T-main`
3. **Run:** from the repository root, `jupyter nbconvert --to notebook --execute --inplace notebooks/W2_project.ipynb`, or *Restart & Run All* in Jupyter.
4. **First run vs. later runs:** if `data/processed/grd_<year>_extended.csv.gz` is missing, incomplete or has an empty `patient_hash`, Section 1 rebuilds the five extracts from the raw files (this takes the longest). Otherwise they are reused. Set `FORCE_REGENERATE = True` in the first cell to rebuild anyway. The project root can be overridden with `W2_ROOT`.
5. **Reproducibility:** fixed seed `SEED = 42` for the 25% stratified sample; no other randomness.

Memory: the five processed extracts hold about 4.66 million rows; 16 GB of RAM is comfortable.

## What the W2 notebook produces
| Notebook section | Content | Output files (`data/processed/`) |
|---|---|---|
| §1 Regeneration | processed extracts with hospital code, hashed id, age, sex and GRD code; Establecimientos join | `grd_<year>_extended.csv.gz`, `regeneration_report.csv` |
| §2 Feedback → action | 10-row table (W1, C1, self-audit) | |
| §3 Data dictionary | variables, types, acceptable values, missing meaning | |
| §4 Data validation | ten executable checks with row counts before and after | `validation_checks.csv` |
| §5 EDA | descriptive table and three figures | `descriptive_by_year_insurance.csv`, `fig1_peso_distribution.png`, `fig2_severity_trend.png`, `fig3_urgency_trend.png` |
| §6 Populations and sampling | target, observed and analytical populations; 25% stratified sample | `kpi_full.csv`, `kpi_sample25.csv` |
| §7 Leakage audit | ten context-specific risks | |
| §8 Split and validation | chronological split (train 2020–2023, test 2024); GroupKFold(5) inside training; train-only preprocessing demo | `split_summary.csv`, `groupkfold_folds.csv` |
| §9 Fiscal-cost groundwork | formula only; no estimate (the official MAI base price was not located) | |
| §10 Management summary | 120–180 words | |

The written W2 report (`GROUP_03_W2_Report.docx`) summarizes these results with the numbers from the executed notebook.

## Key results (2022–2024)
- Isapre/Particular discharges have a 4.9–5.5 point lower share of high-severity cases than FONASA and a 5.5–7.0 point higher share of urgent admissions. In 2020 there is no severity gap; in 2021 it reverses.
- Isapre/Particular discharges are 7.7–10.5 years younger, so the severity gap is not yet adjusted for age or diagnosis mix.

## Known limitations
- The establishment join matches 82.5% of rows.
- In the training rows, the number of patient groups equals the number of rows, so the patient grouping in GroupKFold is inactive until the identifier is diagnosed.
- The 2024 holdout was already explored in W1 and C1; it is a robustness check, not a pristine test.
- The comparison is not adjusted for age, diagnosis or hospital.

## Submission (W2)
```
git add notebooks/W2_project.ipynb report/GROUP_03_W2_Report.docx README.md data/processed/*.csv data/processed/*.png
git commit -m "W2: validation, sampling, leakage audit and chronological holdout"
git tag w2-submission
git push origin main --tags
```
Submit the repository link and the `w2-submission` tag (or commit hash). Do not add `data/processed/*.csv.gz`.

## AI-use disclosure
Claude (Anthropic) was used to troubleshoot Python, PowerShell and encoding errors, to draft and debug code, to structure the notebook and to draft the W2 report and this README from the executed notebook's outputs. All reported values were produced by code the team executed against the real files. The team is responsible for the content and for any remaining errors.
