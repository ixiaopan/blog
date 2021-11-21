---
title: "NLP - Word Representation"
date: "2021-07-10"
description: ""
# tags: []
categories: [
    "NLP",
]
series: ["Machine Learning"]
katex: true
draft: false
---



In the last post, we talked about text preprocessing techniques. However, even the data is clean now, they are still text. We still haven't answered the question: how to covert text into numbers? In NLP parlance, this is called **text representation**. 



<!--more-->



There are two aspects to consider: the level of representation and the meaning of numbers. We know that a sentence is composed of words and each word consists of a group of characters. This means we can represent text at sentence level, character-level, or both. As for numbers, the simplest way is to count the number of occurrences of each word. The most common methods based on this idea includes one-hot encoding, bag-of-word and TF-IDF.



## Frequency-Based





### One-hot

One-hot representation indicates whether a word/character is present in a sentence/word. If true, we assign the value of 1 to that word, otherwise 0.



#### character-level

Figure 1 shows a simple word representation at character level. The whole vocabulary contains 26 English letters. For each word, for example, the word `impossible`

- each row represents a character in `impossible`, so there are 10 rows
- The corresponding value of each row is a vector whose element value is either 0 or 1, indicating whether the corresponding letter is present in the given word

 Thus, we will get a `(10, 26)` matrix for the word `impossible`.



![](/blog/post/images/word-vocab-matrix.png#full "Figure 1: character-vocabulary occurrence matrix with the shape of (|word|, |vocabulary|)")



Below are  simple codes to construct such a matrix.

```python
line = 'Impossible Mr Bennet impossible when I am not acquainted with him'

line = ''.join(line.lower().split())
line, len(line)
# ('impossiblemrbennetimpossiblewheniamnotacquaintedwithhim', 55)

# each row represent a character in the sentence
alphabet = 'abcdefghijklmnopqrstuvwxyz'
letter_tensor = torch.zeros(len(line), len(alphabet)
for i, letter in enumerate(line):
    idx = alphabet.index(letter)
    letter_tensor[i][idx] = 1

```



#### word-level

From the view of sentences, the vocabulary is all the unique words in the corpus and each row represents each word. The implementation of one-hot encoding at word-level is similar to the above codes.



```python
line = 'Impossible Mr Bennet impossible when I am not acquainted with him'

# build vocabulary
vocabulary = set(line.lower().split())
word2idx = {w: idx for idx, w in enumerate(vocabulary)}
len(vocabulary), word2idx['impossible']
# 10, {'impossible': 0, 'not': 1, 'i': 2, 'with': 3, 'acquainted': 4, 'bennet': 5, 'mr': 6, 'him': 7, 'am': 8, 'when': 9}

# each row represent a word in the sentence
word_vector = np.zeros((len(line.split()), len(vocabulary)))
for idx, w in enumerate(line.lower().split()):
    j = word2idx[w]
    word_vector[idx][j] = 1

word_vector.shape
# (11, 10)
```



The results are shown below, we can see that this sentence can be represented as a  `11 x 10` matrix.

```python
# (|sentence|, |vocabulary|)
Impossible [1, 0, 0, 0, 0, 0, 0, 0, 0, 0]
Mr         [0, 1, 0, 0, 0, 0, 0, 0, 0, 0]
Bennet     [0, 0, 1, 0, 0, 0, 0, 0, 0, 0]

```



#### Pros

- simple and intuitive to understand and implement

#### Cons

- The size of a ont-hot vector is proportional to the size of vocabulary, resulting in a sparse representation when we have a large corpus
- The representation matrix doesn't have fixed size. The dimension varies in sentences or words with different lengths.
- Too naive to capture the similarity between words
- It cannot deal with out-of-vocabulary(OOV) problem



### Bag-of-word

The idea of Bag-of-word(BOW) is that all the words in the corpus are in a bag without considering the orders and context. The intuition is similar to the concepts introduced in LDA — a topic is characterized by a  small specific set of words. Therefore, BOW is commonly used to classify documents. If two documents are similar (have the same words), they are likely to be classified into the same group. Each document is represented as a vector of `|V|` dimensions, where the element value of this vector is the frequency of the word in the corresponding doc. Say we have the following corpus,



```python
corpus = [
  'cat eats meat and dog eats meat',
  'cat eats fish',
  'dog eats bones'
]
```



The vocabulary of this small corpus and the doc-term matrix are



```python
['and' 'bones' 'cat' 'dog' 'eats' 'fish' 'meat']

cat eats meat and dog eats meat [1, 0, 1, 1, 2, 0, 2]
cat eats fish [0, 0, 1, 0, 1, 1, 0]
dog eats bones [0, 1, 0, 1, 1, 0, 0]


```



#### one-hot

Sometimes, we don't care about the number of occurrence of words. Just like one-hot encoding, we only want to know whether a word is present in the sentence or not, which would be useful for sentiment analysis. Well, that's easy to implement using Sklearn



```python
# occurrence
vectorizer = CountVectorizer(binary=True)

X_train_dtm = vectorizer.fit_transform(X_train)
X_test_dtm = vectorizer.transform(X_test)

```



#### Pros

- Simple and intutive to understand and implement
- Captures the semantics of documents, if two docs have similar words, they will be close to each other in the word space
- Fixed matrix representation no matter how long a sentence is

#### cons

- It ignores the word order(context), so `Cat bites man` and `Man bites cat` have the same representation
- The size of the matrix is proportional to the size of vocabulary
- It doesn't deal with out-of-vocabulary(OOV) problem
- It doesn't capture the similarity between different words that have the same meaning, e.g `cat eats, cat ate`, BOW will treat them as different vectors though they convey the same semantics.
- As mentioned earlier, the most frequent words are often function words like pronouns, determiners and conjuctions. However, they are of no help for classification.



### TF-IDF



So far, we have learned that one-hot encoding focuses more on the occurrence of words in text while BOW pays more attention to word frequency. In both cases, they consider each word in the corpus euqally(with the same weight). 

In contrast, TF-IDF or term frequency-inverse document frequency allows us to measure the importance of each word relative to other words in the doc and the corpus. This is useful for information retrieval systems, where we expect that the most relevant documents should appear first.



So how does TF-IDF work? Well, as the name suggests, it calculates two things

- term frequency(TF), the normalized frequency of each token $w_i$ in a given doc $d_j$

  
  $$
  TF(w_i, d_j) = \frac{|w_i^{d_j}|}{|d_j|}
  $$
  

  - The intution is that the more frequent a word appears in a doc, the more important it is. Thus, we need to increase its importance

  

- inverse document frequency(IDF), the logarithm of the inverse normalized frequency of each token across all documents

  ​	
  $$
  IDF(w_i) = \text {log} \frac{|D|}{|w_i^D|}
  $$
  

  - The intuiton is fair straightforward — if a word appears across all the docs, for instance, stop words like `a`, `is`, `and`, it's unlikely to capture the characteristics of the doc it belong to, indicating that it's a more common word relative to other words in the same doc. In other words, we need to reduce its importance, that's why we invert the calculation.

  - we use logarithm to further punish terms that appear more frequently across all the docs

    



Putting it together, the TF-IDF is defined as


$$
TF\\_IDF = TF(w_i, d_j) * IDF(w_i)
$$


## Distributed Representation



### CBOW



### Skip-gram



### pre-trained

