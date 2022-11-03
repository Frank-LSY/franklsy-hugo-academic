---
title: SOFA Score Analysis of Cardiac Arrest Patients
summary: Pipeline for Data Cleaning and Aggregation & Traditional Statistical Analysis
tags:
  - Data Mining
date: '2019-12-26T00:00:00Z'

# Optional external URL for project (replaces project detail page).
external_link: ''

image:
  caption: https://rebelem.com/sepsis-3-0/sofa-score/
  focal_point: Smart

url_code: ''
url_pdf: ''
url_slides: ''
url_video: ''

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides: example
---

[What is SOFA score? ](https://en.wikipedia.org/wiki/SOFA_score)

- I created a program to calculate daily SOFA score sub-scales from raw daily assessments for a cohort of cardiac arrest patients admitted to UPMC Presbyterian Hospital and seen by the Post Cardiac Arrest Service from 2010-2019.
- We did a descriptive analysis of basic uni-variate Pearson's correlation between SOFA sub-scores and survival, mRS, and CPC.
- I applied the cluster method to study the correlation between sub-SOFA scores and patients' conditions(death, discharge, etc.)
