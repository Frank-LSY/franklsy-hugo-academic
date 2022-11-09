---
title: Heart Failure Patients EHR Data Mining
summary: An Integrated EHR Data Mining project
tags:
  - Data Mining
date: '2021-03-26T00:00:00Z'

# Optional external URL for project (replaces project detail page).
external_link: ''

url_code: ''
url_pdf: ''
url_slides: ''
url_video: ''

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
---
### There's a detailed working process of data pre-processing work created by [Dr. Ruoyu Chen](https://github.com/ruoyu-chen). You can refer to it [here]({{< ref "reference/pre-processing.md" >}}).

- As you can see from Dr. Chen's work, after getting the cleaned data, Dr. Chen did further exploration with structured EHR data, which applied deep learning model to form the data and tried to predict 30-day readmission of heart failure patients.

#### Considering the work I did, for majority of the time, I put attention on dealing with unstructured medical text data:
  - I used word vector models (CBOW, word2vec) in combination with topic models (LDA, tfidf) to obtain a vector representation of unstructured medical records. I also tried to apply mainstream NLP models (BERT, transformer, etc.) to embedding the documents. Those embeddings were then feed into Dr. Chen's deep learning model for further prediciton work.

#### I also did data analysis work on structred EHR data:
  - We analyzed structured EHR data of heart failure patients from UPMC, 2014-2019 to explore the feasibility of interpreting clinical characteristics data using interpretable machine learning methods.
  - Based on XGBoost Regression, a model was developed to predict EF scores of heart failure patients using structured EHR data; the practical implications of the model and prediction results were explained using the SHAP(a model-independent interpretable machine learning method); supervised clustering was performed using the generated SHAP values, and dimensionality reduction visualization was performed using t-SNE; analyzed the correlation between EHR features in an attempt to delineate different heart failure patient subgroups.

