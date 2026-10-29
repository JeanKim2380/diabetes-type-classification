# Diabetes Type Classification (BRFSS 2024)

Classifying **Type 1 vs Type 2 diabetes** among diagnosed adults in the CDC's 2024 Behavioral Risk Factor Surveillance System (BRFSS) survey, and identifying which health, lifestyle and demographic factors most distinguish the two types.

## Problem

Among survey respondents who report a diabetes diagnosis and know their type, can self-reported health and lifestyle answers distinguish Type 1 from Type 2, and which factors matter most? The classes are heavily imbalanced: in the held-out test set, 107 respondents had Type 1 and 1,050 had Type 2.

## Approach

1. **Data cleaning:** kept completed interviews and respondents with a known diabetes type; dropped columns with missing values and administrative/duplicate fields; removed "refused / don't know" answers per column using the BRFSS codebook value meanings; converted survey codes into real values (metric weights and heights to pounds and inches, feet-and-inches heights to inches, "none" to 0, and "never" to the end of its ordered scale).
2. **Mixed-type association analysis:** chose a correlation measure for each pair of variable types (Spearman, phi, Cramér's V, point-biserial) to remove redundant features.
3. **Interpretable baseline:** class-balanced logistic regression with 10-fold stratified CV. Odds ratios and coefficient sign consistency across folds identify stable predictors of each type.
4. **Model comparison:** logistic regression, random forest and naive Bayes (Bernoulli and multinomial), tuned with `GridSearchCV` on stratified folds. The decision threshold was chosen from out-of-fold predictions on the training data, never on the test set.
5. **Class imbalance:** compared no resampling, random oversampling, SMOTE and random undersampling inside an `imblearn` pipeline, so resampling is applied only to training folds. The variant was selected by CV.
6. **Explainability:** permutation importance on the best model. Age at diagnosis and current insulin use carry most of the predictive signal.

## Results (held-out test set, n = 1,157)

| Model | Balanced accuracy | Macro-F1 | ROC-AUC |
|---|---|---|---|
| Logistic regression (class-balanced) | 0.653 | 0.543 | 0.709 |
| Random forest (class-weighted, threshold 0.54 from OOF) | 0.700 | 0.608 | 0.769 |
| Random forest + random undersampling (selected by CV) | **0.689** | 0.523 | 0.753 |

Balanced accuracy and macro-F1 are reported instead of plain accuracy, because a model that always predicted Type 2 would reach about 91% accuracy while never identifying Type 1.

Random undersampling and random oversampling were within 0.01 balanced accuracy of each other in cross-validation (fold-to-fold standard deviation about 0.03), so the choice between them is not decisive. The variant was still chosen by CV rather than by test score, which is why the final model is not the one with the best test result.

## Limitations

- Precision for the minority class (Type 1) stays low (0.17 to 0.24 across models), so the model is better at flagging *possible* Type 1 cases for follow-up than at confirming them.
- Some feature-screening steps (association analysis) were run on the full cleaned dataset before the train/test split, and the reduced-feature model was chosen using permutation importance on the test set. Its test score is therefore optimistic and is not reported above.
- BRFSS is self-reported survey data, so the results show associations, not causes.

## How to run

1. Download the **2024 BRFSS SAS Transport file** (`LLCP2024.XPT`) from the [CDC BRFSS annual data page](https://www.cdc.gov/brfss/annual_data/annual_2024.html) and place it in the repository root. It is not included here because of its size.
2. Install the dependencies and open the notebook:
   ```bash
   pip install -r requirements.txt
   jupyter notebook diabetes_type_classification.ipynb
   ```

`llcp2024_value_meanings_for_my_columns.xlsx` maps the codebook value meanings for the columns used, and is read by the notebook.

> The interactive Plotly radar charts (odds ratios by diabetes type) render when you run the notebook locally. GitHub does not display Plotly output.

## Tools

Python · pandas · NumPy · SciPy · scikit-learn · imbalanced-learn · Matplotlib · seaborn · Plotly
