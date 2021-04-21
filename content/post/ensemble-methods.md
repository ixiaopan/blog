---
title: "Ensemble Methods"
date: "2021-04-20"
description: ""
# tags: []
categories: [
    "Machine Learning",
]
series: ["Machine Learning"]
katex: true

---



Ensemble means a group of people or a collection of things. Ensemble methods means rather than using a single model, we will use a group of different models to gain a better prediction. In fact, ensemble methods usually defeat other models in Kaggle competitions. In this post, we will talk about the most popular ensemble methods , including voting, bagging, and boosting.



<!-- more -->



## Voting ensemble



In real life, we often take advices from many people. For example, suppose you want to know whether a movie is worthwhile, you may ask your friends who've watched to give the movie a score(out of 5), say `3, 3, 2, 2, 2, 4`. Since 5 people gave a score that is lower than 4, you may think about choosing another movie, let's say Avatar. Then you ask your friends again and have some scores like this `5, 5, 5, 5, 5`. Wow! All of your friends think that Avatar is an amazing movie that should certainly not be missed. Finally, you decide to watch Avatar. From this example, we can see that gathering plenty of opinions are likely to lead an informed decision.

The method used in this example to decide whether to watch a movie or not is known as voting. For classification, there are 2 types of voting named hard voting and soft voting. Hard voting returns the most popular class shown in Figure 1 while soft voting averages the probability of each class and then return the class that has the maximum probability. For regression, we simply average all the predictions.



![Ensemble Voting](/blog/post/images/ensemble-voting.png "Figure 1: Hard voting classifier predicitons (Hands-on machine learning, 2019)")





### Hard Voting





### Soft Voting





### Average for Regression







## Bagging



bootstrap



random forest



## Boosting







## References

[1] A. Géron, *Hands-on machine learning with Scikit-Learn and TensorFlow*. Sebastopol (CA): O'Reilly Media, 2019.

