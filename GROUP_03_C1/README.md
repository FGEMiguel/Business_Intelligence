# GROUP_03_C1

**Business Intelligence · IIB423T-1 · Universidad del Desarrollo**

- **Members:** Sebastián Bunzli, Miguel Figueroa
- **Group ID:** 03
- **Submission date / version:** 29 September 2026 · v1.0
- **Line developed:** Candidate B — severity and insurance type in GRD hospital discharges (public network), refined from the W1 candidates.

## Folder structure and file roles

```
GROUP_03_C1/
  README.md                          this file
  report/
    GROUP_03_C1_Report.pdf           full report (self-contained)
  presentation/
    GROUP_03_C1_Slides.pdf           defense slides
  analysis/
    01_grd_severity_analysis.ipynb   main notebook (run this)
  data/
    input/
      grd_2021_reduced.csv           raw extract, columns used only, before cleaning
      grd_2022_reduced.csv           (source: GRD_PUBLICO_EXTERNO_2022.txt)
      grd_2023_reduced.csv
      grd_2024_reduced.csv
    output/
      grd_2021_cleaned.csv           cleaned per-year table (post grouping/filtering)
      grd_2022_cleaned.csv
      grd_2023_cleaned.csv
      grd_2024_cleaned.csv
      summary_kpis_by_year.csv       KPI 1 & 2 (severity, weight) by year
      load_stats_by_year.csv         row counts / exclusions per year (process record evidence)
      admission_summary_by_year.csv  KPI 3 (urgent admission share) by year
```

## Order to open / run

1. Open `analysis/01_grd_severity_analysis.ipynb`.
2. In Section 1 ("Configuration"), update `DATA_ROOT` to the local path where the course's GRD source files are stored.
3. Run all cells top to bottom (`Restart Kernel and Run All`). The notebook is organized in 8 sections: setup, data-quality checks (Five Cs), cleaning pipeline, research-question note, KPI calculation, interpretation, visualizations, and export of `data/input` and `data/output`.
4. Read `report/GROUP_03_C1_Report.pdf` for the full write-up; the notebook supports and reproduces the figures/tables cited there, but the report is self-contained and does not require opening the notebook to be understood.

## Software and dependencies

- Python 3.13
- Libraries: `pandas`, `matplotlib`, `nbformat` (only needed if regenerating the notebook file itself, not for running the analysis)
- No `requirements.txt` included; install via:
  ```
  pip install pandas matplotlib
  ```

## Data sources and versions

- **GRD (Diagnosis-Related Groups), 2021, 2023, 2024:** FONASA/DEIS open data, files named `GRD_PUBLICO_<year>.txt`, pipe-separated (`|`), latin1 encoding for 2021 and 2024, UTF-16 for 2023.
- **GRD 2022:** published by FONASA as `GRD_PUBLICO_EXTERNO_2022.txt` (UTF-16 encoding) instead of `GRD_PUBLICO_2022`. Verified via `PREVISION` distribution (Section 2 of the notebook) to represent the same public-network population as other years, not a distinct cohort, before including it in the multi-year trend.
- Retrieved via the course's `descargar_datos_W1_actualizado.py` script and manual extraction (some years required 7-Zip for `.rar`/`.zip` archives), from the Chilean Ministry of Health (MINSAL/FONASA/DEIS) open-data repository. Exact download date: 24 September 2026.
- Full original files are not included in this archive due to size; `data/input/*_reduced.csv` are the reduced extracts (needed columns only) that reproduce our cleaning steps. The notebook (Section 1 and 8) documents the exact `usecols` filter used to produce them from the originals.

## Known issues

- A pandas bug (`IndexError` when combining `usecols` with mixed-type inference under the C engine) affected an earlier version of the export step for 2024. Resolved in the current notebook by reading all needed columns as `dtype=str` and converting explicitly afterward. No unresolved execution issues remain as of this submission.
- The duplicate check in the cleaning pipeline (Section 3) is limited to the 5 loaded columns (no patient identifier is loaded), so it is a coarse signal rather than a full record-level duplicate check. Documented as a limitation in the notebook and report.
- 2021 shows two unexplained anomalies relative to other years (inverse severity pattern; higher count of unmapped `PREVISION` categories) — both are documented as open observations in the notebook (Section 6), not resolved further given time constraints.

## AI use

Claude (Anthropic) was used throughout preparation: troubleshooting local environment/SSL/encoding errors, exploring data dictionaries and raw file structure, drafting and debugging analysis code, and organizing this README and the report. All cited documentary sources were independently verified at their original URLs before being used. All data findings (encodings, malformed lines, KPI results) come from code executed by the team against the real files, not from AI-generated figures. See the report's Process Record for tool/prompt-level detail.
