---
title: "Optimization"
date: "2021-11-12"
description: ""
# tags: []
categories: [
    "Machine Learning",
]
series: ["Machine Learning"]
katex: true
---



We've talked about many ML algorithms and DL architectures so far, but how to find the optimal parameters? Mathematically, the process of minimizing the objective functions is called optimization. It's a bit different from the optimization in DL — the global minimum point does not always achieve the best generalization performance. After all, we are minimizing the training error. Besides, the loss function typically is very complex, and there is no analytical solution to it. In this case, we have only one choice — optimization algorithms.



<!--more-->



## Newton's Method



Bascially, Newton's method is a root-finding method, for example, the root of $f(x) = x^2 - 8$. Assume that we start from the point $x_0$, and the intersection of the tangent line and the x-axis is the point $(x_1, 0)$, then we have


$$
\hat f(x_1) = f(x_0) + (x_1 - x_0) f'(x_0) 
$$


let $\hat  f(x_1)=0$, 


$$
x_1 = x_0 - \frac{ f(x_0)}{f'(x_0)}
$$


Similarly, we can repeat the above operation starting from $x_1$ until we think the precision is sufficient. Typically, we generalize these steps as follows,


$$
x_{n+1} = x_n - \frac{f(x_n)}{f'(x_n)}
$$
Let's start from $x_0 = 4$, the results are shown below.

```python
x_0 =4 
x_1 = 3
x_2 = 2.833333
x_3 = 2.828431
```



We see that $x_3$ is very close to the actual value $2\sqrt2$, so we can stop. 



Furthermore, we can also use this method to find the solution to $f'(x) = 0$. In this case, we end up with


$$
x_{n+1}  = x - \frac{f'(x)}{f''(x)}
$$
Actually, it is Newton's second-order strategy (We'll see later).



### Taylor Expression

In fact, the above linear approximation is the first order taylor expression, and it holds when $x_1 - x_0$ is really small. For a larger $x_1-x_0$, we can use the second-order taylor expression. Recall that Taylor expanding around $x^*$ is 


$$
f(x) = f(x^\*) + (x - x^\*)f'(x^\*) + \frac{1}{2}(x-x^\*)^2f''(x^\*) + ...
$$
If $x - x^\*$ is sufficiently small, the higher order terms are ignored. The derivative of $f(x)$ w.r.t $x - x^\*$ is
$$
f'(x\*) - (x - x^\*)f''(x^\*) = 0
$$


Thus, 


$$
x^* = x - \frac{f'(x^\*)}{f''(x^\*)}
$$


We see that Newton's method would be very efficient when we are close to the minimum.



## Gradient Descent



### Batch



### Stochastic



## Momentum





## Adam





## AdaGrad





## RMSProp





## References

- [Newton's Method](https://opentextbc.ca/calculusv1openstax/chapter/newtons-method/)
- [Adam: A Method for stochastic Opimization](https://arxiv.org/abs/1412.6980)

