---
title: "Deep Learning - Regularization"
date: "2021-11-22"
description: ""
# tags: []
categories: [
    "Deep Learning",
]
series: ["Machine Learning"]
katex: true
---



In the previous post, we introduced how to initialize weights. However, weights could be very large or small (great variance in weights). For larger weights, a tiny change in data will lead to large variance; For smaller weights, it has little influence on model, which can be totally discarded. 



<!--more-->



Regularization is such an technique to set an upper threshold for weights, thereby producing a set of weights with smaller variance. Ridge (L2) and Lasso (L1) are two widely used regularization methods, which can help alleviate overfitting and produce stable computation. Apart from this, dropout is another technique widely used in Deep Learning. In this post, we will explain how regularization works in detail.



## Regularization 



### L2 Ridge



Take linear regression as an example, our goal is to minimize the following loss function with L2 regularization


$$
L_{min} = \sum_{i=0}^m (y_i - \bold x_i^t \bold w)^2 \\\\ \text { subject to } \sum_{j=1}^n \bold w_j^2 \le t
$$
PS: $w_0$ is exclude in L2 regularization. It depends (I am not sure, but pls be careful.)

Obviously, this is an inequality constraints optimization, and we can apply Lagrange to solve it,


$$
L = f(x) - \lambda g(x) \\\\ = \frac{1}{2m} \sum_{i=0}^m (y_i - \bold x_i^T \bold w)^2 + \lambda \sum_{j=1}^n \bold w_j^2 - \lambda t \\\\ = \frac{1}{2m}  (\bold X \bold w - \bold y)^T (\bold X \bold w - \bold y) + \lambda \bold w^T \bold w
$$


where $\lambda \ge 0$ according to KKT conditions. Take the derivative of $L$ w.r.t $\bold w$, we have


$$
\frac{\partial}{\partial \bold w} L = \frac{1}{m}  X^T(\bold X^T \bold w -\bold y) + \lambda \bold w
$$


The let $\frac{\partial}{\partial \bold w} L = 0$, we obtain an analytical solution to $\bold w$


$$
\bold w' = (\bold X^T \bold X + \lambda I)^{-1} \bold X^T \bold y
$$


- $\lambda = 0$, it's the OLS
- $\lambda \rarr \infin$, $w' \rarr 0$



### L1 Lasso

L1 is a bit different from L2. The regularization term is the sum of the absolute value of the weights, or L1 norm. Unfortunately, there is no closed-form solution, so we use gradient descent to find the estimate.


$$
L = (\bold X \bold w - \bold y)^T (\bold X \bold w - \bold y) + \lambda \sum_{d=1}^D |\theta_d|
$$


Take the derivative of $L$ w.r.t $w_k$


$$
\frac{\partial}{ w_k} L = - \sum_{i=0}^m (y_i - \sum_{j \ne k}^d x_{ij} w_j - w_kx_{ik}) x_{ik} \\\\ = - \sum_{i=0}^m (y_i - \sum_{j \ne k}^d x_{ij} w_j) x_{ik} + w_k \sum_{i=0}^m x_{ik}^2 \\\\ \ \\\\  \sim - \rho_k + w_k z_k
$$


where $z_k$ is the sum of the squared features of all the examples. If the data is normalized, $z_k = 1$



### Intuition

Though L1 and L2 both shrink the weights, L1 can further reduce weights to $0$ to make feature selection automatically. Note that L2 is also known as weight decay because it encourages weight values to decay towards 0 in sequential learning.



![](/blog/post/images/lasso-ridge.png#full "Figure 1: Lasso Vs Ridge. The lasso gives a sparse solution.")



From the view of Lagrange, the solution is the intersection point between the loss function and the regularization term, as Figure 1 shows. We can see that L1 is spikier than L2 in high dimension. Thus, the ellipse of the loss function is more likely to touch the sharp points in L1.



## Dropout



Dropout means that we randomly drop out hidden/visible units during training a neural network. The dropout rate is denoted by $p$. When a node is dropped out, all connections including incoming and ougoing connections related to it are removed from the network temporarily (in each epoch, so it could be kept during next epoch.)



For each layer $l$, there is a random variable $\bold r$ following Bernoulli distribution, i.e. $r \sim \text {Bern} (p)$. The value of $r_l$ is either 0 or 1, so with dropout, the forward process is shown below


$$
y_{l}' = r_{l} \odot y_{l} \\\\ \ \\\\ z_{l+1} = w_{l+1} y_l' + b_{l+1} \\\\ \ \\\\ y_{l+1} = \sigma(z_{l+1})
$$




![](/blog/post/images/dropout-network.png "Figure 2: Dropout network. Source from [3]")



When testing, we do not drop out nodes, but we need to scale the network by $1-p$ because we train a sub-network with its weights scaled with $1-p$. From the below equation, we see that the expected hidden value at training time is $(1-p)h_i$ for each input node. Similary, at test time, we hope the same expected value, so we multiply by $1-p$ . In practice, many deep learning libraries will rescale the network by multiplying $\frac{1}{1-p}$ during training phrase, so there is no need to scale the network again during testing.


$$
E(h_i) = (1-p) h_i + p * 0 = (1-p) h_i
$$


Intuitively, dropout is an ensemble method, we combine many different models to reduce variance and improve generalisation. Typically, $p$ is set to $0.5$ to achieve the best regularization.





## Early Stopping



Early stopping is simple and efficient regularization technique, which stops training when the validation error stops consecutive decreasing, typically we observe 5 times.



```python
early_stop_num_epoch = 5

def train():
  ...
  for epoch in range(20):
    if cur_loss < prev_loss:
      epoch_no_improve = 0
	else:
      epoch_no_improve += 1
      
    if epoch_no_improve >= early_stop_num_epoch:
      break
  ...
```





## Refer

[1] [Ridge regression and L2 regularization - Introduction](https://xavierbourretsicotte.github.io/intro_ridge.html)

[2] [does-l2-normalization-of-ridge-regression-punish-intercept-if-not-how-to-solve](https://stats.stackexchange.com/questions/322101/does-l2-normalization-of-ridge-regression-punish-intercept-if-not-how-to-solve)

[3] [Dropout: a simple way to prevent neural networks from overfitting](https://www.cs.toronto.edu/~hinton/absps/JMLRdropout.pdf)

[4] [Simplified Math Behind Dropout](https://towardsdatascience.com/simplified-math-behind-dropout-in-deep-learning-6d50f3f47275)

