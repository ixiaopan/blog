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



Ensemble means a group of people or a collection of things.Thus, ensemble methods means rather than using a single model, we will use a group of different models to gain a better prediction. In fact, ensemble methods often outperform other models in Kaggle competitions. In this post, we will talk about the most popular ensemble methods , including voting, bagging, and boosting.



<!--more-->





## Example



In real life, we often take advices from others. For example, suppose you want to know whether a movie is worthwhile to watch, you may ask your friends who've watched to give the movie a score(out of 5), say `3, 3, 2, 2, 2, 4`. Since 5 people gave a score that is lower than 4, you may think about choosing another movie, let's say Avatar. Then you ask your friends again and have some scores like this `5, 5, 5, 5, 5`. Wow! All of your friends think that Avatar is an amazing movie that should certainly not be missed. You agree with their opinons and decide to watch Avatar finally. From this example, we can see that **gathering plenty of opinions from different people** are likely to make an informed decision.



Here, I highlight the words **different people**. It makes little sense if we only asks for people who have the same interests. Therefore, the more diverse the people are, the more sensible our decisions are. Basically, the idea behind it is the wisdom of collaborating.



## Voting



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



```python
from sklearn.ensemble import BaggingClassifier 
from sklearn.tree import DecisionTreeClassifier

ensemble_clf = BaggingClassifier(DecisionTreeClassifier(), n_estimators=300, max_sample=100, bootstrap=True)

```



### Random Forest



Bagging can be used for any models. Among them random forest is the special one. As its name suggests, it is exclusively designed for decision trees. Besides, it introduces extra randomness when growing trees. 



![](/blog/post/images/random-forest.jpeg "Figure 3: A random subset of features at each split for each tree (Reference [2])")



Specifically, it randomly choose a subset of $m'$ of the features at each split instead of using all features shown in Figure 3. By doing so, all trees can have much different training data set further, so they are **less similar** to each other, which results in more significant predictions.



```python

from sklearn.ensemble import RandomForestClassifier, BaggingClassifier 
from sklearn.tree import DecisionTreeClassifier

random_forest_clf = BaggingClassifier(splitter='random', DecisionTreeClassifier(), max_leaf_nodes=16,n_estimators=300, max_sample=100, bootstrap=True)

## is equivalent to this
random_forest_clf=RandomForestClassifier(n_estimators=300, max_leaf_nodes=16)

```



### Extra-Trees

TODO





### Why Bagging works



Take random forests as an example, each decision tree is a machine learned from a data set. Based on the theory of bias and variance, we know that the mean meachine can be expressed as $f'_m=E_D[f'(x|D)]$. Thus, $f'(x|D)$ can be interpreted as a random variable $X$, and $f'_m$ can be described as $\mu = E(X)$. 



Since we have $N$ decision trees in a random forest, there are $N$ random variables $X_i$, where $\mu = E(X_i)$. We can construct a new random variable $Z = \frac{1}{n}\sum_i^nX_i$, which represents the mean of $n$ random **independent** variables $X_i$. 



The expected value of $Z$ is given by


$$
E[Z] = E[ \frac{1}{n}\sum_i^nX_i ] =\frac{1}{n} E[\sum_i^nX_i] = \frac{1}{n} nE[X_i] = \mu
$$




The variance is


$$
 E[(Z - E[Z])^2] = E[(\frac{1}{n}\sum_i^nX_i - \mu)^2] = \frac{1}{n^2} E[(\sum_i^n (X_i - \mu))^2]
$$

$$
= \frac{1}{n^2}E[\sum_i^n(X_i - \mu)^2 + \sum_i^n\sum_{j=1,i \ne j}^n (X_i -\mu)(X_j -\mu)]
$$


Since $X_i$ is independent of $X_j$ ( $i\ne j$ ),


$$
E[\sum_i^n\sum_{j=1,i \ne j}^n (X_i -\mu)(X_j -\mu)] = 0
$$




we are left with


$$
E[(Z - E[Z])^2] = \frac{1}{n^2}E[\sum_i^n(X_i - \mu)^2] = \frac{1}{n} \sigma^2
$$


where $E[(X_i-\mu)^2]=\sigma^2$.



From the above euqation, we can see that ensemble methods reduce variances as $n$ increases when our models are **uncorrelated**.



On the contrary, if our models are correlated with the correlation coefficient $\rho$


$$
\rho = \frac{E[(X_i - \mu)(X_j - \mu)]} {\sqrt{\sigma^2(X_i)}\sqrt{\sigma^2(X_j)}}
$$


The variance is 


$$
E[(Z - E[Z])^2] = \frac{1}{n} \sigma^2 + \frac{n-1}{n} \rho \sigma^2 = \rho \sigma^2 + \frac{(1 - \rho)}{n} \sigma^2
$$


As $n$ increases, the second term vanishes and we are left with the first term. Therefore, if we want the ensemble methods to do well, we need our models to be uncorrelated.



**Random forests do a good job because of both the randomness of training data sets sampled via bootstrapping and the randomness of features considered at each split.** The algorithm results in much less correlated trees, which trades a higher bias for a lower variance, generally yielding better results.





### Feature Importance



Like decision tree, random forests can tell us the relative importance of each feature. It is measured by calculating the sum of the reduction in impurity over all the nodes that are split on that feature.



```python

random_forest_clf.feature_importances_

```





## Boosting





## References

[1] A. Géron, *Hands-on machine learning with Scikit-Learn and TensorFlow*. Sebastopol (CA): O'Reilly Media, 2019.

[2] T. Yiu, “Understanding random forest - towards data science,” Towards Data Science, 12-Jun-2019. [Online]. Available: https://towardsdatascience.com/understanding-random-forest-58381e0602d2. [Accessed: 23-Apr-2021].



https://towardsdatascience.com/ensemble-methods-bagging-boosting-and-stacking-c9214a10a205

https://towardsdatascience.com/an-implementation-and-explanation-of-the-random-forest-in-python-77bf308a9b76

