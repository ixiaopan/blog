---
title: "Probabilistic Model"
date: "2021-05-26"
description: "In the previous post [Descriptive Statistics](https://ixiaopan.github.io/blog/post/descriptive-statistics/), we focused on summary statistics on a particular data set rather than inference. However, in machine learning, we are usually attempting to make inference. For instance, given the training images with 2 categories, say 'Cat' and 'Dog', which one does a new unseen image belong to? Typically, we compare the probability of being classified into 'Cat' or 'Dog'."
# tags: []
categories: [
    "Machine Learning",
]
series: ["Machine Learning"]
katex: true
---



In the previous post [Descriptive Statistics](https://ixiaopan.github.io/blog/post/descriptive-statistics/), we focused on summary statistics on a particular data set rather than inference. However, in machine learning, we are usually attempting to make inference. For instance, given the training images with 2 categories, say 'Cat' and 'Dog', which one does a new unseen image belong to? Typically, we compare the probability of being classified into 'Cat' or 'Dog'. Here, we use **probability** to describe uncertainty and Bayes' theorem to make a classification. Perhaps you've already been familiar with them when you were in college, but today we will revisit them from the aspect of the application in machine learning. Hopefully you can gain a new sight from this.



## Probability



At the beginning, let's have a quick refresh on probability. Conventionly, we use a capital letter, such as $X$ or $Y$, to represent a random variable. Suppose we have two random variables of interest, $X$ and $Y$,

- The joint probability of $X$ that takes the value of $x$ and $Y$ that takes the value of $y$ is written as $P(X = x, Y=y)$, which means that the probability of $x$ and $y$ happening at the same time

  

- Given $Y=y$, the conditional probability of $X$ given $Y=y$ is denoted by $P(X|Y=y)$



There are two major rules of probability that we should remember,



- the sum rule, i.e. the marginal probability of $X$ that takes the value of $x$, irrespective of the value of $Y$


$$
P(X=x) = \sum_{y \in Y} P(X=x, Y=y)
$$


- the product rule, i.e. the joint probability of $X$ and $Y$ can be written as the product of the conditional probability and the marginal probability


$$
P(X, Y) = P(Y|X) P(X) = P(X|Y)P(Y)
$$


From the above formula, we can deduce the following equation, which is also known as Bayes' Rule,


$$
P(Y|X) = \frac{P(X|Y)P(Y)}{P(X)}
$$




## Bayes' Inference



In machine learning, our goal is to learn $\Theta$ from a given $D$, but $\Theta$ is uncertain. One way to measure uncertainty is probability, so the question becomes how to compute the probablity of $\Theta$ given the data $D$, i.e. $P(\Theta|D)$.  But the only thing we know is the observed data, so how can we do it? The answer is the Bayes' Rule, which helps us convert this into a forward problem, where parameters are known and thus we can draw data from the distribution determined by that paramaters. The formula is defined as follows,


$$
P(\Theta|D) = \frac{P(D|\Theta)P(\Theta)}{P(D)}
$$


- $P(\Theta)$ is called the prior probability, i.e. the best guess about $\Theta$ before we see the data. You might have a good estimate on it or simply have no idea at all
- $P(D|\Theta)$ is the likelihood of the data given the parameters $\Theta$ 
- $P(D)$ is the evidence or marginal probability denoted by $P(D) = \sum_{\theta \in \Theta }P(D, \theta)$
- $P(\Theta|D)$ is the posterior probability, i.e. the updated probability of $\theta_i$ after we see the data



### Pros and cons

Pros

- Bayes' rule provides us a full probabilistic description of the parameters
- It doesn't overfit since we are not choosing the best parameters that fit the data perfectly
- In some cases, the prior and the likelihood can be easy to be defined

Cons

- However, we need to compute $P(D)$, which is not reasonable sometimes, e.g. there are too many possible values of $\Theta$
- Posterior might not be described as a nice probability function



### MAP

If we ignore the evidence $P(D)$ (after all, it's just a constant for any $\theta$), we are only left with the numerator. An easy way to compute $P(\Theta|D$) is to find the maximum value, though it's not a strictly probability


$$
{\argmax}_{\Theta} log (P(D|\Theta)) + log (P(\Theta))
$$


This method is called Maximum a Posterior. However, it can overfit because we are finding the parameters that maximising the likelihood of the observed data, which means we will get a model that fit the data with no errors.



### MLE



## Conjugate Prior



### Bernoulli Distribution



#### Beta



### Poisson Distribution



#### Gamma



### Incremental Updates





## Discriminal vs Generative





## Graphical Models





## Latent Dirichlet Allocation



### Latent Variable





