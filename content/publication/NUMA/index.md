---
title: 'NUMA-aware FFT-based Convolution on ARMv8 Many-core CPUs'

# Authors
# If you created a profile for a user (e.g. the default `admin` user), write the username (folder name) here
# and it will be replaced with their full name and linked to their profile.
authors:
  - Xiandong Huang
  - Qinglin Wang
  - admin
  - Ruochen Hao
  - Songzhu Mei
  - Jie Liu

# Author notes (optional)

date: '2021-10-03T00:00:00Z'

# Schedule page publish date (NOT publication's date).
publishDate: '2021-12-22T00:00:00Z'

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ['1']

# Publication name and optional abbreviated publication name.
publication: In *ISPA/BDCloud/SocialCom/SustainCom*
publication_short: In *ISPA/BDCloud/SocialCom/SustainCom*

abstract: Convolutional Neural Networks (CNNs), one of the most representative algorithms of deep learning, are widely used in various artificial intelligence applications. Convolution operations often take most of the computational overhead of CNNs. The FFT-based algorithm can improve the efficiency of convolution by reducing its algorithm complexity, there are a lot of works about the high-performance implementation of FFT-based convolution on many-core CPUs. However, there is no optimization for the non-uniform memory access (NUMA) characteristics in many-core CPUs. In this paper, we present a NUMA-aware FFT-based convolution implementation on ARMv8 many-core CPUs with NUMA architectures. The implementation can reduce a number of remote memory access through the data reordering of FFT transformations and the three-level parallelization of the complex matrix multiplication. The experiment results on a ARMv8 many-core CPU with NUMA architectures demonstrate that our NUMA-aware implementation has much better performance than the state-of-the-art work in most cases.

# Summary. An optional shortened abstract.


tags: []

# Display this page in the Featured widget?
featured: false

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

---

{{% callout note %}}
Click the _Cite_ button above to import publication metadata into their reference management software.
{{% /callout %}}

