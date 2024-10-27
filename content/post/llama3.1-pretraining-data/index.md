---
draft: false
title: 'Data Processing for Pre-training Llama 3.1'
date: 2024-09-20T13:40:59-07:00
lastmod: 2024-09-20T13:40:59-07:00
tags:
categories:
authors:
    - admin
---


_The performance of large language models (LLMs) is highly dependent on the quality of the data they are trained on. However, the data curation and cleaning process is underrated in today's discussions of AI._
- - -
This article illustrates how Llama 3.1 developers collect and clean data to be used for pre-training Llama 3.1, which involved a multi-faceted approach to data curation, cleaning, and filtering. The main content is taken from the technical report of Llama 3.1 that could be found [here](https://ai.meta.com/research/publications/the-llama-3-herd-of-models/).

# Content and Domain Distribution
Llama 3.1 was trained on a massive dataset of 15 trillion multilingual tokens, a substantial increase compared to the 1.8 trillion tokens used to train Llama 2.

The final data mix for Llama 3.1 consists of approximately 50% general knowledge, 25% mathematical and reasoning content, 17% code, and 8% multilingual data.

The developers designed filters to remove websites likely to contain adult content, unsafe content and personally identifiable information (PII), though no specifics were provided for the filters. Additionally, they sought to reduce the representation of art and entertainment domains, though, I assume, these data would consist of the majority of the internet tokens.

# Data Cleaning and Filtering

The cleaning and filtering process is crucial for ensuring that the training data is of high quality. This in involves a multi-step procedure including text extraction, de-duplication and heuristic filtering.

## Text Extraction and Formatting

The initial steps involved extracting text from HTML and potentially removing Markdown formatting elements. They built parsers optimized for boilerplate removal and content recall. They maintaine the image alt attribute text since mathematical content is often represented as pre-rendered images where the math is also provided in the alt attribute. Interestingly enough, they removed all Markdown markers from the web data because they found Markdown format was harmful to the performance of the model. It is a mystery.

 
## De-duplication

De-duplication is applied at URL, document and line levels. This process are not only applied across the entire dataset but also within each sub dataset that belongs to individual language.

### URL-level
They first removed duplicate documents based on their URLs and keep the most recent pages corresponding to each URL.

### MiHash 
On the document level, they employed MinHash, a technique for estimating the similarity between datasets, to further remove duplicates. MinHash is a powerful technique for efficiently identifying near-duplicate documents from large dataset. It works on statistical properties of the text rather than semantic meaning by creating a compact fingerprint representation of each document. The technique has a lot of benefits. For example, it detects documents that are not exactly identical but have high textual similarity. Users can also tune the similarity threshold in order to balance precision and recall. Though specifics of this technique is not provided in the technical report, a detailed tutorial on MinHash can be found [here](https://mccormickml.com/2015/06/12/minhash-tutorial-with-python-code/).

### Aggressive line removal
On line level, they removed lines that appeared 6 times in each bucket of 30M documents. This way they are able to remove boilerplate content from the website such as navigation menus, cookie warnings, etc. Note that this method has a high false positive rate that it removes some high-quality data also.

  

## Heuristics-based Filtering

Various heuristics were employed to identify and filter out low-quality data:

### N-gram Convergence Ratio
While the exact details of its application are not elaborated upon, this metric was likely used to assess the linguistic diversity of the dataset and potentially filter out repetitive or low-quality segments.

### Dirty Word Filtering 
Removing text containing offensive or inappropriate language. It is quite straightforward.

### Token Distribution KL Divergence
Utilizing the Kullback-Leibler divergence to measure the difference between the token distribution of individual documents and the overall corpus distribution. Documents with excessively divergent token distributions, suggesting outlier or low-quality content, were filtered out.

### Model-based Filtering
Llama 2, a predecessor to Llama 3.1, was utilized to develop a quality classification model. This model was trained to identify high-quality text based on human-defined criteria, further enhancing the quality of the training data.

# Code and Reasoning Data
Special attention was given to the processing of code and reasoning data, which was deemed beneficial for improving the model's reasoning abilities:

## HTML Extraction
A tailored HTML extraction method was developed specifically for code data, as the token distribution of code and math is substantially different from that of natural language. Specifically, they trained code classifiers and reasoning classifiers. Both classifiers are DistilRoberta model, which is a distilled version of RoBERTa-base model. It is intended to be fine-tuned on tasks that use the whole sentence to make decisions such as classification problems.

## Annealing
To further enhance performance on reasoning tasks, an annealing technique was employed. The data used for annealing is a set of 40M tokens of very high quality (probably code and mathematical?) upsampled from data sources. They increased the learning rate and linearly annealed the learning rate to 0, maintaining a context length of 128K tokens. Finally, they computed the average of model checkpoints during annealing to produce the final pre-trained model.