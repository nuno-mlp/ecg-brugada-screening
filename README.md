# ECG Brugada Screening

Python machine learning pipeline for ECG-based Brugada syndrome screening using the Brugada-HUCA 12-lead ECG dataset.

This project explores whether ECG morphology features extracted from standard 12-lead recordings can help distinguish Brugada type 1 cases from non-Brugada controls. It uses information from the right-precordial leads, where Brugada-related ST-T abnormalities are typically assessed.

This is a research and portfolio project. It is not intended for clinical diagnosis or direct medical use.

## Background

Brugada syndrome is a rare, potentially life-threatening inherited cardiac disorder that predisposes primarily young, otherwise healthy individuals to ventricular arrhythmias and sudden cardiac death. A 12-lead ECG is central to the diagnosis of Brugada syndrome. Three ECG patterns have historically been described. Type 1 is characterized by coved ST-segment elevation greater than 2 mm in at least one right-precordial lead (`V1`–`V3`), followed by a negative T wave, and is the only pattern considered diagnostic. This pattern may occur spontaneously or be induced by sodium-channel blocker testing. Saddleback-like patterns, historically described as type 2 or type 3, are not considered diagnostic on their own.[^brugada-statpearls]

[^brugada-statpearls]: Shukla K, Basile EJ, Van Name JP, et al. *Brugada Syndrome*. [Updated 2025 Dec 1]. In: StatPearls [Internet]. Treasure Island (FL): StatPearls Publishing; 2026 Jan-. Available from: https://www.ncbi.nlm.nih.gov/books/NBK519568/

## Dataset

The project uses Brugada-HUCA (version 1.0.0), a public PhysioNet dataset containing 363 standard 12-lead ECG recordings: 76 confirmed Brugada syndrome cases and 287 normal controls. Each recording is 12 seconds long and sampled at 100 Hz.

The raw dataset is not included in this repository. To reproduce the workflow, download the dataset separately and place the raw files under:

```text
data/raw/
```

## Repository structure

```text
.
├── data/
│   ├── raw/                  # Raw dataset files, not versioned
│   └── processed/            
├── notebooks/
│   ├── 01_dataset_inspection.ipynb
│   ├── 02_ecg_morphology_and_feature_extraction.ipynb
│   ├── 03_baseline_modeling_and_evaluation.ipynb
│   └── 04_oof_error_analysis.ipynb
├── results/
│   ├── figures/
│   └── *.csv
├── src/
├── requirements.txt
└── README.md
```

## Workflow

The project is organized as a notebook-based pipeline. Each notebook produces files that are used by the next stage.

1. **Dataset inspection**
   Loads the original Brugada-HUCA metadata, interprets the available labels, checks the ECG file structure, and exports the processed metadata table.

   **Output**

   ```text
   data/processed/metadata_interpreted.csv
   ```

2. **ECG morphology and feature extraction**
   Loads the interpreted metadata and raw ECG records, extracts R-peak-centered beat windows, builds representative median beats, performs beat-level and median-beat quality control, and exports ECG morphology features from the right-precordial leads.

   **Outputs**

   ```text
   data/processed/ecg_morphology_features.csv
   data/processed/ecg_morphology_feature_qc.csv
   ```

3. **Baseline modeling and evaluation**
   Loads the extracted morphology features and QC table, prepares the dataset for modelling, compares `V1`–`V2` and `V1`–`V3` feature sets across baseline machine learning models, and generates out-of-fold predictions for error analysis.

   **Outputs**

   ```text
   results/baseline_cv_results.csv
   results/baseline_feature_set_comparison.csv
   results/baseline_oof_error_analysis.csv
   results/oof_misclassified_records.csv
   ```

4. **Out-of-fold error analysis**
   Reviews selected out-of-fold misclassified records to understand where the baseline model fails and why.

   **Outputs**

   ```text
   results/oof_error_review_selected.csv
   results/oof_error_review_summary.csv
   results/oof_error_review_qc_summary.csv
   results/figures/
   ```

## Setup

Create a Conda environment and install the dependencies:

```bash
conda create -n ecg-brugada python=3.11
conda activate ecg-brugada
pip install -r requirements.txt
```

From the project root, start Jupyter Notebook:

```bash
jupyter notebook
```

Then run the notebooks in order:

```text
notebooks/01_dataset_inspection.ipynb
notebooks/02_ecg_morphology_and_feature_extraction.ipynb
notebooks/03_baseline_modeling_and_evaluation.ipynb
notebooks/04_oof_error_analysis.ipynb
```

## Baseline performance

Among the tested baselines, the best result came from logistic regression using the `V1`–`V3` morphology feature set. This model was evaluated on 361 records retained by the processing pipeline, including 74 Brugada-positive cases.


| Metric | Value |
|---|---:|
| Balanced accuracy, mean CV | 0.862 |
| Balanced accuracy, CV std | 0.048 |
| ROC-AUC           |         0.916 |
| Recall            |         0.839 |
| Precision         |         0.653 |
| F1-score          |         0.732 |

These results suggest that the extracted ECG morphology features contain useful signal for distinguishing Brugada-positive records from controls, although the baseline still misses some positive cases and produces a moderate number of false positives.

## Error analysis summary

The reviewed errors were usually not caused by obvious signal-processing failures: R-peak placement, beat acceptance, and median-beat quality were generally usable in the selected records.

The main error modes identified so far were:

* the current beat-extraction window may truncate or underrepresent later repolarization morphology in some ECGs; 
* the current feature set has difficulty separating clear type 1 Brugada morphology from saddleback-like or atypical right-precordial patterns;
* selected records showed possible mismatch between dataset labels and visible ECG morphology

## Limitations

This is an exploratory ECG-only machine learning project. It does not use the full clinical information required for Brugada syndrome diagnosis or risk stratification.

The dataset is relatively small, with 363 ECG records and 76 Brugada-positive cases, which limits the strength of the performance conclusions. Error analysis also suggested possible mismatch between dataset labels and visible ECG morphology in selected records; these cases should be interpreted as candidates for further review, not as definitive label errors.

ECG morphology was extracted using a custom automated pipeline, not a clinically validated ECG delineation algorithm. The current feature extraction is based on fixed R-peak-centered windows and may truncate T-wave morphology in some ECGs.

## Dataset citation

Costa Cortez, N., & Garcia Iglesias, D. (2026). *Brugada-HUCA: 12-Lead ECG Recordings for the Study of Brugada Syndrome* (version 1.0.0). PhysioNet. RRID:SCR_007345. https://doi.org/10.13026/0m2w-dy83

## Author

Nuno Pires

