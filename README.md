# NeuroPOD

# Drug-Associated Cardiac Arrest & Guideline Utilization before WLST

R analysis script comparing drug-associated (DA) and non-drug-associated out-of-hospital cardiac arrest (OHCA) patients who died after withdrawal of life-sustaining therapy (WLST). The main question is whether prognostication before WLST followed guideline recommendations.

**Analysis period:** 22 Mar 2026 to 04 Aug 2026
**Data cleaning:** Claude Sonnet 5.5 (24 Oct 2026)

## Requirements

- R (recent version) and the following packages: `tidyverse`, `survival`, `gtsummary`, `flextable`, `officer`, `naniar`, `marginaleffects`, `broom`, `scales`, `gt`, `ComplexUpset`, `patchwork`
- The first section of the script installs them with `install.packages()`. You can comment this out after the first run.

## Input

- `Insert Your CSV File`: one row per patient. The path is hardcoded near the top of the script (`read.csv(...)`), so update it for your machine.
- The coding legend for every categorical variable is in the `RECODE CATEGORICAL VARIABLES` section. "Unknown" codes are converted to `NA` for `housing`, `hist_drug`, `witnessed` and `pat_outcome`.

## Cohort flow

| Step | Filter | N remaining | Excluded |
|---|---|---|---|
| All records | Non-missing `status` (excludes non-included hospital sites, age <19, missing identifiers) | n | n |
| Deceased | `status == "Deceased"` | n | n |
| WLST | `cause_death == "WLST"` | n | n |
| Admitted beyond ER (`data_main`) | `hosp_adm != "ER"` | n | n |
| Analysis set (`data_plotrosc`) | ROSC-to-death < 1500 hours | see script output | |

## Key definitions

- **Guideline-adherent:** at least 2 poor-prognosis findings **and** death at least 3 days (72 h) after ROSC.
- **Poor-prognosis findings (6):** pupillary light reflex at 72 h, myoclonus, EEG, CT, MRI, SSEP. Missing values are treated as 0 (not positive) representing the absence of this finding.
- **Sensitivity analysis:** CT and MRI are counted as a single neuroimaging modality (5 findings in total).
- **Time to ROSC:** `dispatch_to_rosc`, in minutes. Midnight crossovers in dispatch and ROSC clock times are corrected.

## Script structure

1. **Setup and import:** packages and data load.
2. **Recoding:** factors, arrest location (residential / non-residential), initial rhythm (shockable / non-shockable / other).
3. **Derived time variables:** ICU-to-outcome, time to ROSC, dispatch-to-ROSC, ROSC-to-outcome.
4. **Cohort filtering:** produces `data_deceased`, `data_wlst`, `data_main` and `data_plotrosc`.
5. **Descriptives:**
   - Death at or after 72 h, by DA status.
   - Prognostic tests performed (CT, EEG, SSEP, MRI).
   - Distribution of the number of poor-prognosis findings.
6. **Guideline adherence:** chi-square test and two-sample proportion test, then the CT/MRI-combined sensitivity analysis.
7. **Table 1:** demographics and arrest characteristics by DA status.
8. **Logistic regression** (outcome: guideline adherence):
   - Unadjusted: one model per variable.
   - Minimally adjusted: age, sex, hospital site, arrest year.
   - Fully adjusted: the above plus arrest location, initial rhythm, time to ROSC, witnessed status, bystander CPR.
9. **Prognostic tests and findings tables:** Fisher's exact tests by DA status.
10. **ROSC-to-death summary** (21 days or less): median and IQR by DA status.
11. **UpSet plots:** combinations of prognostic tests used, for DA and non-DA patients.
12. **Marginal probabilities:** standardized adherence probabilities and risk differences, plus odds ratios, via `marginaleffects`.

## Outputs

| File | Contents |
|---|---|
| `Table1.docx` | Table 1 (working directory) |
| `~/Downloads/odds_ratio_table.docx` | Unadjusted, minimally adjusted and fully adjusted ORs |
| `marginal_probabilities.docx` | Standardized marginal probabilities (working directory) |

Other results (chi-square tests, summary statistics, UpSet plots, the prognostic tests and findings tables) print to the console or plot window and are not saved by the script.

## Notes

- Run the script top to bottom. Later sections depend on objects created earlier (e.g. `data_guidadh`, `model_minimal`, `model_full`).
- The chi-square tests use no continuity correction. The script checks that all expected cell counts are at least 5.
- `drug_assoc` is the exposure of interest. The reference level is "No" (non-DA).
