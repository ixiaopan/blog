---
title: "Logistic Regression"
date: "2021-06-14"
description: ""
# tags: []
categories: [
    "Machine Learning",
]

katex: true
---



We've known that linear regression can be used to predict a continuous value, but sometimes the target variable might be categorical, i.e. whether tomorrow is sunny or someone has a cancer. Can we still use linear regression to solve this classification problem? The answer is Yes and the algorithm that we will introduce in this article is known as logistic regression. Well, we can also try Perceptron since it's also a linear classifier. PS: Don't mix it up with linear regression. Logistic regression is a classification algorithm.



<!--more-->



## Logistic Function



Given an input dataset $D = [x_1, x_2, ..., x_n]$ with the corresponding lable $Y = [y_1, y_2, ..., y_n]$ where $x_i \in R^d$ and $y_i \in {0, 1}$, we want to construct a linear classifer that gives us a value $z_i$ as a weighted sum of all features


$$
z_i = w^T x_i + b
$$


### Range issue

Now the question is how $z_i$ relate to $y_i$ since the range of $z_i$ is from $-\infin$ to $\infin$ while $y_i$ is either 0 or 1. Any value that is larger then 1 or lower than 0 has no meaning because we interpret $y_i$ as probability.



### Log-odds

Obviously, we need to come up with a way to transform $z_i$ so as to obtain a value that lies between 0 and 1. But how? What about $f(x) = \text{log} x$ ? No, it fails since the domain of the log function can only be positive. Maybe we could make a modification about the log function like this, 




$$
z_i = \text{log } \frac{p_i}{1 - p_i}
$$




where $p/(1-p)$ is called odds and $z$ is often called the logit. The inverse of the log-odds function, also known as **sigmoid function**, is defined as follows, 


$$
\hat y_i = p_i = \frac{1}{1 + e^{-z_i}}
$$




Figure 1 shows what sigmoid function looks like. We can see that log-odds helps us to solve the range problem.



![](/blog/post/images/log-odds.png "Figure 1: sigmoid activation function")



## Parameter Estimation



We've defined the logistic model, the next step is to estimate the parameters, i.e. $w$. Minimising the loss function and MLE are two common ways to solve this. In fact, minimising the loss is equivalent to MLE in logistic regression.



### Binominal

In the case of binary classification, $y_i$ follows Bernoulli Distribution, i.e. $y_i \sim Bern(n, p_i)$


$$
p(y_i = 1 |x_i) = p_i
$$

$$
p(y_i = 0 |x_i) = 1 - p_i
$$


The likelihood of a single sample is,


$$
p(y_i|x_i) = p_i^{y_i} (1-p_i)^{1-y_i}
$$


So the likelihood function of the whole training data is,




$$
L = \prod_{i=1}^N p(y_i|x_i)
$$


The log likelihood function is given by




$$
\text{log} L = \sum_{i=1}^N \text{ log } p(y_i|x_i) = \sum_{i=1}^N  y_i\text{log} p_i + (1-y_i) \text{log} (1-p_i)
$$

$$
= \sum_{i=1}^N \text{log} (1-p_i) + \sum_{i=1}^N y_i \text{log} \frac{p_i}{1-p_i}
$$

$$
= \sum_{i=1}^N \text{log} (1-p_i) + \sum_{i=1}^N y_i  (w^Tx_i + b)
$$

$$
= \sum_{i=1}^N \text{log} (1- \frac{1}{1 + e^{-z_i}}) + \sum_{i=1}^N y_i  (w^Tx_i + b)
$$



$$
= \sum_{i=1}^N -\text{log} (1 + e^{(w^Tx_i + b)}) + \sum_{i=1}^N y_i  (w^Tx_i + b)
$$


Our goal is to maximise the log likelihood function, so we take the derivative of $L$ w.r.t $w$


$$
\frac{\partial L}{\partial w_j} = -\sum_{i=1}^N \frac{e^{(w \cdot x_i)}}{1+e^{(w \cdot x_i)}}  x_{ij} + \sum_{i=1}^N y_i  x_{ij}
$$

$$
= \sum_{i=1}^N  (y_i - z_i)  x_{ij}
$$


However, this is a transcendental equation(an equation containing the variables being solved for), which means we cannot solve analytically, and thus there is no closed-form solution. But we can still find the estimation of $w$ through gradient descent using the above derivative.



The loss function used in logistic function is defined as follows,


$$
L = - \sum_{i=1}^N y_i \text{log} p_i - (1-y_i) \text{log} (1-p_i)
$$


which is often called the 'Binary Cross Entropy'. Obviously, the loss function is exactly the negative log likelihood function. Thus, maximising likelihood is equal to minimising the loss. 



```python
def crossEntropyLoss(X, y, theta):
  epsilon = 1e-12

  p = logistic(X @ theta)

  cost = -np.mean(y * np.log(p + epsilon) + (1 - y) * np.log(1 - p + epsilon))

  return cost
```



### Multinominal

What if we have more than one target category? It's not a simple yes or no classification since now we have many possible classes.  Suppose there are $k$ classes, and each sample must belong to one class, so we have


$$
\sum_{j=1}^k p_{i, j} = 1
$$


Just like the binominal case, we choose one class as the reference model and estimate the 'odds'


$$
z_{i,j} = \text{log } \frac{p_{i, j}}{p_{i, J}} = \bold w_j ^Txx_i + b_j
$$

$$
p_{i, j} = p_{i, J} e^{z_{i, j}} = p_{i, J} e^{\bold w_j ^Tx + b_j}
$$




To obtain $p_{i, j}$, we need to eliminate $p_{i, J}$. Combing the two equations, we have


$$
\sum_{j=1}^k p_{i, j} = p_{i, J} \sum_{j=1}^k  e^{z_{i, j}}  = 1
$$

$$
\text{=> } p_{i, J}  = \frac{1}{\sum_{j=1}^k  e^{z_{i, j}} }
$$


plug $p_{i, J}$ into $p_{i,j}$, we have


$$
p_{i, j} = \frac{ e^{\bold w_j ^Tx + b_j} }{\sum_{j=1}^k  e^{z_{i, j}} }
$$


which is known as **softmax function**. 

Again, we use MLE to estimate $\bold w$. The difference from binominal case is that now there are $k$ estimates we need to calculate. The likelihood of a single sample is,


$$
p(y_i|x_i) = \prod_{j=1}^{k} p_{ij}^{I_{ij}}
$$


where $I_{ij} = 1$ if $x_i$ belong to class $j$, otherwise $I_{ij}=0$. The likelihood function of the whole training data is,


$$
L = \prod_{i=1}^N \prod^k_{j=1} p_{ij}^{I_{ij}}
$$


The log likelihood function is given by


$$
\text{log } L = \sum_{i=1}^N \sum_{j=1}^k I_{ij} \text{log }(p_{ij})= \sum_{i=1}^N \sum_{j=1}^k I_{ij} \text{log } \frac{ e^{\bold w_j ^Tx_i + b_j} }{\sum_{m=1}^k  e^{z_{i, m}} }
$$

$$
= \sum_{i=1}^N \sum_{j=1}^k I_{ij} (z_{ij} - \text{log } \sum_{m=1}^k e^{z_{i,m}})
$$




Now let's take the derivative of $L$ w.r.t the coefficients from the $j\text{th}$ class $\bold w_j$




$$
\frac{\partial L}{\partial \bold w_j} = \sum_{i=1}^N (I_{ij} x_i -\sum_{j=1}^k I_{ij}\frac{e^{z_{ij}} x_i}{\sum_{m=1}^k e^{z_{i,m}}}) = \sum_{i=1}^N (I_{ij} - p_{i, j}) x_i
$$


The matrix form can be written as


$$
\frac{\partial L}{\partial \bold w_j} = \bold X^T (I_j - p_j)
$$


where $\bold X  = [x_1, x_2, ... x_N]^T $ is a $N \times D$ matrix and 


$$
I_j = \begin{bmatrix}I_{1j}\\\\ I_{2j}\\\\ ...\\\\ I_{nj} \end{bmatrix},
p_j = \begin{bmatrix}p_{1j}\\\\ p_{2j}\\\\ ...\\\\ p_{nj} \end{bmatrix}
$$


Therefore, 


$$
\frac{\partial L}{\partial \bold W} = \bold X^T (\bold I - \bold P) \in R^{D \times K}
$$


where $\bold I = [I_1, I_2, ..., I_k] \in R^{N \times K}, \bold P = [p_1, p_2, ..., p_k] \in R^{N \times K}$





## Comments



Logistic regression is a traditional classification model that often works effectively. However, it is problematic if data are linearly separable. If $w, b$ perfectly separates data linearly, so does cb, cw with $c \gt 0$, so there is no parameter vector that maximises likelihood.





## References

- https://towardsdatascience.com/why-not-mse-as-a-loss-function-for-logistic-regression-589816b5e03c

