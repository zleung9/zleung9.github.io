---
title: Uncertainty-aware predictions of molecular X-ray absorption spectra using neural
  network ensembles
authors:
- Mikhail Segal, Fanchen Meng, Zhu Liang, Mark S. Hybertsen, Xiaohui Qu, Eli Stavitski,
  Shinjae Yoo, Deyu Lu, Matthew R. Carbone Animesh Ghose
date: '2023-01-01'
doi: ''
publishDate: '2024-10-26T23:42:20.020072Z'
publication_types:
- article-journal
publication: '*Physical Review Research*'

abstract: As machine learning (ML) methods continue to be applied to a broad scope of problems in the physical sciences, uncertainty quantification is becoming correspondingly more important for their robust application. Uncertainty-aware machine learning methods have been used in select applications, but largely for scalar properties. In this work, we showcase an exemplary study in which neural network ensembles are used to predict the x-ray absorption spectra of small molecules, as well as their pointwise uncertainty, from local atomic environments. The performance of the resulting surrogate clearly demonstrates quantitative correlation between errors relative to ground truth and the predicted uncertainty estimates. Significantly, the model provides an upper bound on the expected error. Specifically, an important quality of this uncertainty-aware model is that it can indicate when the model is predicting on out-of-sample data. This allows for its integration with large-scale sampling of structures together with active learning or other techniques for structure refinement. Additionally, our models can be generalized to larger molecules than those used for training, and also successfully track uncertainty due to random distortions in test molecules. While we demonstrate this workflow on a specific example, ensemble learning is completely general. We believe it could have significant impact on ML-enabled forward modeling of a broad array of molecular and materials properties.
url_pdf: "https://journals.aps.org/prresearch/abstract/10.1103/PhysRevResearch.5.013180"
url_code: "https://github.com/AI-multimodal/XAS-NNE"
---