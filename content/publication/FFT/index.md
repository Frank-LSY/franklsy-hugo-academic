---
title: 'Evaluating FFT-based algorithms for strided convolutions on ARMv8 architectures.'

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

date: '2021-12-21T00:00:00Z'

# Schedule page publish date (NOT publication's date).
publishDate: '2022-03-25T00:00:00Z'

# Publication type.
# Legend: 0 = Uncategorized; 1 = Conference paper; 2 = Journal article;
# 3 = Preprint / Working Paper; 4 = Report; 5 = Book; 6 = Book section;
# 7 = Thesis; 8 = Patent
publication_types: ['2']

# Publication name and optional abbreviated publication name.
publication: In *ACM SIGMETRICS Performance Evaluation Review*
publication_short: In *ACM SIGMETRICS Performance Evaluation Review*

abstract: Convolutional Neural Networks (CNNs) have been widely adopted in all kinds of artificial intelligence applications. Most of the computational overhead of CNNs is mainly spent on convolutions. An effective approach to reducing the overhead is FFT-based fast algorithms for convolutions. However, current FFT-based fast implementations cannot be directly applied to strided convolutions with a stride size of greater than 1. In this paper, we first introduce rearrangement- and sampling-based methods for applying FFT-based fast algorithms on strided convolutions. Then, the highly optimized parallel implementations of the two methods on ARMv8-based many-core CPU are presented. Lastly, we benchmark the implementations against the two GEMM-based implementations on this ARMv8 CPU. Our experimental results with convolutions of different kernel, and feature maps, and batch sizes show that the rearrangementbased method generally exceed the sampling-based one under the same optimizations in most cases, and these two methods can get much better performance than GEMMbased ones when the kernel, feature maps and batch sizes are large. The experimental results with the convolutional layers in popular CNNs further demonstrate the conclusion above.

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

