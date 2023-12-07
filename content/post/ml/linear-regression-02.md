---
title: "Linear Regression 02"
date: "2021-04-17"
description: "In the previous post, we talked about simple linear regression. However, we only considered one predictor. It's quite common to have mulitple predictors for real-world problems."
# tags: []
categories: [
    "Machine Learning",
]

katex: true

---



In [the previous post](https://ixiaopan.github.io/blog/post/linear-regression-01/), we talked about simple linear regression. However, we only considered one predictor. It's quite common to have multiple predictors for real-world problems. For example, if we want to predict car prices, we should consider many factors like car sizes, manufacturers and fuel types. The simple linear regression is not suitable for this case. Therefore, we need to extend it to accommodate the multiple predictors.



<!--more-->



## Multiple Linear Regression



Suppose we have a data set with the size of $n$,  and each data point has $d$ dimensions. Then the input data is denoted by  $X \in R^{n \times d}$, and the parameters and targets are denoted by $\bold w \in R^d$, $\bold y \in R^n$ respectively. Thus, the loss function can be written by the following equation:


$$
L = \sum_i^{n} (\bold x_i \bold w - \bold y_i)^2 = (\bold X \bold w - \bold y)^T(\bold X \bold w - \bold y)
$$

$$
= \bold w^T\bold X^T \bold X \bold w  - \bold y^T \bold X \bold w - \bold w^T \bold X^T \bold y + \bold y^T \bold y
$$

$$
= \bold w^T\bold X^T \bold X \bold w  - 2 \bold w^T \bold X^T \bold y + \bold y^T \bold y
$$




Then we take the derivative of $L$ with respect to $\bold w$ as simple linear regression before. Well, we need to know a little bit about the matrix calculus


$$
\frac{\partial}{\partial \bold x} \bold x^TA\bold x = (A + A^T)\bold x
$$

$$
\frac{\partial}{\partial \bold x} A^T \bold x = A
$$




The gradient of $L$ can be seen easily


$$
\frac{\partial}{\partial \bold x } L = (2\bold X^T\bold X)\bold w - 2 \bold X^T\bold y
$$




Setting this gradient to zero,


$$
\bold w= (\bold X^T\bold X)^{-1} \bold X^T \bold y
$$


However, this equation is unlikely to work if  $\bold X^T\bold X$ is not invertible (singular), such as the number of features are more than the number of observations ($n < d$). One way to solve this equation is to use SVD.



### Pseudoinverse

SVD technique can decompose any matrix $A$ into the multiplication of three matrices, i.e.  $U\Sigma V^T$. Thus the above equation can be written in the following form,



$$
\bold w = A^+y
$$



$$
A^+ = (\bold X^T\bold X)^{-1} \bold X^T = V\Sigma^{-1}U^T
$$



In practice, the algorithm will set the elements of $\Sigma$ that less than a smaller threshold to zero, then take the inverse of all nozero values, and finally transpose the resulting matrix i.e. $(U\Sigma V^T)^{-1}$



> When A has more columns than rows, then solving a linear equation using the pseudoinverse provides one of the many possible solutions. Speciﬁcally, it provides the solution $x = A^+ y$ with minimal Euclidean norm $||x||_ 2$ among all possible solutions.
>
> When A has more rows than columns, it is possible for there to be no solution. In this case, using the pseudoinverse gives us the x for which Ax is as close as possible to y in terms of Euclidean norm $|| Ax − y ||_ 2$ .
>
> — Deep Learning, p44



### Comparison of algorithms

Below are common methods to estimate LR.

- Analytical solution calculates $(X^TX)^{-1}$ directly, which is a $d \times d$ matrix, so larger $d$ causes slow computation
- SVD is a dimension reduction method, again, smaller $d$  is fast
- GD-based methods tend to be affected by the scale of features

|                        | Large N | Large D | Scaling Required |
| ---------------------- | ------- | ------- | ---------------- |
| Equation               | fast    | slow    | No               |
| SVD                    | fast    | slow    | No               |
| Batch Gradient Descent | slow    | fast    | Yes              |
| SGD                    | fast    | fast    | Yes              |



## Probabilistic Interpretation



### Assumption

Linear Regression works based on the following assumptions

- Y has a linear relationship to X. To check this, we can plot a scatterplot
- Y values are independent
- For each $x_i$, there is an error $\epsilon_i$ following Gaussian distribution $\epsilon_i \sim N(0, \sigma^2)$



### Residual Plot

A residual plot shown in Figure 1 is a graph that shows the relationship between residuals and $X$. If the residuals are randomly placed around the x axis, a linear model is appropriate. Otherwise, we would transform data to satisfy linearity.



![](/blog/post/images/residual-plot.png#full "Figure 1: Residual plots. [Source from [2]]")



### MLE

We have found the parameters by minimizing the loss, but now we are going to use another method to derive the same result, which is known as maximum likelihood estimation(MLE).



The basic idea of MLE is that if the data were generated from some model, then what's the parameters of the model were most likely to make this happen? In other words, we are finding the parameters that maximize the probability of the data $D$ that we've seen.



Suppose we have a data set of inputs $X={x^{(1)}, x^{(2)}, ..., x^{(N)}}$ and corresponding target variables ${y_1, y_2, .., y_N}$ with a Gaussian noise $\epsilon_i \sim N(0, \sigma^2)$, we obtain


$$
y'_i \sim N(b + ax_i, \sigma^2)
$$
 



Next we construct the likelihood of all data points,

$$
L(\theta|D) = \prod_{n=1}^N p(y_i|x_i, a, b, \sigma^2)
$$

For a target value $y_i$, the probability of $y_i$ given the parameters $a, b, \sigma$ is 


$$
p(y_i|x_i, a, b, \sigma^2) = \frac{1}{\sqrt{2\pi\sigma}} e^{-\frac{(y_i - b - ax_i)^2}{2\sigma^2}}
$$
Our goal is to find the appropriate parameters $a, b, \sigma$ to maximise all the likelihood of $y_i$. Usually, we take the log likelihood to make computation more simpler,



$$
In(L(\theta|D)) =\sum_i^n In(\frac{1}{\sqrt{2\pi\sigma}} e^{-\frac{(y_i - b - ax_i)^2}{2\sigma^2}}) \\\\ = \frac{N}{2}In\sigma - \frac{N}{2}In2\pi - \frac{1}{2\sigma^2}\sum_{n=1}^N(y_i - b - ax_i)^2
$$


From above equation, we can see that maximizing the likelihood is equivalent to minimizing the sum of squared error.



## Geometry of LR



In this section, we will look at the geometry of the linear regression. In $N$-dimensional space whose axes are the values of $y_1, y_2, ..., y_n$ , the least-squares solution is obtained by finding the orthogonal projection of the target vector $y$ onto the subspace spanned by the columns of $X$.



![Geometry interpretation of the least-squares solution](/blog/post/images/geometry-linear-regression.png#full "Figure 1: Geometry interpretation of the least-squares solution (PRML 2006)")





From the following matrix form, we can see that the predicted value $\bold y'$ lies the column space of $X$. If the true target value $\bold y$ also lies in this space, then the loss of linear regression is zero, which is never the case in real life.




$$
\displaystyle{\bold y' = \bold X \bold w = \begin{bmatrix}1&x_{11} & x_{12} & ... & x_{1d}\\\\ 1&x_{21} & x_{22} & ... & x_{2d}\\\\ ... \\\\ 1&x_{n1} & x_{n2} & ... & x_{nd} \end{bmatrix}
\begin{bmatrix}w_0\\\\ w_1\\\\ w_2\\\\ ...\\\\ w_d \end{bmatrix}
}
$$



## Weighted Least Square



We assume $\epsilon_i$ share the same variance (homoskedasticity), however, it is not always the case (heteroskedasticity). In this case, we add a weight to each residual, as shown below


$$
L_{min} = \sum w_i(y_i - ax_i - b)^2
$$


We can see that large noise will be punished heavily, i.e. $w_i$ tends to small. The idea is that we'd like to pay more attention to data with little noise for estimation rather than to concerntrate all noisy part of the data.



## Non-linear Data



### Transformation

| Model             | Predicted Value             |
| ----------------- | --------------------------- |
| Logarithmic model | $y' = a \text{log} (x) + b$ |
| Reciprocal model  | $y' = 1/(ax + b)$           |
| Quadratic model   | $ y' = (ax + b)^2$          |
| Exponential model | $y' = e^{ax+b}$             |



### Fitting Polynomials



### Model Selection

AIC, Akaike's Information Criterion is an estimator for the K-L divergence.


$$
AIC = 2K - 2 \text{log}(L)
$$


where $K$ is the number of predictors and $L$ is the maximised likelihood value. The goal is to find the minimum AIC, so 

- $K$ will punish models with many predictors
- $L$ rewards good fit models



## References



[1] C. Bishop, *Pattern Recognition and Machine Learning*. 2006.

[2] "Interaction Effects in Regression", *Stattrek.com*, 2021. [Online]. Available: https://stattrek.com/multiple-regression/interaction.aspx?tutorial=reg. [Accessed: 11- May- 2021].

