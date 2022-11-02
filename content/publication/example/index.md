---
title: 'Understanding Heart Failure Patients EHR Clinical Features via SHAP Interpretation of Tree-Based Machine Learning Model Predictions'

# Authors
# If you created a profile for a user (e.g. the default `admin` user), write the username (folder name) here
# and it will be replaced with their full name and linked to their profile.
authors:
  - admin
  - Ruoyu Chen
  - Wei Wei
  - Mia Beilovsky
  - Xinghua Lu

# Author notes (optional)
author_notes:
  - 
  - 'Computer School, Beijing Information Science & Technology University, Beijing, China'

date: '2021-10-31T00:00:00Z'

# Schedule page publish date (NOT publication's date).
publishDate: '2022-02-21T00:00:00Z'

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ['1']

# Publication name and optional abbreviated publication name.
publication: In *2021 AMIA Annu Symp Proc*
publication_short: In *AMIA Annu Symp Proc*

abstract: Heart failure (HF) is a major cause of mortality. Accurately monitoring HF progress and adjusting therapies are critical for improving patient outcomes. An experienced cardiologist can make accurate HF stage diagnoses based on combination of symptoms, signs, and lab results from the electronic health records (EHR) of a patient, without directly measuring heart function. We examined whether machine learning models, more specifically the XGBoost model, can accurately predict patient stage based on EHR, and we further applied the SHapley Additive exPlanations (SHAP) framework to identify informative features and their interpretations. Our results indicate that based on structured data from EHR, our models could predict patients’ ejection fraction (EF) scores with moderate accuracy. SHAP analyses identified informative features and revealed potential clinical subtypes of HF. Our findings provide insights on how to design computing systems to accurately monitor disease progression of HF patients through continuously mining patients’ EHR data.

# Summary. An optional shortened abstract.


tags: []

# Display this page in the Featured widget?
featured: true

# Custom links (uncomment lines below)
# links:
# - name: Custom Link
#   url: http://example.org

url_pdf: ''
url_code: ''
# url_dataset: ''
# url_poster: ''
# url_project: ''
url_slides: ''
url_source: ''
url_video: ''

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
image:
  focal_point: ''
  preview_only: false

# Associated Projects (optional).
#   Associate this publication with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `internal-project` references `content/project/internal-project/index.md`.
#   Otherwise, set `projects: []`.
projects:
  - example

# Slides (optional).
#   Associate this publication with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides: "example"` references `content/slides/example/index.md`.
#   Otherwise, set `slides: ""`.
slides: example
---

{{% callout note %}}
Click the _Cite_ button above to demo the feature to enable visitors to import publication metadata into their reference management software.
{{% /callout %}}

{{% callout note %}}
Create your slides in Markdown - click the _Slides_ button to check out the example.
{{% /callout %}}

Supplementary notes can be added here, including [code, math, and images](https://wowchemy.com/docs/writing-markdown-latex/).
