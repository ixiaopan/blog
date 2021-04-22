---
title: "Ensemble Methods"
date: "2021-04-20"
description: "Ensemble means a group of people or a collection of things. Ensemble methods means rather than using a single model, we will use a group of different models to gain a better prediction. In fact, ensemble methods usually defeat other models in Kaggle competitions. In this post, we will talk about the most popular ensemble methods , including voting, bagging, and boosting."
# tags: []
categories: [
    "Machine Learning",
]
series: ["Machine Learning"]
katex: true

---



Ensemble means a group of people or a collection of things. Ensemble methods means rather than using a single model, we will use a group of different models to gain a better prediction. In fact, ensemble methods usually defeat other models in Kaggle competitions. In this post, we will talk about the most popular ensemble methods , including voting, bagging, and boosting.



<!--more-->



In real life, we often take advices from others. For example, suppose you want to know whether a movie is worthwhile to watch, you may ask your friends who've watched to give the movie a score(out of 5), say `3, 3, 2, 2, 2, 4`. Since 5 people gave a score that is lower than 4, you may think about choosing another movie, let's say Avatar. Then you ask your friends again and have some scores like this `5, 5, 5, 5, 5`. Wow! All of your friends think that Avatar is an amazing movie that should certainly not be missed. You agree with their opinons and decide to watch Avatar finally. From this example, we can see that **gathering plenty of opinions from different people** are likely to make an informed decision.



Here, I highlight the words **different people**. It makes little sense if we only asks for people who have the same interests. Therefore, the more diverse the people are, the more sensible our decisions are.





## Voting ensemble



The method used to decide whether to watch a movie or not in this example is known as **voting**. For classification, there are 2 types of voting named **hard voting** and **soft voting**. 

- Hard voting returns the most popular class shown in Figure 1.
- Soft voting averages the probability of each class and then return the class that has the maximum probability. 



### Hard Voting



![Ensemble Voting](/blog/post/images/ensemble-voting.png "Figure 1: Hard voting classifier predicitons (Hands-on machine learning, 2019)")





Figure 1 can also be illustrated  in a mathematical way,


$$
y' = mode(C_1(x), C_2(x), ..., C_n(x))
$$




For example, `{0, 1, 0, 1, 1}` are the class labels predicted by our 5 different classifiers for a data point $x$. By hard voting, the final class label is `class 1` .



```
C1 -> 0
C2 -> 1
C3 -> 0
C4 -> 1
C5 -> 1
```



### Weighted Hard Voting



Hard voting works nice, but in some cases, some people might be more professional than others. Hence, their opinions are much more significant. How to distinguish professionals and common people?

We assign weights to them. Specifically, we assign higher weights to professionals while common people have lower weights. Then we calculate weighted sum of occurrence of each class label and find the class label that has the maximum value.


$$
y' = \operatorname*{argmax}_i w_j\sum_j [C_j == i]
$$




where $[C_j == i] = 1$ if classifier $j$ predicts class label i and 0 otherwise.



For example, if we assign the following weights to the previous 5 classifiers, then we will have `0.7` for class 0 and`0.5` for class 1. Thus, `class 0` wins because 0.7 is greater than 0.5.



```
0.4, C1 -> 0
0.1, C2 -> 1
0.3, C3 -> 0
0.2, C4 -> 1
0.2, C5 -> 1
```



### Soft Voting

Instead of predicting the class label directly, some classifiers like logistic regression can predict the probability of each class label that $x$ belongs to. Then we simply average these probabilities for each class label. Certainly, you can assign weights to classifiers.


$$
y' = \operatorname*{argmax}_i \frac{1}{n} \sum_j^n w_j p_{ij}
$$


where $p_{ij}$ is the probability of class label $i$ that $x$ belongs to when using classifier $C_j$.







### Average for Regression



We simply **average the predictions of different machines** for a regression task.




$$
y' = \frac{1}{n} \sum_j^n w_j C_j
$$




## Bagging



In order to make our models different from each other, we use various algorithms to train the same data, as discussed above. Another way to have a set of diverse models is to train the same model on different data sets. But usually we only have one training data set. Where do other data sets come from? Well, they are sampled with replacement from the original data set, which is known as **bootstrapping**.



![The process of bagging](/blog/post/images/bootstrap.png "Figure 2: The process of bagging  (Hands-on machine learning, 2019)")



Specifically, given a training data set $D=(x_i, y_i)_i^n$ of the size $N$, we build an ensemble model of size $m$ according to the following steps:



- For $i=1, 2, 3, ..., m$
  - draw $N'(N' \le N)$ samples with replacement from $D$, which is denoted by $D^\*_i$
  - build a model (e.g. decision tree) $T^\*_i$   based on  $D_i^\*$

- For an unseen data, aggregate the predictions of all $T^\*$ 
  - perform a majority vote for classification
  - average the predictions for regression



## Random Forest







## Boosting





## References

[1] A. Géron, *Hands-on machine learning with Scikit-Learn and TensorFlow*. Sebastopol (CA): O'Reilly Media, 2019.

