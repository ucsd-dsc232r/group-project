# Milestone 4: Second Model, Final Report & Submission

## Introduction

In this final milestone you will build your second model using dimensionality reduction techniques, complete your final report, and submit your finished project. You will update all your code and README.md in your main branch for the final submission.

*Your GitHub README is the primary deliverable—it should tell a complete, professional story of your big data project that you can share with future employers.*

---

## Instructions for Second Model (15 points)

### 1. Train Your Second Model using Dimensionality Reduction (3 points)

Your second model must include **unsupervised learning** for dimensionality reduction, followed by additional analysis:

| Technique | Implementation | Course Reference |
|---|---|---|
| PCA (Principal Component Analysis) | `pyspark.ml.feature.PCA` or manual implementation with Spark | Class 8-10 |
| SVD (Singular Value Decomposition) | `pyspark.mllib.linalg.distributed.RowMatrix.computeSVD` | Class 8 |
| Distributed PCA with Ray | Ray Data preprocessing or custom implementation | Class 16 |

**Follow dimensionality reduction with one of:**

- **Clustering:** K-Means, GMM, or other clustering on reduced features
- **Visualization & Interpretation:** Eigenvalue analysis, explained variance plots, component interpretation
- **Supervised Model:** Train a model on the reduced-dimension features

### 2. Evaluate Your Model (3 points)

- Compare training vs. test performance
- Analyze explained variance (for PCA/SVD)
- Evaluate clustering quality (silhouette score, etc.) if applicable

### 3. Fitting Analysis (3 points)

Answer the following:

- Where does your model fit in the fitting graph?
- What are potential future improvements or next models?
- How does dimensionality reduction affect your results compared to the full feature set?

### 4. Update README.md (1 point)

Include all new work. Upload all code and notebooks. Provide links in your README.md.

### 5. Conclusion Section (3 points)

What is the conclusion of your 2nd model? What can be done to improve it?

*Note: The conclusion should be its own independent section. Methods will have models 1 and 2, Conclusion will have results and discussion for both.*

### 6. Predictions Analysis (2 points)

Provide predictions showing correct classifications, false positives (FP), and false negatives (FN) from your test dataset.

---

## Instructions for Final README

Your final README.md should be a **complete, professional document** that tells the story of your project:

1. A complete introduction explaining your project and its importance
2. All prior milestone submissions integrated into a cohesive narrative
3. All code uploaded as Jupyter notebooks that can be easily followed
4. A written report with all required sections
5. Your final model included in every section (Methods, Results, Discussion)
6. **Your GitHub repo must be made public** by the morning of the next day after the submission deadline

---

## Written Report Sections

### Introduction to Your Project (3 points)

Why was this project chosen? Why is it interesting? Discuss the general/broader impact of having a good predictive model. Why is this important?

For DSC 232R, also address: Why did this problem require big data and distributed computing? What would be impossible or impractical without Spark/Ray?

### Figures (3 points)

Your report should include relevant figures to help narrate your story, including legends (similar to a scientific paper). For reference, search machine learning and your model type in Google Scholar for examples.

Include visualizations of: data exploration, PCA/SVD results (explained variance, component plots), model performance, and predictions.

### Methods Section (5 points)

This section includes exploration results, preprocessing steps, and models chosen in the order they were executed. Describe the parameters chosen. Create sub-sections for each step:

- **Data Exploration**
- **Preprocessing** (using Spark)
- **Model 1** (your first distributed model)
- **Model 2** (PCA/SVD + clustering or supervised)

Include code blocks using markdown: `` ```python ... ``` ``

*Note: A methods section does not include "why"—the reasoning goes in the Discussion section. This is just a summary of your methods.*

### Results Section (5 points)

Present the results from your methods. Include figures about your results. No exploration or interpretation here—this is mainly a summary of your results. Sub-sections should mirror your Methods section.

Include: accuracy metrics, confusion matrices, explained variance plots, clustering visualizations, etc.

### Discussion Section (3 points)

This is where you discuss the "why" and your interpretation—your thought process from beginning to end. Discuss how believable your results are at each step. Discuss any shortcomings.

It's okay to criticize your own work—this shows intellectual merit and scientific thinking. In science we rarely find perfect solutions. If your results seem too good, scrutinize them carefully!

### Conclusion (3 points)

This is where you share your opinions and possible future directions. What would you have done differently? Close with final thoughts about:

- What you learned about big data processing
- How distributed computing changed your approach
- What you would explore with more time/resources

### Statement of Collaboration (3 points)

This is a statement of contribution by each member. This will be taken into consideration when making the final grade for each member in the group.

Did you work as a team? Was there a team leader? Project manager? Coder? Writer? Please be truthful as this will determine individual grades in participation.

There is no job that is better than the other:

- If you did no code but did the entire write-up and gave feedback during the steps and collaborated—full credit.
- If you only coded but gave feedback on the write-up—full credit.
- If you managed everyone, deadlines, meetings, and communicated with teaching staff—full credit.

**Every role is important as long as you collaborated and were integral to the completion of the project. If a person did nothing, they risk getting a zero.** Just like in any job, if you did nothing, you risk getting fired. Teamwork is one of the most important qualities in industry and academia!

**Format:** Start with `Name: Title: Contribution`. If someone contributed nothing, write: "Did not participate in the project."

---

## Voting (Separate Assignment - 5 points)

Voting will be released as a **separate assignment** after the MS4 deadline. You will have 2 days to decide on your top 3 favorite projects.

- You can vote for yourself, but you cannot vote for any one group more than once
- You must submit 3 votes to get credit
- Make sure your repository is made public so other students can view and vote for your project
- The top 3 projects (voted by students) will receive extra credit

---

## Submission and Grading

**Important:** Any git commits past the deadline will not be considered!

Submit your GitHub URL for your **main branch**.

**Points: 40** (Voting is 5 additional points as a separate assignment)
