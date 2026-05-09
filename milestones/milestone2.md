# Milestone 2: GitHub Setup, SDSC Expanse, & Data Exploration

## Introduction

For this second part of your group project, you will be creating a GitHub Repository for your work, setting up your SDSC Expanse environment, and exploring your dataset using Spark.

*This project simulates a real-world big data workflow. The skills you develop here—configuring Spark sessions, optimizing memory allocation, working with distributed datasets—translate directly to cloud platforms like AWS EMR, Google Dataproc, and Databricks.*

---

## SDSC Expanse Setup

All work for this project must be done on **SDSC Expanse**, not on your local machine. Follow the setup instructions carefully:

- **Login Instructions:** [portal.expanse.sdsc.edu](https://portal.expanse.sdsc.edu/) – Use your "ucsd.edu" credentials
- **First-Time Setup:** See the [Group Project README](https://github.com/ucsd-dsc232r/group-project/blob/main/README.md) for detailed instructions on creating symbolic links and configuring your environment
- **Spark Configuration:** See the [Spark HPC Best Practices Guide](https://github.com/ucsd-dsc232r/group-project/blob/main/SPARK_HPC_BEST_PRACTICES.md) for memory and executor configuration
- **Troubleshooting:** See the [FAQs](https://github.com/ucsd-dsc232r/group-project/blob/main/FAQs.md) for common issues

---

## Instructions

### 1. GitHub Repository Setup (2 points)

a. Create a GitHub ID (if you don't have one)
b. Create a GitHub Repository (Public or Private—it will need to be Public for final submission) and add your group members as collaborators
c. Provide a link to your dataset in your README.md

### 2. SDSC Expanse Environment Setup (2 points)

a. Document your SDSC Expanse setup in your README.md
b. Include your **SparkSession configuration** with justification for your memory/executor settings
c. Use the formula: `Executor instances = Total Cores - 1` and `Executor memory = (Total Memory - Driver Memory) / Executor Instances`
d. **Include a screenshot of your Spark UI** showing multiple executors active during data loading

### 3. Data Exploration using Spark (4 points)

All data exploration must be done using **Spark DataFrames**, not Pandas. Use operations like:

- `df.count()`, `df.describe().show()`, `df.printSchema()`
- `df.groupBy().agg()` for aggregations
- `df.select().distinct().count()` for unique values

Answer the following:

a. How many observations does your dataset have?
b. Describe all columns in your dataset: their scales and data distributions. Describe categorical and continuous variables. Describe your target column.
c. Do you have missing and duplicate values in your dataset?
d. For image data: describe number of classes, image sizes, uniformity, cropping/normalization needs.

### 4. Data Plots (4 points)

a. Create visualizations using Spark aggregations + matplotlib/plotly (sample data for plotting if needed)
b. Plot your data with various chart types: bar charts, histograms, scatter plots, etc.
c. Clearly explain each plot and what insights it provides
d. For image data: plot example classes

### 5. Preprocessing Plan (3 points)

Describe how you will preprocess your data. **Only explain—do not perform preprocessing** (that is for MS3).

- How will you handle missing values?
- How will you handle data imbalance (if applicable)?
- What transformations will you apply (scaling, encoding, feature engineering)?
- What Spark operations will you use for preprocessing?

Link your Jupyter notebook to your README.md. All code and notebooks must be uploaded to your repo.

---

## SparkSession Configuration Reference

Include a properly configured SparkSession in your notebook. Example configuration:

```python
from pyspark.sql import SparkSession

# Example: 8 cores, 128GB total memory
spark = SparkSession.builder \
    .config("spark.driver.memory", "2g") \
    .config("spark.executor.memory", "18g") \
    .config("spark.executor.instances", 7) \
    .getOrCreate()
```

**Quick Reference:**

| Cores | Total Memory | Driver | Executors | Executor Memory |
|---|---|---|---|---|
| 8 | 16GB | 2GB | 7 | 2GB |
| 8 | 128GB | 2GB | 7 | 18GB |
| 16 | 64GB | 2GB | 15 | 4GB |

---

## Submission and Grading

You will still be able to edit your GitHub repo for the data exploration part for your final submission, but we will grade this part of your submission as if it were finalized for Milestone 2 given the deadline. Only commits before the deadline will be used to evaluate your submission.

**Important:** Prior to submitting, create a branch of your main repo named `Milestone2`

**Any git commits past the deadline will not be considered!**

Submit your GitHub URL for your Milestone2 branch.

**Points: 15**
