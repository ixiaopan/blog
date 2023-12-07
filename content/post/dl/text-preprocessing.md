---
title: "NLP - Text Preprocessing"
date: "2021-06-22"
description: ""
# tags: []
categories: [
     "Deep Learning",
]

katex: true

---



From now on, we will focus on a specific domain — Natural Language Processing(NLP), in part because my summer project is about Named Entities Recognition(NER). Therefore, I need to know some text preprocessing techniques and have a good understanding of the state-of-art NLP models, particularly BiLSTM + CRF. The very first step in NLP is text preprocessing, so I am going to start from here.



<!--more-->



## Converting text to lowercase

For some letter-based languages such as English, letters could be either lowercase or uppercase, e.g. `book` , `Book` or `BOOK`. Since they represent the same word `book` with the same semantics, there is no need to encode them three times. The common way is to convert all words into a consistent written style — lowercase because of its readable and concise style.



However, this method sometimes could affect semantics of some specific words, for instance,  `May` and `may` are two different words. Well, for tasks that involve parts-of-speech analysis, it's a matter.



In python, it's easy to convert a string into lowercase,



```python
text = text.lower()
```



## Removing Punctuation



For some tasks, punctuations such as `, . " @ #` are meaningless to us, so we should remove them. In Python, we can do this using  the `sub()` method of the `re` module to replace any matched punctuation with an empty character



```python
re.sub('[,\'.!"]', '', '"20.2 dollars! That\'s impossible", he said.')
#>> 202 dollars Thats impossible he said

```



From the above code, we can see some issues, 

- Abbreviation, `That's -> Thats` 
- Prices, `20.2 dollars -> 202 dollars`



Maybe we can write some specific rules for a corpus to remove particular punctuations, but it will be time-consuming and inflexible.



## Tokenization

Tokenization is a technique to split a piece of text into a smaller unit. The unit could be 

- a single word
- a character 
- a combination of several words



Why do we tokenize text? The answer is that the ultimate goal of tokenization is to build vocabulary for a corpus. After all, words in each sentence are ultimately from the vocabulary. 

Okay, so how do we perform tokenization to get a sequence of words? A naive way is to split a string by whitespace shown below.



```python
text = text.split(' ')
```



However, there are some issues. For example, `United Kindom` will be split into two words,  `United` and `Kindom`. A common practice is to use the popular [natural language toolkit (NLTK)](https://www.nltk.org/]) to do this



```python
from nltk.tokenize import word_tokenize

tokens = word_tokenize(text)

```



With these tokens, how to build vocabulary? Usually, there are two ways to do this,

- include each uniqe token in the vocabulary
- include only the top K frequently occurring tokens; the idea is that repetition usually conveys the most important information



## Stemming & Lemmatization



In grammatical aspect, a word could have several variants to express tense, mood or something. For example,

- `go, goes, went, gone` are different tenses of `go` 
- `higher, highest` are comparative and superlative form of `high`



In other words, these variants are originated from their **root** words. Generally, most words follow a general rule of to generate the corresponding forms, though there are some rare cases, as shown below,

- `eat, ate, eaten`
- `good, better, best`



Stemming and Lemmatization are two common methods to convert each word back to its original form or the root. 



### Stemming



Stemming is a simple and crude method that simply chopps off letters from the end of a word until **a common root** is found, which is the idea of Potter Stemming algorithm. 



```python
from nltk.stem import PorterStemmer, WordNetLemmatizer

words = ['run', 'runner', 'running', 'ran', 'runs', 'easily', 'saw']
p_stemmer = PorterStemmer()
for word in words:
    print(word + '-->' + p_stemmer.stem(word))

# run-->run
# runner-->runner
# running-->run
# ran-->ran
# runs-->run
# easily-->easili
# saw-->saw 
```



We can see that the result of stemming may not be a word, e.g. `easili`, and that the tense forms of some special words like `ran, saw` are not processed correctly.



### Lemmatization

On the contrary, lemmatization returns the **true root** or **lemma** of a word by considering the whole vocabulary of a language and the context of that word in the sentence. In the case of `ran` and `saw`, the true root words or lemmas are `run` and `see` respectively. 

NLTK provides a popular lemmatizer for us — WordNet Lemmatizer, which is an large lexical database of English. But before using it, we need to know the parts of speech of each word, which can also be done using `pos_tag()` method of NLTK.



```python
# we mannually specify the parts of speech for each word
words = [
("grows", 'v'), ('running', 'v'), ("better", 'a'), ("cats", 'n'), ('quickly', 'r')
]

lemmatizer = WordNetLemmatizer()
[ lemmatizer.lemmatize(w, p) for w, p in words ]
# ['grow', 'run', 'good', 'cat', 'quickly']
```



## Removing stopping words



It seems that we've obtained the tokens as desired. But wait, let's plot the number of occurrence of each word. 



![](/blog/post/images/tokens_counts.png#full "Figure 1: The top 10 frequently tokens")



Figure 1 shows the frequency of each token in the built-in NLTK corpus named `1789-Washington.txt`. It can be seen that most tokens are determiners, prepositions, conjuctions or other **function words** that have little lexical meaning. Obvisously, we need to remove them, which can be done easily using NLTK,



```python
from nltk.corpus import stopwords

stop_words_en = stopwords.words('english')

tokens = [ w for w in tokens if w not in stop_words_en ]

```





## Putting it together



So far, we've introduced some basic and necessary text preprocessing techniques. Now let's put it together to build the whole text preprocessing pipeline.



```python
import string, re
import nltk
from nltk.corpus import wordnet
from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords
from nltk.tag import pos_tag
from nltk.stem.wordnet import WordNetLemmatizer


def get_wordnet_pos(tag):
    if tag.startswith("J"):
        return wordnet.ADJ

    elif tag.startswith("R"):
        return wordnet.ADV
    
    elif tag.startswith("V"): 
        return wordnet.VERB
    
    else:
        return wordnet.NOUN

lem = WordNetLemmatizer()
stop_words_en = stopwords.words('english')

def clean_sentence(text):
    # lower
    text = text.lower()

    # remove punctuation
    text = re.sub('\n', ' ',text)
    text = text.translate(str.maketrans('','',string.punctuation))

    # tokenization
    # text.split(' ') # naive methods, e.g. New York will be splitted into ['New', 'York']
    tokens = word_tokenize(text) 

    # lemmatize, [a, go, c, ...]
    tokens = pos_tag(tokens) # [(a, 'NN'), (went, 'VB'), (c, 'NN')]
    tokens = [ lem.lemmatize(w, get_wordnet_pos(tag)) for w, tag in tokens ]

    # remove stopping words
    tokens = [ w for w in tokens if w not in stop_words_en ]
    tokens = [ w for w in tokens if len(w) > 1 ]

    return ' '.join(tokens)

```



