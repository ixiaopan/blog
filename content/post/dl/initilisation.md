---
title: "Deep Learning - Weight Initialization"
date: "2021-11-24"
description: ""
# tags: []
categories: [
    "Deep Learning",
]
katex: true
---



The goal of machine learning is to find the best parameters. To train the model, we must start from some values. So, what do the initial parameters look like? Can we set all of them to the same value? Or we assign random values to the weights? In this post, we will talk about another important aspect of learning — weight initialization.

<!--more-->



## Constant Values



What if we set all parameters to the same values, say $c$? Assuming that all the biases are $0$, the $m_{th}$ hidden unit in the first layer is


$$
z_{1m} = \sigma (cx_1+cx_2+...+cx_n)
$$
 and the $k_{th}$ hidden unit in the second layer is
$$
z_{2k} = \sigma ( cz_{11} + cz_{12} + .. cz_{1m}) = \sigma ( m \cdot c \cdot z_{1m})
$$
We find that the hidden units in the same layer are the same (the same structure, you can imagine that they are copies of one node), ultimately leading to the same gradient for weights belonging to the same hidden layer. Since both the weights and the gradients are the same, we learn nothing.



## What kind of weights



- too small $\rarr$ gradient vanishing, slow learning
- too large $\rarr$ gradient exploding



So what kind of values should we use to initialize weights? According to [1], there are two rules of thumb:



> - The mean of the activations should be zero
>
> - The variance of the activations should stay the same across every layer



Mathematically, they can be expressed as 


$$
\text {E}(z_{n-1}) = \text {E}(z_n) = 0 \\\\  \text {Var} (z_{n-1}) = \text {Var}(z_n)
$$


where $n$ indicates the $n_{th}$ hidden layer and $z_n$ is the random variable representing the hidden node. Under the above two assumptions, it would avoid gradient exploding and gradient vanishing. 

But why should the variance stay the same?  Well, we find an explanation from [2]. Suppose 

- all the inputs are $x_n = 1$
- and all the weights follows the Gaussian distribution with a mean of $0$ and a variance of 1, that is $w_i \sim N(0, 1)$



the hidden nodes in the first layer is given as follows (we omit the activation function)
$$
h_{1m} = w_1 x_1 + w_2x_2 + .. w_nx_n = w_1 + w_2 + .. w_n
$$


Now, let's compute the mean and variance of this new variable $h_{1m}$,


$$
E(h_{1m}) = E(w_1 + w_2 + .. +w_n) = E(w_1) + E(w_2) + ... +E(w_n) = 0 \\\\ \text{Var}(h_{1m}) = \text{Var}(w_1 + w_2 + .. +w_n) = \text{Var}(w_1) + \text{Var}(w_2) + ... + \text{Var}(w_n) = n
$$


Generally, $n$ is quite large, say 100, then the standard deviation is $10$. This means, if we randomly obtain a hidden node, it will lie 10 units far away from the mean. After the activation function, say Sigmoid, the result is close to 1 or 0. As we discussed earlier, this will lead to gradient vanishing. Thus, to avoid this, we should shrink the variance.



## Xavier Initializaiton



Xavier Initialization works on the assumption that the variance of the activations stays the same across layers, which lead to the following initialization.


$$
W_n \sim N(0, \frac{1}{m_{in}})
$$
where $m_{in}$ is the number of the input nodes. Why? Where does this value $\frac{1}{m_{in}}$ come from? Assuming again that all the biases are $0$, we're tring to find the variance of $W_n$ using the following equation.


$$
\text {Var} (z_{n+1} )= \text {Var} (z_{n} ) = \text {Var} ( \text{tanh} (\sum_{i=1}^m w_i z_i ))
$$


Since we're just starting training, the values are so small that they are in the liner regime of $tanh$. Thus, we have
$$
z = \text {tanh(y)} = y
$$


With this, we obtain


$$
\text {Var} (z_{n} ) = \text {Var} ( \text{tanh} (\sum_{i=1}^m w_i z_i )) = \text {Var} (\sum_{i=1}^m w_i z_i ) = \sum_{i=1}^m \text {Var} ( w_i z_i )
$$


Now we assume that 

- $w_i$ is i.i.d
- $z_i$ is i.i.d
- $w_i$ is independent of $x_i$
- $E(z_i) =0$
- $E(w_i) = 0$



then we can decompose the variance of a product into a product of variances, as shown below


$$
\text {Var} (z_{n+1} )= \text {Var} (z_{n} ) = \sum_{i=1}^m \text {Var} ( w_i z_i ) \\\\ = E^2[w_i] Var(z_i) + Var(w_i)E^2[z_i] + Var(w_i)Var(z_i) \\\\ = \sum_{i=1}^m Var(w_i)Var(z_i) = m \text{Var} (W_n) \text{Var} (z_n)
$$

We obtain


$$
\text{Var} (W_n) = \frac{1}{m}
$$


One thing worth noting is that the initial suggested variance was $\frac{2}{m_{in} + m_{out}}$, which is the harmonic mean of the input and the output,


$$
 2 \* (\frac{1}{m_{in}}\* \frac{1}{m_{in}} ) / (\frac{1}{m_{in}} + \frac{1}{m_{out}}) 
$$




If we're using ReLu, the suggested value is $\frac{2}{m}$, yep, multiplying by 2 the variance of Xavier, also known as He initialization.




## References

[1] [Katanforoosh & Kunin, "Initializing neural networks", deeplearning.ai, 2019.](https://www.deeplearning.ai/ai-notes/initialization/)

[2] [Weight Initialization Explained | A Way To Reduce The Vanishing Gradient Problem](https://deeplizard.com/learn/video/8krd5qKVw-Q)





