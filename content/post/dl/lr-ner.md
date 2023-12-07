---
title: "Literature Review on NER"
date: "2021-11-17"
description: ""
# tags: []
categories: [
    "Deep Learning",
]
katex: true
---



Named entity recognition (NER) is a classical task in NLP, which aims to identify meaningful chunks of text and classify them into appropriate entity types, such as PERSON or LOCATION. For instance, "Tom" is classified as "PERSON" while "England" is labelled as "LOCATION" in the sentence "Tom comes from England". NER serves as the foundation for a range of downstream tasks such as information extraction. In practical applications, you may notice that some websites will autofill the application form from your resume. In this series of posts, we will explain NER in detail, from models, datasets to domain adaptation. First, let's do a brief literature review on NER.


## NER Models



### Traditional Method

The earliest NER methods are rule-based and dictionary-based methods[1].

#### Rule-based 

How

- lexical patterns + domain knowledge + regular expressions

Pros & cons

- simple

- manually creating rule is time-consuming
- rules are language or domain dependent



#### Dictionary-based

How

- look up a terminological dictionary

Pros & Cons

- focus on the most important terms rather than all words in the field, e.g. biological field

- only named entities occurred in dictionaries can be identified
- dictionaries need to be updated frequently



#### Statistical-based

How

-  learn a statistical model composed of a set of rules derived from the annotated corpus

popular methods

- HMM
- CRF
- SVM



### Neural Network

The contemporary NER models are based on neural networks and various combinations of pre-trained word embedding and character embedding [2].

Pros & Cons

- do not require extensive feature engineering or domain-specific resources

- a large amount of training data



Based on word representation, neural architectures for NER can be classified into three types[3]

- the word-based architecture

  - word embedding (window approach) -> CNN_CRF
  - variants of LSTM_CRF -> BiLSTM_CRF

- the character-based architecture 

  - character embedding (CNN)  -> LSTM

- the hybrid architecture combining both word-level and character-level representation

  - character (CNN) and word embedding -> BiLSTM_Softmax
  - character (CNN) and word embedding -> BiLSTM_CRF
  - character (BiLSTM) and word embedding (skip-N-gram) -> BiLSTM_CRF
  - character (GRU) and word embedding (GRU) -> BiLSTM_CRF

  

## Domain Adaptation

Transfer learning in NLP[5] is broadly classified into four subcategories

- domain adaptation (our study area)
  - the source and target domain share the same language but have different word distributions
- cross-lingual learning
- multi-task learning
- sequential transfer learning. 



### Feature-based

mapping two domains into a low-dimensional space



### Parameter-based

- Fine-Tuning
- Multi-Task Learning



## Multi-source Domain Adaptation



The standard domain adaptation setting covers a single source domain and a single target domain with few or no labelled data. Extensions include multiple source domains and a single target domain or a single source domain and multiple target domains.

- pooling
  - pool all training data from all source domains and train a single model for the pooled data, ignoring the domain discrepancy.
- Ensemble each model trained on each source domain
- MULT with shared CRF layer
- MUTL with private CRF layer



## NER Datasets

- [MUC]( https://www-nlpir.nist.gov/related_projects/muc/)
- [CoNLL-2002]( https://www.clips.uantwerpen.be/conll2002/ner/) and [CoNLL-2003]( https://www.clips.uantwerpen.be/conll2003/ner/)
  - a collection of newswire articles covering 4 languages: Spanish, Dutch, English, and German
  - PER, LOC, ORG, and MISC
- Twitter
  - collected from multiple English-speaking countries and social user from 2009 to 2019
  - the same entity types as CoNLL-2003
- OntoNotes
  - a large-scale multilingual annotated corpus with multi-layer linguis- tic information like predicate structure and word sense.
  - 3 languages (English, Arabic, and Chinese) and many genres:broadcast conversation (BC), broadcast news (BN), magazine (MZ), newswire(NW), telephone conversation (TC), and web data (WB)

Most NER datasets only contain flat entities, but there are particular datasets that contain nested entities. 

- [ACE]( https://catalog.ldc.upenn.edu/LDC2006T06)
  - a multilingual corpus composed of data selected from various sources, including broadcast and newswire
- GENIA[4]
  - created for biology dut to the complicated naming conventions in biology, there are plenty of nested disease names or cell names



## Summary Table

| Related work                                                 | NER Model                                  | Results                                                      |
| ------------------------------------------------------------ | ------------------------------------------ | ------------------------------------------------------------ |
| Rau (1991)[@rauExtractingCompanyNames1991]                   | Rule-based method                          | The author designed the first NER system based on hand-designed rules. |
| Zhou **et al**. (2006) [@zhouMaxMatcherBiologicalConcept2006] | Dictionary-based method                    | They proposed a new dictionary-based method that focused on important biological terms and achieved the best performance on GENIA corpus. |
| Bikel **et al**. (1999) [@bikelAlgorithmThatLearns1999]      | HMM-based method                           | They designed the first NER tagger based on HMM, outperforming the rule-based method on the MUC-6 dataset. |
| Liu **et al**. (2015)[@liuEffectsSemanticFeatures2015]       | CRF-based method                           | The authors implemented a drug name recognition system based on CRF, showing significant improvements compared to dictionary-based methods. |
| **Neural Networks**                                          |                                            |                                                              |
| Collobert **et al**. (2011)[@collobertNaturalLanguageProcessing2011] | CNN-CRF(word-level embedding)              | They were the first to employ word-level representation in the neural-based NER models and achieved 89.59 F1-scores on the CoNLL-2003 English dataset. |
| Huang **et al**. (2015)[@huangBidirectionalLSTMCRFModels2015] | LSTM-CRF(word-level embedding)             | They compared four variants of LSTM-based NER models combining pre-trained word embeddings and other hand-crafted features, and showed that BiLSTM performed the best. |
| Kim **et al**. (2015)[@150806615CharacterAware]              | LSTM-CNN(char-level embedding)             | They used character-level embedding for word representation based on CNN and demonstrated that using character-level information only is sufficient for neural-based NER models. |
| Chiu and Nichols (2016)[@chiuNamedEntityRecognition2016]     | BiLSTM-CNN(char- and word-level embedding) | They concatenated char-level representation generated from CNN and pre-trained word-level embeddings as the inputs of BiLSTM, achieving 91.62 F1-scores on CoNLL-2003. |
| Lample **et al**. (2016) [@lampleNeuralArchitecturesNamed2016] | BiLSTM-CRF(char and word-level embedding)  | They used BiLSTM rather than CNN for char-level representation and CRF to make predictions, serving as the baseline model in recent research. |
| Yang **et al**. (2016)[@yangMultiTaskCrossLingualSequence2016] | GRU-CRF(char and word-level embedding)     | The authors replaced BiLSTM with GRU to study multi-task and cross-lingual learning, showing a slight improvement on the CoNLL-2003 English dataset. |
| **Domain Adaptation**                                        |                                            |                                                              |
| Lee **et al**. (2018)[@leeTransferLearningNamedEntity2018]   | INIT                                       | The INIT method based on fine-tuning was first applied in domain adaptation for NER. |
| Yang **et al**. (2017)[@yangTransferLearningSequence2017]    | MULT                                       | They were the first to design parameter-sharing architectures following the multi-task learning strategy to study cross-domain NER. |
| Lin and Lu (2018)[@lin2018neural]                            | neural adaptation layer                    | They directly modified the BiLSTM-CRF architecture by introducing three layers to perform domain adaptation for NER. |
| Jia **et al**. (2019)[@jiaCrossDomainNERUsing2019]           | language modelling                         | The authors introduced language modelling to deal with the target domain with no labelled data. Their results on CoNLL-2003 exhibited great improvements over supervised domain adaptation methods. |
| Jia and Zhang (2020)[@jiaMultiCellCompositionalLSTM2020]     | the multi-cell structure                   | They proposed a novel transfer method based on MULT from the aspect of the entity-type level instead of the entity-instance level. Their new model outperformed many BiLSTM models using MULT on Twitter dataset and others. |
| Wang **et al**. (2020)[@MultiDomainNamedEntity]              | MultDomain–SP–Aux                          | They employed various domain adaptation methods, including INIT and MULT, to investigate the robustness of NER models when data from multiple domains are available. |





## References

[1] Yan Wen, Cong Fan, Geng Chen, Xin Chen, and Ming Chen. 2020. A survey on named entity recognition. In *Communications, signal processing, and systems*, Springer Singapore, Singapore, 1803–1810.

[2] Arya Roy. 2021. Recent Trends in Named Entity Recognition (NER). *arXiv:2101.11420 [cs]* (January 2021). Retrieved from http://arxiv.org/abs/2101. 11420

[3] Vikas Yadav and Steven Bethard. 2019. A survey on recent advances in named entity recognition from deep learning models. Retrieved from http://arxiv.org/abs/ 1910.11470

[4] J D. Kim, T. Ohta, Y. Tateisi, and J. Tsujii. 2003. GENIA corpus—a semantically annotated corpus for bio-textmining. *Bioinformatics* 19, (July 2003), i180–i182. DOI:https://doi.org/10.1093/bioinformatics/btg1023

[5] Sebastian Ruder. 2019. Neural transfer learning for natural language processing. Thesis. NUI Galway.
