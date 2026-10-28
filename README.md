# Predicting Traffic Accident Severity with Machine Learning

**Language:** English | [Srpski](README.sr.md)

- **Thesis:** *Application of Machine Learning Methods for Predicting the Severity of Traffic Accidents* (original title in Serbian: *Primjena metoda mašinskog učenja za predviđanje ozbiljnosti saobraćajnih nezgoda*)
- **Author:** Matea Masleša, Information Technology School (ITS), Belgrade, 2026
- **Supervisor:** Dr Đorđe Petrović

## Goal

Based on the circumstances of a traffic accident (place, time, type and description of the accident), predict its **outcome** in three classes: material damage, injured and fatal. The thesis compares four models and focuses on severe class imbalance.

## Data

- **Source:** Ministry of Interior of the Republic of Serbia, *Data on traffic accidents by police departments and municipalities for 2025*, open data portal ([link](https://data.gov.rs/sr/datasets/podatsi-o-saobratshajnim-nezgodama-po-politsijskim-upravama-i-opshtinama/)).
- **Size:** 33,415 accidents from 2025, 9 original columns.
- **The data are not in the repository.** Instructions are in `data/README.md`.
- **Class distribution:** material damage about 59%, injured about 39%, **fatal only 1.4% (463 accidents)**.

## Methodology

1. **Exploratory analysis:** distribution by police department and municipality, accident type, month, hour and weekday (figures in `results/`).
2. **Features (11):** categorical: police department, municipality, accident type, accident description, weekday. Numerical: longitude and latitude, plus month and hour encoded cyclically (sin/cos).
3. **Data split:** 80/20, stratified (`random_state=42`): 26,732 for training, 6,683 for testing (92 fatal accidents). The split is done **before** encoding and resampling so that the test set stays untouched.
4. **Class imbalance:** SMOTENC (training set only; fatal class from 371 to 15,855 samples) together with class weights in the models.
5. **Models:** Logistic Regression, Decision Tree, Random Forest and CatBoost.
6. **Evaluation:** accuracy, weighted and macro F1, plus precision, recall and F1 for the fatal class. Additionally, 5-fold stratified cross-validation for Random Forest (SMOTENC inside the pipeline).
7. **Leakage check:** the accident description was checked and does not directly contain the word for "fatal" for the fatal class (0.00%).

## Results (test set, 6,683 accidents)

| Model | Accuracy | Weighted F1 | Macro F1 | Precision (fatal) | Recall (fatal) |
|---|---|---|---|---|---|
| Logistic Regression | 0.64 | 0.69 | 0.48 | 0.04 | **0.42** |
| Decision Tree | 0.67 | 0.69 | 0.48 | 0.03 | 0.20 |
| Random Forest | 0.73 | 0.73 | 0.51 | 0.06 | 0.11 |
| **CatBoost** | **0.74** | **0.74** | **0.53** | 0.10 | 0.13 |

The table is in `results/rezultati_modela.csv`; confusion matrices and the model comparison are in `results/` (`14`–`20`).

**Key findings**
- **CatBoost** has the best overall accuracy (0.74) and the best macro F1 (0.53).
- **Logistic Regression** finds the most fatal accidents (recall 0.42) but with very low precision (0.04), i.e. most of its "alerts" are false alarms.
- Random Forest cross-validation: weighted F1 0.734 ± 0.003, recall for fatal accidents 0.076 ± 0.011. Results are stable, but recall for the fatal class stays low.
- The choice of model depends on the goal: CatBoost for overall accuracy, Logistic Regression for catching rare severe accidents, at the cost of false alarms.

## Limitations

- **The fatal class is still predicted poorly.** Precision is 0.03–0.10 and recall 0.11–0.42, so the model is not suitable for stand-alone warnings.
- The features describe the accident itself (including its description), so the model **estimates severity from the circumstances of the accident**; it does not predict it in advance.
- Random Forest overfits the training data (training accuracy 0.92, test 0.73).
- Only one year of data and one random stratified split. The test set has just 92 fatal accidents, so estimates for that class are uncertain.
- Hyperparameters were not systematically tuned.

## Structure and usage

```
data/        instructions for downloading the data
notebooks/   Diplomski_kod.ipynb (code with saved outputs)
results/     figures and results table
```

```bash
pip install -r requirements.txt
jupyter notebook notebooks/Diplomski_kod.ipynb
```

Download the data into the `data/` folder before running (see `data/README.md`). The notebook comments are in Serbian.
