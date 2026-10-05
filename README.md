# Multimodal Non-Invasive Diabetes Classification Dataset

[![DOI](https://img.shields.io/badge/DOI-YOUR__DOI__HERE-blue)](https://doi.org/YOUR_DOI_HERE)
[![Data](https://img.shields.io/badge/Data-Mendeley%20Data-red)](YOUR_MENDELEY_URL_HERE)
![Rows](https://img.shields.io/badge/rows-15%2C720-informational)
![Columns](https://img.shields.io/badge/columns-56-informational)
<!-- Add a license badge once the license is confirmed (see "License" below). -->

A harmonized, tabular dataset for **non-invasive diabetes-status classification**. It merges five source collections into one 56-column schema covering three sensing tiers: near-infrared (NIR) optical signals, wearable physiological signals, and demographic/environmental covariates.

> **Read this first:** 43.9% of the rows are synthetic, and real/synthetic status is fully determined by source collection. Baseline accuracy is near ceiling (AUC up to 0.999). Part of that comes from the synthetic sources and from source-specific artifacts, not from physiology alone. See [Known limitations and usage warnings](#known-limitations-and-usage-warnings) before benchmarking anything.

![Dataset overview](assets/dataset_overview.png)

---

## Table of contents

1. [At a glance](#at-a-glance)
2. [Source collections](#source-collections)
3. [Feature tiers](#feature-tiers)
4. [Column schema](#column-schema)
5. [Label definition](#label-definition)
6. [Train / validation / test split](#train--validation--test-split)
7. [Quick start](#quick-start)
8. [Baseline results](#baseline-results)
9. [Known limitations and usage warnings](#known-limitations-and-usage-warnings)
10. [Repository structure](#repository-structure)
11. [Data access](#data-access)
12. [Citation](#citation)
13. [License](#license)
14. [References](#references)
15. [Authors and contact](#authors-and-contact)
16. [Acknowledgements](#acknowledgements)

---

## At a glance

| Property | Value |
| --- | --- |
| Rows | 15,720 (one row = one measurement session) |
| Columns | 56 |
| Source collections | 5 |
| Real / synthetic rows | 8,826 (56.1%) / 6,894 (43.9%) |
| Target | Binary diabetic status: 10,480 diabetic (66.7%), 5,240 non-diabetic (33.3%) |
| Blood glucose range | 70.3 to 299.8 mg/dL |
| Missing values | None (36 source-exclusive columns are imputed, see [limitations](#known-limitations-and-usage-warnings)) |
| Duplicate rows | None |
| Format | CSV, plus `data_dictionary.csv` and an EDA notebook |
| Source institutions | USA, Vietnam, Mexico, Norway (four named); one undisclosed |
| Subjects | Computer Science; Biomedical Engineering |

## Source collections

The dataset is a **secondary compilation**. No new primary human-subjects data were collected by the authors.

| Source collection | Rows | Share | Status | Modality / notes |
| --- | ---: | ---: | --- | --- |
| PhysioCGM [1] | 6,993 | 44.5% | Real | Wearable physiology and CGM, Type 1 diabetes participants. Provides the Tier 2 columns. |
| Nature_Scientific_Reports_NIR_Glucose [3] | 5,920 | 37.7% | **Synthetic** | Generated to emulate the 660/940 nm optical schema of the cited study. Not the original measurements. |
| Kaggle_Raman_Diabetes [2] | 974 | 6.2% | **Synthetic** | Generated to emulate the Raman screening schema of the cited study. Not the original measurements. |
| Raman_Sugars | 933 | 5.9% | Real | Raman spectroscopy of sugars (GitHub-hosted, no peer-reviewed publication identified). |
| NTNU_NIR_Glucose | 900 | 5.7% | Real | NIR spectroscopy of aqueous glucose (NTNU Dataverse). |
| **Total** | **15,720** | **100%** | | |

Original sources:

- PhysioCGM: <https://springernature.figshare.com/articles/dataset/PhysioCGM_a_multimodal_physiological_dataset_for_non-invasive_blood_glucose_estimation/28136294>
- NTNU aqueous-glucose NIR: <https://doi.org/10.18710/NSHFAK>
- Raman_Sugars: <https://github.com/Alvaro-FG/Raman_Sugars>
- Raman Spectroscopy of Diabetes (Kaggle): <https://www.kaggle.com/datasets/codina/raman-spectroscopy-of-diabetes>
- Multi-wavelength NIR sensing system: <https://doi.org/10.1038/s41598-025-28673-4>

## Feature tiers

Features are grouped by sensing cost and invasiveness, so you can study accuracy versus sensor complexity.

| Tier | Group | Features | Association with label (Pearson r) |
| --- | --- | --- | --- |
| 1 | Optical (NIR) | NIR 660 nm, NIR 940 nm, optical ratio (660/940) | Strongest: NIR 940 = -0.72, NIR 660 = -0.67, ratio = +0.45 |
| 2 | Wearable physiological | Heart rate (avg / max / instantaneous), HR variability, BVP variability, skin conductance, movement intensity | Weak: all \|r\| <= 0.13 |
| 3 | Demographic / environmental | BMI, age, SpO2, skin temperature, ambient temperature, wrist temperature | Near zero: all \|r\| <= 0.05 |

Selected correlations with the diabetic label (from the all-column ranking):

| Feature | r | Feature | r |
| --- | ---: | --- | ---: |
| NIR 940 nm | -0.72 | HR Variability | 0.05 |
| NIR 660 nm | -0.67 | Skin Conductance Avg (µS) | 0.03 |
| Optical Ratio (660/940) | 0.45 | BMI (kg/m²) | 0.02 |
| Row Completeness Score* | -0.12 | Movement Intensity Avg | 0.01 |
| BVP Variability | -0.10 | Wrist Temp Avg (°C) | 0.00 |
| BVP Max | -0.09 | Volunteer Age (years) | -0.02 |
| HR Min (bpm) | 0.12 | SpO2 (%) | -0.03 |
| HR Avg (bpm) | 0.10 | Ambient Room Temp (°C) | 0.00 |
| Heart Rate (instantaneous) | 0.10 | HR Max (bpm) | 0.09 |

\* Row completeness score is a preprocessing quality indicator, not a physiological or optical measurement. Do not use it as a feature.

## Column schema

Column names follow a `category__description` convention. The 56 columns break down as:

| Group | Count | Notes |
| --- | ---: | --- |
| Optical / NIR | 3 | Tier 1 features |
| `physiological__*` | 3 | Heart rate (Tier 2), SpO2 and skin temperature (Tier 3) |
| `shared__*` | 3 | **Duplicate copies of blood glucose, heart rate, and the diabetic label** |
| Demographic | 2 | Age, BMI |
| Single-occurrence columns | 11 | Binary label, auxiliary final-label field, volunteer ID, reading count, volunteer count, sensor type, environmental context, data-type flag, target variable, source-dataset name, canonical split |
| Auxiliary / per-source fields | 36 | Retained from the original collections; imputed for rows from other sources |

The full machine-readable dictionary (column name, inferred type, non-null count, category) is in [`data_dictionary.csv`](data/data_dictionary.csv).

## Label definition

`canonical_label` is binary: **1 = diabetic, 0 = non-diabetic**. It was derived from blood glucose using the ADA fasting threshold: label = 1 if glucose > 126 mg/dL, else 0.

| Source | Rows | Label/threshold mismatches | Mismatch % |
| --- | ---: | ---: | ---: |
| Kaggle_Raman_Diabetes | 974 | 1 | 0.10% |
| NTNU_NIR_Glucose | 900 | 4 | 0.44% |
| PhysioCGM | 6,993 | 27 | 0.39% |
| Raman_Sugars | 933 | 4 | 0.43% |
| Nature_Scientific_Reports_NIR_Glucose | 5,920 | 621 | **10.49%** |

Four sources follow the threshold almost exactly. The Nature_Scientific_Reports_NIR_Glucose labels follow an additional, undocumented generation rule.

## Train / validation / test split

A pre-assigned, label-stratified partition is stored in `canonical_split`.

| Split | Rows | Share |
| --- | ---: | ---: |
| Train | 11,004 | 70.0% |
| Validation | 2,358 | 15.0% |
| Test | 2,358 | 15.0% |

The split is stratified by diabetic label. The documentation does not state that it is grouped by volunteer or source. See the warning below.

## Quick start

```bash
pip install pandas scikit-learn xgboost
```

```python
import pandas as pd

df = pd.read_csv("data/dataset.csv")  # adjust to your CSV filename
print(df.shape)  # (15720, 56)

LABEL = "canonical_label"
SPLIT = "canonical_split"
SOURCE = "source_dataset"  # adjust to the source-name column listed in data_dictionary.csv

# Check the exact split values before filtering
print(df[SPLIT].value_counts())

# Drop anything that encodes or duplicates the target (glucose copies, label copies,
# target variable, label_has_DM2, ...) and non-measurement bookkeeping columns.
leaky = [c for c in df.columns
         if any(k in c.lower() for k in ("glucose", "label", "target", "completeness"))]
drop = set(leaky) | {SPLIT, SOURCE}
features = [c for c in df.select_dtypes("number").columns if c not in drop]

train = df[df[SPLIT] == "train"]  # verify spelling against value_counts() above
test = df[df[SPLIT] == "test"]

from xgboost import XGBClassifier
from sklearn.metrics import roc_auc_score

model = XGBClassifier(random_state=42).fit(train[features], train[LABEL])
print("AUC:", roc_auc_score(test[LABEL], model.predict_proba(test[features])[:, 1]))
```

Real-data-only evaluation, which is the more honest difficulty estimate:

```python
REAL = ["PhysioCGM", "NTNU_NIR_Glucose", "Raman_Sugars"]
real = df[df[SOURCE].isin(REAL)]
```

Also try leave-one-source-out evaluation to see how much of the score is source identity rather than glucose-related signal.

## Baseline results

Eight scikit-learn / XGBoost models with library-default hyperparameters, trained on the train split (11,004 rows) and evaluated on the test split (2,358 rows). Blood glucose is **not** an input feature. Ranked by AUC.

| Model | Accuracy | Precision | Recall | F1 | AUC | Time (s) |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Random Forest | 0.9905 | 0.9966 | 0.9890 | 0.9928 | 0.9989 | 7.124 |
| XGBoost | 0.9924 | 0.9981 | 0.9905 | 0.9943 | 0.9985 | 0.839 |
| Gradient Boosting | 0.9822 | 0.9890 | 0.9843 | 0.9866 | 0.9969 | 6.354 |
| SVM (RBF) | 0.9707 | 0.9695 | 0.9871 | 0.9783 | 0.9959 | 4.257 |
| K-Nearest Neighbors | 0.9711 | 0.9985 | 0.9580 | 0.9778 | 0.9929 | 0.002 |
| Decision Tree | 0.9831 | 0.9904 | 0.9843 | 0.9873 | 0.9826 | 0.246 |
| Naive Bayes | 0.9192 | 0.9411 | 0.9375 | 0.9393 | 0.9604 | 0.008 |
| Logistic Regression | 0.9128 | 0.9281 | 0.9423 | 0.9351 | 0.9580 | 0.028 |

The majority-class baseline is 0.667 accuracy. These numbers are a sanity check on the schema from a single split, not an upper bound or a clinical performance estimate.

## Known limitations and usage warnings

1. **Real/synthetic is confounded with source.** Kaggle_Raman_Diabetes and Nature_Scientific_Reports_NIR_Glucose (43.9% of rows) are fully synthetic; the other three are fully real. A two-sample Kolmogorov-Smirnov test found significant real-versus-synthetic differences in all 16 numeric Tier 1-3 features (largest for NIR 940 nm, D = 0.57, and NIR 660 nm, D = 0.51). This mixes synthetic-generation effects with true between-study differences in populations, sensors, and protocols. Treat real and synthetic rows as separate strata.
2. **Near-ceiling baselines are partly an artifact.** The synthetic sources contribute more cleanly separable classes than the real ones. Report results on the real-only subset alongside full-dataset results.
3. **Target-leaking columns are present.** The CSV contains glucose and label duplicates (`shared__*`), a target-variable column, and an auxiliary final-label field. Remove them before training, as in the quick start.
4. **36 of 56 columns are imputed placeholders** for rows from sources that do not natively have them (for example Sucrose, Fructose, Maltose, `n_spectral_channels`, `label_has_DM2`). They are not sensor measurements. Do not use them for source-specific analysis or as general features.
5. **Possible source-identity shortcuts.** Several features show a sharp single-value spike in one class: BVP variability near 100, skin conductance near 0, and movement intensity near 62-65 in non-diabetic rows; BMI near 29.25 and wrist temperature near 33.6 in diabetic rows. This pattern is consistent with source-specific constants or imputation, so a model can learn the source rather than physiology.
6. **The split is not documented as subject-wise.** It is stratified by label. Rows include a volunteer identifier and multiple sessions per volunteer, so the same volunteer may appear in train and test. Verify, and use grouped splits if it matters for your claim.
7. **Inconsistent labeling for one source.** Nature_Scientific_Reports_NIR_Glucose labels disagree with the 126 mg/dL rule in 10.49% of rows.
8. **Single split, default hyperparameters.** No cross-validation or tuning was done.
9. **No raw waveforms.** Only session-level summary features are released, so the data does not suit models that need raw signal input.
10. **Not for clinical use.** This is a research dataset. It must not be used for diagnosis or treatment decisions.

## Repository structure

> Adjust this to match what you actually commit.

```
.
├── README.md
├── LICENSE
├── CITATION.cff
├── assets/
│   └── dataset_overview.png
├── data/
│   ├── dataset.csv               # 15,720 rows x 56 columns
│   └── data_dictionary.csv       # column name, dtype, non-null count, category
├── notebooks/
│   └── eda.ipynb                 # data dictionary, audits, plots, outliers, baselines
└── figures/                      # correlation heatmaps, distributions, boxplots
```

If the CSV is large, use Git LFS or point to the Mendeley download instead of committing it directly.

## Data access

The CSV, data dictionary, and EDA notebook are available from Mendeley Data. No registration is required.

- Repository: Mendeley Data
- DOI: [YOUR_DOI_HERE](https://doi.org/YOUR_DOI_HERE)
- URL: <YOUR_MENDELEY_URL_HERE>

## Citation

If you use this dataset, please cite it, **and** cite the original source collections you rely on.

```bibtex
@dataset{rahman_hira_rana_2026_multimodal_diabetes,
  author    = {Rahman, Anichur and Hira, Md Irfanul Kabir and Rana, Md Shohel},
  title     = {A Multimodal Non-Invasive Dataset Integrating Near-Infrared Optical
               Signals, Wearable Physiological Signals, and Demographic Features
               for Diabetes Classification},
  year      = {2026},
  publisher = {Mendeley Data},
  version   = {1},
  doi       = {YOUR_DOI_HERE}
}
```

<!-- Update year/venue once the Data in Brief article is published. -->

## License

**TODO: confirm before publishing the repo.** Check the license on the Mendeley Data record and use the same one here. Each source collection also has its own terms (PhysioCGM, Kaggle, NTNU, and the Raman_Sugars GitHub repository); confirm that redistribution of the merged data is allowed under each.

## References

1. W. Quamer et al., A multimodal physiological dataset for non-invasive blood glucose estimation, *Scientific Data* 12 (2025) 1822. <https://doi.org/10.1038/s41597-025-06090-6>
2. E. Guevara et al., Use of Raman spectroscopy to screen diabetes mellitus with machine learning tools, *Biomedical Optics Express* 9 (2018) 4998-5010. <https://doi.org/10.1364/BOE.9.004998>
3. A.T.P. Nguyen et al., The open-source multi-wavelength non-invasive blood glucose sensing system, *Scientific Reports* 15 (2025) 45731. <https://doi.org/10.1038/s41598-025-28673-4>
4. S.V.K.R. Rajeswari, P. Vijayakumar, Development of sensor system and data analytic framework for non-invasive blood glucose prediction, *Scientific Reports* 14 (2024) 9206. <https://doi.org/10.1038/s41598-024-59744-7>
5. H. Jiang, T. Yao, C. Ding, PPG-based glucose sensors: a review, *Artificial Intelligence Review* 58 (2025) 391. <https://doi.org/10.1007/s10462-025-11379-4>
6. J.P. Singh et al., Association of hyperglycemia with reduced heart rate variability (The Framingham Heart Study), *Am. J. Cardiology* 86 (2000) 309-312. <https://doi.org/10.1016/S0002-9149(00)00920-6>
7. K. Khan et al., Non invasive blood glucose estimation using green light photoplethysmography and machine learning, *Frontiers in Digital Health* 8 (2026) 1705086. <https://doi.org/10.3389/fdgth.2026.1705086>
8. X. Zhang et al., Machine-learning assisted glucose measurement system based on infrared photoacoustic spectroscopy, *Optics and Lasers in Engineering* 195 (2025) 109365. <https://doi.org/10.1016/j.optlaseng.2025.109365>
9. M.H. Al-Jammas et al., A non-invasive blood glucose monitoring system, *Computers in Biology and Medicine* 191 (2025) 110133. <https://doi.org/10.1016/j.compbiomed.2025.110133>
10. T.W. Bae et al., A feasibility study on noninvasive blood glucose estimation using machine learning analysis of near-infrared spectroscopy data, *Biosensors* 15 (2025) 711. <https://doi.org/10.3390/bios15110711>
11. J. Liu et al., In vivo Raman spectroscopy for non-invasive transcutaneous glucose monitoring on animal models and human subjects, *Spectrochimica Acta Part A* 329 (2025) 125584. <https://doi.org/10.1016/j.saa.2024.125584>
12. R.A. Fraser et al., Integration of artificial intelligence and wearable technology in the management of diabetes and prediabetes, *npj Digital Medicine* 8 (2025) 687. <https://doi.org/10.1038/s41746-025-02036-9>

## Authors and contact

- **Anichur Rahman** (corresponding author), School of Computing, Georgia Southern University, USA. ar36248@georgiasouthern.edu
- **Md Irfanul Kabir Hira**, Department of CSE, NITER (University of Dhaka), Bangladesh
- **Md Shohel Rana**, School of Computing, Georgia Southern University, USA

Issues and questions: please open a GitHub issue.

## Acknowledgements

Thanks to the creators and maintainers of the five source collections that made this compilation possible: PhysioCGM, the Raman-spectroscopy diabetes-screening dataset, the 660/940 nm multi-wavelength NIR sensing dataset, Raman_Sugars, and the NTNU aqueous-glucose NIR dataset. This work received no specific grant from funding agencies in the public, commercial, or not-for-profit sectors.
