# Assignment 1 - Diabetes Prediction using Machine Learning

## Objective
Build a Machine Learning classification model to predict whether a patient is likely to have diabetes based on medical attributes.

## Dataset
- File: `diabetes.csv`
- Total Records: 768
- Total Columns: 9
- Features: `Pregnancies`, `Glucose`, `BloodPressure`, `SkinThickness`, `Insulin`, `BMI`, `DiabetesPedigreeFunction`, and `Age`
- Target Variable: `Outcome`
  - 0 - No Diabetes
  - 1 - Diabetes
- Class counts: 500 no-diabetes records and 268 diabetes records

## Technologies Used
- Python
- Pandas
- NumPy
- Matplotlib
- Seaborn
- Scikit-learn
- Jupyter Notebook

## Tasks Completed
1. Data Loading and Exploration
2. Outcome Distribution Visualization
3. Data Preprocessing
4. Handling Zero Values as Missing Measurements
5. Train-Test Split
6. Logistic Regression Model Building
7. Model Training and Prediction
8. Model Evaluation and Performance Analysis

## Preprocessing
- Zero values in Glucose, BloodPressure, SkinThickness, Insulin, and BMI are treated as missing.
- Missing values are imputed using the median.
- Numerical features are standardized.
- Data is split into 80% training and 20% testing sets using stratification.
- Training set: 614 records (400 class 0, 214 class 1).
- Testing set: 154 records (100 class 0, 54 class 1).
- All features are numeric, so categorical encoding is not required.

## Model
Logistic Regression is used because the target has two classes. It provides a suitable, interpretable baseline for binary classification. Imputation and scaling are fitted within a Scikit-learn pipeline using training data only.

## Evaluation Metrics
- Training Accuracy: 0.796
- Testing Accuracy: 0.708
- Precision: 0.600
- Recall: 0.500
- F1-Score: 0.545
- ROC-AUC Score: 0.813

Test confusion matrix (rows are actual labels; columns are predicted labels):

|  | Predicted 0 | Predicted 1 |
|---|---:|---:|
| Actual 0 | 82 | 18 |
| Actual 1 | 27 | 27 |

The test accuracy is lower than the training accuracy by about 8.9 percentage points. The model identifies 27 of 54 diabetes cases in the test set (recall 0.500), so accuracy should be considered alongside recall and the confusion matrix.

## Files
- `Diabetes_Prediction.ipynb` - Complete notebook implementation and visualizations.
- `diabetes.csv` - Dataset used for training and testing.
- `requirements.txt` - Python packages required to run the notebook.

## Platform
Jupyter Notebook (VS Code or Google Colab). For Google Colab, upload `diabetes.csv` before running the notebook.

## Run the Notebook
Install dependencies from this folder if needed:

```bash
python -m pip install -r requirements.txt
```

Open `Diabetes_Prediction.ipynb`, select a Python kernel, and run all cells. The notebook contains code cells; this Markdown file provides the assignment documentation.

> This model is for educational purposes only and is not intended for medical diagnosis.
