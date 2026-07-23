# RNA-seq Immunotherapy Response Prediction

This repository contains a Google Colab notebook for preprocessing an RNA-seq raw count matrix and applying a pretrained 51-gene logistic regression model to predict response to anti-PD-1/PD-L1 immunotherapy. The workflow was developed for an independent clinical cohort of patients from Peru and Colombia with advanced solid tumors.

## Repository contents

```text
.
├── PredictionModel_English.ipynb
├── modelo_logistico1.pkl
└── README.md
```

- `PredictionModel_English.ipynb`: data preprocessing, quality-control exploration, prediction, performance evaluation, and visualization.
- `modelo_logistico1.pkl`: pretrained logistic regression model based on 51 genes.
- `README.md`: instructions for running the workflow.

The raw count matrix is not included in the repository and must be provided separately.

## Main workflow

The notebook performs the following steps:

1. Mounts Google Drive in Google Colab.
2. Loads the pretrained model.
3. Imports the RNA-seq raw count matrix.
4. Removes missing values and converts the data to numeric format.
5. Generates exploratory quality-control plots.
6. Converts raw counts to counts per million (CPM).
7. Applies the `log2(CPM + 1)` transformation.
8. Selects and orders the 51 genes required by the model.
9. Generates predicted response probabilities and classifications.
10. Evaluates performance using the AUC, classification report, and confusion matrix when clinical response labels are available.
11. Produces publication-ready visualizations, including SHAP plots and cohort-level performance figures.

## Input data

The input file must be a comma-separated count matrix in `.csv` format:

- Rows: genes.
- Columns: patient samples.
- First column: gene identifiers or gene symbols.
- Values: non-negative raw RNA-seq counts.

Example:

```text
Gene,BIA001,BIA002,BIA003
SLC6A12,125,74,210
BICC1,35,18,42
PAPLN,402,315,287
```

Gene names must match those expected by the model. All 51 required genes must be present, and their columns must be ordered exactly as they were during model training. The notebook contains the required gene list.

## Running the notebook in Google Colab

1. Upload the following files to Google Drive:

   - `PredictionModel_English.ipynb`
   - `modelo_logistico1.pkl`
   - Your raw count matrix, for example `matrizCruda.csv`

2. Open the notebook in Google Colab.

3. Update the model and count-matrix paths:

```python
ruta_modelo = "/content/drive/MyDrive/your-folder/modelo_logistico1.pkl"

val = pd.read_csv(
    "/content/drive/MyDrive/your-folder/matrizCruda.csv",
    header=0,
    index_col=0,
    sep=","
)
```

4. Update `lista_filas` with the sample identifiers to be analyzed.

5. If known clinical outcomes are available, update `yVal` using:

```python
0 = non-responder
1 = responder
```

The order of the values in `yVal` must match the order of the samples in the processed dataframe.

6. Run the cells sequentially from top to bottom.

## Requirements

The workflow uses Python and the following main packages:

- NumPy
- pandas
- Matplotlib
- seaborn
- SciPy
- scikit-learn
- imbalanced-learn
- SHAP
- LightGBM
- scikit-posthocs
- adjustText
- joblib

Most packages are available in Google Colab. Missing dependencies can be installed with:

```python
!pip install shap scikit-posthocs adjustText lightgbm
```

Because serialized scikit-learn models can be version-dependent, using the same scikit-learn version employed during model training is recommended. The notebook includes a cell that prints the relevant Python and package versions.

## Model output

The model returns:

- A predicted class for each sample:
  - `0`: predicted non-responder
  - `1`: predicted responder
- A predicted probability of response between 0 and 1.

Example:

```python
predicted_class = nb.predict(filtered_df2)
response_probability = nb.predict_proba(filtered_df2)[:, 1]
```

The notebook uses a probability threshold of `0.28` for one of the external-cohort evaluations. This threshold should only be changed when supported by a predefined validation strategy.

## Important considerations

- Do not apply an additional transformation if the matrix has already been converted to `log2(CPM + 1)`.
- Verify that all required genes are available before prediction.
- Preserve the exact gene order used during model training.
- Confirm that sample identifiers and clinical labels are aligned.
- The model is intended for research use and has not been established as a standalone clinical diagnostic tool.

## Cohort

The independent cohort comprises patients from Peru and Colombia with advanced solid tumors treated with anti-PD-1/PD-L1 immunotherapy. Pretreatment tumor samples were analyzed by RNA sequencing, and clinical response was assessed according to RECIST 1.1.

## Citation

If you use this code, please cite the associated publication. Citation details will be added once the manuscript is published.

## Contact

For questions about the repository or the prediction workflow, please open an issue in the GitHub repository.
