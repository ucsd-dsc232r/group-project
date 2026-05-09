# Milestone 3: Preprocessing & First Distributed Model

## Introduction

In this milestone you will continue working on your main branch, focusing on completing preprocessing using Spark and building your first distributed model. You will evaluate your model and analyze where it fits on the underfitting/overfitting spectrum.

*Remember: All processing must be done on SDSC Expanse using Spark or Ray. Your model training should demonstrate meaningful use of distributed computing resources.*

---

## Instructions

### 1. Complete Preprocessing using Spark (8 points)

Finish major preprocessing using **Spark DataFrame operations** or **Spark MLlib transformers**:

- **Scaling/Transforming:** Use `StandardScaler`, `MinMaxScaler`, or `Normalizer` from `pyspark.ml.feature`
- **Imputing:** Use `Imputer` for handling missing values
- **Encoding:** Use `StringIndexer`, `OneHotEncoder`, or `VectorAssembler`
- **Feature Engineering:** Create new features using Spark SQL functions, `PolynomialExpansion`, or custom transformations

Your preprocessing pipeline should be reproducible and documented in your notebook.

### 2. Train Your First Distributed Model (8 points)

Train your first model using one of the following **distributed implementations**:

| Model | Implementation | Course Reference |
|---|---|---|
| Decision Trees / Random Forests | `pyspark.ml.classification.DecisionTreeClassifier` / `pyspark.ml.classification.RandomForestClassifier` | Classification lectures |
| Gradient Boosted Trees | `pyspark.ml.classification.GBTClassifier` | Boosting lectures |
| XGBoost (Distributed) | `xgboost.spark.SparkXGBClassifier` or Ray Train `XGBoostTrainer` | Class 15-16 |
| Ray Train Models | `ray.train.xgboost.XGBoostTrainer` / `ray.train.lightgbm.LightGBMTrainer` | Class 16 |

**Requirements:**

- Model must run on **SDSC Expanse** (not locally)
- Training must use **multiple executors/workers** (verify via Spark UI or Ray Dashboard)
- Evaluate your model: compare **training vs. test error**
- For supervised learning: include example ground truth and predictions for train, validation, and test sets

### 3. Fitting Analysis (4 points)

Answer the following questions:

- Where does your model fit in the fitting graph (underfitting vs. overfitting)?
- Build at least one model with **different hyperparameters** and compare results
- Which model performs best and why?
- What are the next models you are thinking of for Milestone 4 and why?

### 4. Conclusion Section (5 points)

Write a conclusion for your first model:

- What is the conclusion of your 1st model?
- What can be done to possibly improve it?
- How did distributed computing help with this task?

### 5. Speedup Analysis (5 points) — NEW

Measure and report the speedup achieved by your distributed implementation:

1. **Baseline Measurement:** Run a representative operation (e.g., your preprocessing pipeline or model training) with 1 executor. Record wall-clock time.
2. **Scaled Measurement:** Run the same operation with your full executor configuration. Record wall-clock time.
3. **Calculate Metrics:**
   - Speedup = T₁ / Tₙ (where n = number of executors)
   - Efficiency = Speedup / n
4. **Analyze:** Compare your measured speedup to the theoretical maximum (Amdahl's Law). Estimate what fraction of your code is parallelizable.

**Include a table in your README.md:**

| Executors | Time (sec) | Speedup | Efficiency |
|---|---|---|---|
| 1 | X | 1.00x | 100% |
| 7 | Y | X/Y | (X/Y)/7 |

See the [Speedup Measurement Guide](https://github.com/ucsd-dsc232r/group-project/blob/main/Class16/06_speedup_measurement.md) for detailed instructions.

### 6. Update README.md

Update your README.md to include your new work. Make sure to upload all code and notebooks. Provide links in your README.md.

---

## Important Notes

> **Model Restrictions for DSC 232R:**
>
> - You must use a **distributed implementation** (Spark MLlib, Spark XGBoost, or Ray Train)
> - Simple linear regression is **not acceptable** unless combined with substantial feature engineering using Spark
> - Models that only run on the driver (e.g., scikit-learn on collected data) are **not acceptable**
> - Your training should demonstrate meaningful parallelization across multiple executors

---

## Submission and Grading

**Important:** Prior to submitting, create a branch of your main repo named `Milestone3`

**Any git commits past the deadline will not be considered!**

Submit your GitHub URL for your Milestone3 branch.

**Points: 30**
