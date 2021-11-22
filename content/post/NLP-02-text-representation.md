---
title: "NLP - Text Representation"
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



### Bag of Word

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
def bagofWord(corpus):
    vocab = set(' '.join(corpus).lower().split())
    word2idx = {word: idx for idx, word in enumerate(vocab)}

    doc_term = []
    for doc in corpus:
        doc_v = [0]*len(vocab)
        for word in doc.lower().split():
            if word in vocab:
                doc_v[ word2idx[word]] += 1
        doc_term.append(doc_v)

    print(word2idx)
    for i, doc_v in enumerate(doc_term):
        print(corpus[i], list(doc_v))

#
# {'eats': 0, 'and': 1, 'dog': 2, 'fish': 3, 'cat': 4, 'meat': 5, 'bones': 6}
# cat eats meat and dog eats meat [2, 1, 1, 0, 1, 2, 0]
# cat eats fish [1, 0, 0, 1, 1, 0, 0]
# dog eats bones [1, 0, 1, 0, 0, 0, 1]
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



### Bag of N-gram

Basically, n-gram is a sequence of N tokens. Generally, we treat each word as an independent unit, in this case, a word is 1-gram or unigram. Similarly, a two-word sequence of words is 2-gram (bigram), three words is 3-gram (trigram), and so on so forth. A simple implementation is shown below.



```python
def create_ngrams(tokens, n=2):
	ngrams = zip(*[tokens[i:] for i in range(n)])
	return [' '.join(gram) for gram in ngrams]
```



So what's the use of n-gram?

- estimate the proability of the last word of an n-gram given the previous words
- assign probabilities to entire sequences



```python
corpus = [
  'Dog bites man',
  'Man bites dog',
  'Dog eats meat',
  'Man eats food'
]

count_vect = CountVectorizer(ngram_range=(1, 2))
bow_rep = count_vect.fit_transform(corpus)

count_vect.vocabulary_
# {'dog': 3, 'bites': 0, 'man': 10, 'dog bites': 4,
# 'bites man': 2, 'man bites': 11, 'bites dog': 1, 'eats': 6, 'meat': 13,
# 'dog eats': 5, 'eats meat': 8, 'food': 9, 'man eats': 12, 'eats food': 7}
```

Pros and Cons

- captures word order and context in a way

- dimentionality increases as $n$ increases
- it still have OOV problem



### TF-IDF

So far, we have learned that one-hot encoding focuses more on the occurrence of words in text while BOW pays more attention to word frequency. In both cases, they consider each word in the corpus euqally (with the same weight). 



In contrast, TF-IDF allows us to measure the importance of each word relative to other words in the doc and the corpus. This is useful for **information retrieval systems**, where we expect that the most relevant documents should appear first. 



How does TF-IDF work? As the name suggests, it calculates two quantities:

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
  

  - The intuiton is fair straightforward — if a word appears across all the docs, for instance, stop words like `a`, `is`, `and`, it's unlikely to capture the characteristics of the doc it belong to. That is, they are more common compared to other less frequent words in the same doc. In other words, we need to reduce its importance, that's why we invert the calculation.

  - we use logarithm to further punish terms that appear more frequently across all the docs




Putting it together, the TF-IDF is defined as


$$
TF\\_IDF = TF(w_i, d_j) * IDF(w_i)
$$


## Distributed Representation



Continuous Bag Of Words (CBOW) uses context words to predict the center word while Skip-Gram use the current word to predict its neighbouring words. The number of context words is determined by a parameter called "window size".



### CBOW

Basically, CBOW is a multi-class classification, which can be descibed as follows,

- words are encoded in one-hot format
- each one-hot encoded vector is fed into the first layer in order to get the embedding of that word
- combine the above real valued vectors in some way such that it captures the overall context
- finally, the linear layer and softmax layer are used to predict probability distribution over vocabulary. The largest prob indicates the most likely target word



```python
class CBOW(nn.Module):
    def __init__(self, voc_size, embd_size):
        super(CBOW, self).__init__()
        self.embedding = nn.Embedding(voc_size, embd_size)
        self.fc = nn.Linear(embd_size, voc_size)
        
    def forward(self, x):
        out = self.embedding(x).sum(1)
        out = self.fc(out)
        out_prob = F.softmax(out)
        return out_prob
```



However, there are two serious problems as one-hot matrix becomes sparse due to increasing vocabulary increases

- the calculation between one-hot and Embedding layer
- the calculation between Embedding layer and the linear layer
- the calculation of softmax layer



#### Vectorization

The first problem is easy to solve because there is no need to store the entire one-hot matrix. After all, it's just used to extract the embedding of the corresponding token, which corrsponds the row of the embdding matrix $E$. In other words, we can just store the row index of the words in one dimensional array.



```python
class CBOW(nn.Module):
    ...
    def one_hot(self, context):
        '''
        context: 'thank very much'
        target: you
        '''
        
        indices = [ lookup[token] for token in context.split() ]
        context_vec = np.zeros(len(indices))

        # instead of one-hot (sparse matrix), we keep the row index of the embedding matrix
        context_vec[:len(indices)] = indices
        context_vec[len(indices):] = mask # in case of the number of tokens in the context is less than the window size
        
        return context_vec

```



One thing we should notice is that same word could occur many times. For example, "Like" occurs twice  in the sentence "Like (Sunday) Like Rain". When performing backpropagation, we should accumulate the gradients for word "Like". This is because that we aggerate the embeddings of all the context words for each instance, and then fed it into a linear layer to make a prediction in the forward process as shown below. 


$$
E_{Like} * 2 + E_{Rain} = E_{aggerate} \\\\ Out = E_{aggerate} * W_{fc}
$$
where $E_{Like}$ is the embedding of word "Like".



```python
out = self.embedding(x).sum(1) # (batch, |D|, |E|)
out = self.fc(out)
```



### Skip-gram



### Negative Sampling



The idea is to transform a multi-class problem into many binary classification problems by sampling the positive example and several negative examples (typically 10 ~ 50) according the frequency distribution of the vocabulary. Frequent words tend to be sampled and rare words is likely not to be sampled, since rare words barely occur in real life and more frequent words would lead to a better generalisation.



### Subsampling

The idea of subsampling is that the vecot representation of frequent words do not change significantly, so we do not need to include them all for each training example.  For example, "the" is frequently used in almost every sentence, so this is no need to train "France" and " the".  Word $w_i$ in the training set will be discarded with a higher probability computed by the formula


$$
P(w_i) = 1 - \sqrt {\frac{t}{f(w_i)}}
$$
where $f(w_i)$ is the frequency of word $w_i$ and $t$ is a chosen threshold (typically around $10^{-5}$). If a word appears very frequently, then $P(w_i)$ is close to $1$, which means it will not be used as the context word and target word. This technique could also be applied to discard less frequent phrases, see [this post](https://medium.datadriveninvestor.com/skip-gram-model-broken-down-subsampling-n-grams-feab04a6f220#:~:text=In%20layman%E2%80%99s%20terms%2C%20subsampling%20is%20to%20sample%20the,to%20sample%20a%20word%20has%20to%20be%20defined.).



### Evaluation

King - man  + woman =  Queen



## Refer

- https://kelvinniu.com/posts/word2vec-and-negative-sampling/
- [Skip-Gram Paper](https://proceedings.neurips.cc/paper/2013/file/9aa42b31882ec039965f3c4923ce901b-Paper.pdf)

