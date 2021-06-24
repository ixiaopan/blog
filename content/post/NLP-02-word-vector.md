---
title: "NLP - Word representation"
date: "2021-06-24"
description: ""
# tags: []
categories: [
    "NLP",
]
series: ["Machine Learning"]
katex: true
draft: true
---



<!--more-->



## Basic vectorization

### One-hot

#### character-level

```python
line = '“Impossible, Mr. Bennet, impossible, when I am not acquainted with him'

cleaned_line = ' '.join(clean_text(line))
cleaned_line, len(cleaned_line)
# ('impossiblemrbennetimpossiblewheniamnotacquaintedwithhim', 55)

# each row represent a character in the sentence
letter_tensor = torch.zeros(len(cleaned_line), 128)
for i, letter in enumerate(cleaned_line):
    idx = ord(letter) if ord(letter) < 128 else 0
    letter_tensor[i][idx] = 1

```



#### word-level



```python
# build vocabulary
vocabulary = set(clean_text(corpus))
word2idx = {w: idx for idx, w in enumerate(vocabulary)}
len(vocabulary), word2idx['impossible']
# (7265, 5746)

# each row represent a word in the sentence
words_in_line = clean_text(line)
word_vector = torch.zeros(len(words_in_line), len(vocabulary))
for idx, w in enumerate(words_in_line):
    j = word2idx[w]
    word_vector[idx][j] = 1

word_vector.shape
# torch.Size([11, 7265])
```



### Bag-of-Word

#### Frequency



#### Occurence



### Bag of N-Grams



### TF-IDF





## Distributed Representation



### CBOW





### Skip-gram





## Pre-trained embedding



### GloVe



