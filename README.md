# EEG-Based Classification of Parkinson's Disease

This project explores the use of resting-state EEG signals to distinguish participants with Parkinson's disease (PD) from healthy controls using signal processing, spectral feature extraction, and machine-learning methods.

The analysis was developed as part of a Neuroengineering and Neuroimaging project within the Master's in Health Data Science at Universitat Rovira i Virgili (URV).

## Dataset

The project uses the publicly available **Rest eyes open** EEG dataset from OpenNeuro (`ds004584`, version 1.0.0), containing recordings from 149 participants:

- 100 participants with Parkinson's disease
- 49 healthy controls

The raw EEG dataset is not included in this repository.

**Dataset:** [OpenNeuro ds004584](https://openneuro.org/datasets/ds004584/versions/1.0.0)  
**DOI:** [10.18112/openneuro.ds004584.v1.0.0](https://doi.org/10.18112/openneuro.ds004584.v1.0.0)

To reproduce the complete workflow, download the dataset and place it under:

`data/ds004584/`

## Workflow

The analysis follows this general pipeline:

**Raw EEG → Filtering → Re-referencing → ICA artifact removal → Epoching → Spectral feature extraction → Subject-independent train/test split → Feature selection → Classification**

### EEG preprocessing

EEG recordings were processed individually while preserving participant identity throughout the workflow.

Preprocessing included:

- Band-pass filtering between 1 and 40 Hz
- EEG re-referencing
- ICA-based artifact removal
- Manual inspection of ICA components for each participant
- Segmentation into 2-second epochs

ICA component exclusions were recorded by participant and are included in `ica_component_selection.csv`.

### Feature extraction

Features were extracted independently from each EEG epoch and included:

- Mean, standard deviation, RMS, and peak-to-peak amplitude
- Absolute spectral power in delta, theta, alpha, and beta bands
- Relative band power
- Alpha/theta and theta/beta power ratios

This resulted in **840 candidate EEG features**.

### Feature selection and validation

Participants were divided into training and test sets before feature selection. All epochs belonging to the same participant remained within the same partition to prevent subject leakage.

Mutual Information was calculated using the training data, and the 50 highest-ranked features were selected for model development.

Model robustness was additionally evaluated using 5-fold GroupKFold cross-validation, with participant identity used as the grouping variable.

## Models

Four classifiers were evaluated:

- Logistic Regression
- Support Vector Machine (RBF kernel)
- Random Forest
- Multilayer Perceptron (MLP)

## Results

Random Forest achieved the strongest overall performance among the evaluated models.

| Model | Test Accuracy | Test F1 | Cross-validation F1 |
|---|---:|---:|---:|
| Random Forest | 0.732 | 0.810 | 0.736 ± 0.012 |
| MLP | 0.686 | 0.750 | 0.712 ± 0.044 |
| Logistic Regression | 0.662 | 0.688 | 0.650 ± 0.068 |
| SVM | 0.609 | 0.590 | 0.594 ± 0.084 |

The selected features showed a notable representation of theta-band spectral information across multiple EEG channels.

## Interpretation and limitations

The results suggest that spectral characteristics of resting-state EEG contain information that can contribute to distinguishing Parkinson's disease from healthy controls.

Predictions in this analysis were performed at the **epoch level**, while train/test separation and cross-validation were controlled at the **participant level**. Therefore, the reported performance should not be interpreted as patient-level diagnostic accuracy.

Further work could include subject-level prediction through epoch aggregation, nested feature selection and model validation, and evaluation on an independent dataset.

## Repository contents

- `parkinsons_eeg_classification.ipynb` — complete analysis workflow
- `ica_component_selection.csv` — participant-specific ICA component exclusions
- `master_table_basic.csv` — participant labels and dataset organization
- `mi_feature_ranking.csv` — Mutual Information feature ranking
- `mi_feature_ranking_top50_base.csv` — feature ranking used for the Top 50 analysis
- `mi_selected_features_top50.csv` — features selected for model development
- `model_metrics_top50.csv` — model performance results
- `.gitignore` — excludes raw data, large intermediate files, and temporary files

The intermediate feature dataset (`eeg_features_dataset.csv`) is also excluded from the repository because of its size. It can be regenerated from the raw EEG data using the analysis notebook.

## Tools

The analysis was developed in Python using tools and libraries including:

- MNE-Python
- NumPy
- pandas
- SciPy
- scikit-learn
- Matplotlib
- Jupyter Notebook

## Reproducibility

The repository includes the analysis notebook and the participant-specific ICA component selections used during preprocessing.

Because the raw EEG dataset and the large intermediate feature dataset are not stored in the repository, reproducing the complete workflow requires downloading the original OpenNeuro dataset first.

The saved ICA component selections allow the manually reviewed artifact-removal decisions to be reused without repeating the complete ICA component inspection process.

## Note

This project is intended as an exploratory data science and neuroengineering analysis. The models and results are not intended for clinical diagnosis or clinical decision-making.