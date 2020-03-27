# Biomedical Entity Recognition

## Overview

[Entity extraction](https://en.wikipedia.org/wiki/Named-entity_recognition) is a subtask of information extraction (also known as Named-entity recognition (NER), entity chunking and entity identification). Biomedical named entity recognition is a critical step for complex biomedical NLP tasks such as: 
* Extraction of diseases, symptoms from electronic medical or health records.
* Drug discovery
* Understanding the interactions between different entity types such as drug-drug interaction, drug-disease relationship and gene-protein relationship.

This real-world scenario focuses on how a large amount of unstructured unlabeled data corpus such as PubMed article abstracts can be analyzed to train a domain-specific word embedding model. Then the output embeddings are considered as automatically generated features to train a neural entity extraction model using Keras with TensorFlow deep learning framework as backend and a small amoht of labeled data.

## Description

The aim of this real-world scenario is to highlight how to use Azure Machine Learning cloud-based environment to solve a complicated NLP task such as entity extraction from unstructured text. Here are the key points addressed:

1. How to train a neural word embeddings model on a text corpus of about 18 million PubMed abstracts using [Spark Word2Vec implementation](https://spark.apache.org/docs/latest/mllib-feature-extraction.html#word2vec).
2. How to build a deep Long Short-Term Memory (LSTM) recurrent neural network model for entity extraction.
2. Demonstrate that domain-specific word embeddings models can outperform generic word embeddings models in the entity recognition task. 
3. Demonstrate how to train and operationalize deep learning models using Azure Machine Learning.

