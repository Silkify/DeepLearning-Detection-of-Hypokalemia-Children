# ECG-based detection of hypokalemia in children: data preparation and evaluation code

This repository contains the code used to prepare the ECG datasets and to evaluate model predictions for the manuscript "Deep Learning Detection of Electrocardiographic Hypokalemia in Children: Development on Interpretation-Based Labels and External Validation Against Measured Potassium."

**Model code and weights will be added after the patent application is completed.** This covers the model definition (dual-stream network, auxiliary-feature gate, and squeeze-and-excitation blocks), the training scripts, and the trained weights.

## Contents

| File | Description |
|---|---|
| `data_preparation.ipynb` | Band-pass filtering (0.67-40 Hz), amplitude normalization, R-peak detection, clean-beat selection, R-aligned median representative beat, derived ECG features (post-R energy ratio and absolute slope), cohort eligibility steps, and 1:1 ECG-potassium matching for the external cohort |
| `evaluation.ipynb` | Evaluation of saved predictions: fold-wise Platt recalibration, threshold selection, discrimination and threshold-based measures with patient-level bootstrap and Clopper-Pearson CIs, calibration, matching-window and potassium-cutoff analyses, logistic regression comparators, decision curve analysis, model updating, device measurements, and device-generated statements |
| `record_ids_and_flow.ipynb` | Writes the list of ZZU pECG record identifiers with the split assignment and prints the external cohort flow |
| `appendix16_record_ids_split.csv` | ZZU pECG record and patient identifiers with the split assignment (`fold1`-`fold5`, cross-validation folds of the training/validation set; `test`, held-out test set); same file as Multimedia Appendix 16 |
| `zzu_predictions.csv` | Predictions of the final model for all 11,898 ZZU pECG recordings: out-of-fold predictions for the training/validation set (from the fold model that did not train on the record) and the 5 fold-model and ensemble predictions for the held-out test set, before (`raw`) and after fold-wise Platt recalibration (`recalibrated`) |

### Predictions (`zzu_predictions.csv`)

| Column | Description |
|---|---|
| `record_id`, `patient_id`, `split` | Identifiers and split, as in `appendix16_record_ids_split.csv` |
| `hypokalemia_label` | 1, hypokalemia interpretation code (AHA statement 211); 0, otherwise |
| `prob_raw`, `prob_recalibrated` | Predicted probability of hypokalemia before and after fold-wise Platt recalibration (ensemble mean for the test set) |
| `fold1_raw` ... `fold5_recalibrated` | Test set only: probability from each of the 5 fold models |

The prespecified threshold is 0.0213 on `prob_recalibrated` (test set: AUROC 0.873; 15.8% of recordings flagged). Predictions for the external cohort are not shared because they are derived from data that are not publicly available.

## Data

- Development data: the Pediatric ECG Database (ZZU pECG), available at https://doi.org/10.6084/m9.figshare.27078763.
- External data: not publicly available because of institutional policy and the conditions of the institutional review board approval.

## Usage

1. Place the ZZU pECG data as `ch.parquet` and the external cohort data as `ko.parquet`, and run `data_preparation.ipynb`. The notebook writes `rebuild_v3/china_v2.npz`, `rebuild_v3/korea_v2.npz`, and the flow and record tables. The external-cohort column names in the notebook (potassium value and test time) follow the source hospital's export and need to be adjusted for other data.
2. Run `evaluation.ipynb` with the saved prediction files in `predictions/` (see the first cell for the expected files and keys). The development and internal test predictions are provided in `zzu_predictions.csv`; the external analyses require the non-public external data. The device-measurement analysis in the last cell also needs `korea_measures.parquet` (device-reported intervals and axes for the external ECGs, in the same row order as `ko.parquet`). The notebook also writes the patient-level split to `results_evaluation/patient_split.npz`.
3. Run `record_ids_and_flow.ipynb` to print the external cohort flow and write `appendix16_record_ids_split.csv`.

Python 3.10 with numpy, pandas, pyarrow, scipy, scikit-learn, matplotlib, joblib, and tqdm.
