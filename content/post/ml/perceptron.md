---
title: "Perceptron"
date: "2021-06-13"
description: "Perceptron is a traditional classification algorithm and it is the basis of the neural network. Though it's out of date, it plays a historical role in the development of neural network. Knowing how it works will help us lay the foundations for the future study of the neural network."
# tags: []
categories: [
    "Machine Learning",
]
katex: true
---



Perceptron is a traditional classification algorithm and it is the basis of the neural network. Though it's out of date, it plays a historical role in the development of neural network. Knowing how it works will help us lay the foundations for the future study of the neural network.





<!--more-->



## Model



Perceptron is a linear binary classifier, where linear means that it separates the input space linearly and binary means that it classify a sample into one of the two categories. Let $\bold x_1, \bold x_2, \bold x_3, ..., \bold x_n$ be observations and $y_n \in -1, 1$ be target classes, Perceptron model is defined as follows,


$$
\hat y_n = f( \bold w^T \bold x_n)
$$


where $f(x)$ is an active function


$$
f(x) = \begin{cases} 1 & \text{if $x \ge 0$} \\\\ -1 & \text{otherwise}\end{cases}
$$


## Loss Function



If we classifiy correctly, $\hat y_n $ will have the same sign as $y_n$. Otherwise, it will have the opposite sign to $y_n$. 


$$
g(x) = \begin{cases} \hat y_n y_n \ge 0& \text{if classified correctly} \\\\ \hat y_n y_n \lt 0 & \text{otherwise}\end{cases}
$$


At the same time, we find that the sign of $\hat y_n$ is determined by $w^Tx$, so we have


$$
g(x) = \begin{cases} y_n (\bold w^T \bold x_n) \ge 0& \text{if classified correctly} \\\\ y_n (\bold w^T \bold x_n) \lt 0 & \text{otherwise}\end{cases}
$$


We don't care about what value of $w^tx_n$ is, instead, we are more interested in its sign. Naturally, the loss function is defined as follows,


$$
L = -\sum_{n \in M} y_n (\bold w^T \bold x_n)
$$


where $M$ is the set of misclassified points. It's clear that the loss function will reach the minimum point if we classify all samples correctly.





## Parameter Estimation



Though we could take the derivative of $L$ w.r.t. $\bold w$


$$
\frac{\partial L}{\partial \bold w} = -\sum_{n \in M} \bold x_n y_n
$$


Obviously, we cannot set this equal to 0 and get a solution for $\bold w$. A common way to find the optimal parameter is to use **gradient descent**, as shown below


$$
\bold w^{t+1} = \bold w^t + \eta \sum_{n \in M} \bold x_n y_n
$$


where $\eta$ is the learning rate. The updating of $\bold w$ could be iteratively — update $w$ when a sample is misclassified each time. Since $y_n \in -1, 1$, we can decompose the above equation into the below two equations


$$
\bold w^{t+1} = \begin{cases} \bold w + x_n & \text{if $y_n = 1$} \\\\ \bold w - x_n  & \text{if $y_n = -1$}\end{cases}
$$


If a point belongs to positive category, i.e. $y_n = 1$, but our model misclassifies it as a negative category, then we would move $\bold w$ along the direction towards this point because of  $w + x_n$ and vice versa.



## Algorithm



Putting it together, the whole algorithm of Perceptron is

- Randomly initialise $w$
- Until convergence or some stopping rules reached,
  - for $x_1, x_2, ..., x_n$
    - compute $\hat y_n = f(w^T x_n)$
    - if $\hat y_n y_n < 0$,
      - if $y_n = 1$, update $w' = w + \eta x_n$
      - if $y_n = -1$, update $w' = w - \eta x_n$

