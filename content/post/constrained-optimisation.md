---
title: "Constrained Optimisation"
date: "2021-06-03"
description: "When I first learned machine learning, I was scared by the complicated formulas. Actually, I spent much time going over subjects like Linear Algebra and Calculus since I'd already forgotten them. But with time, I feel more and more confident in understanding them, though they sometimes still confuse me. Anyway, in my opinion, there is no need to know every detail about each equation, after all, we are not mathematicians. Instead, learning how to use these math formulas to solve real-wold problems is the key."
# tags: []
categories: [
    "Machine Learning",
]
series: ["Machine Learning"]
katex: true

---



When I first learned machine learning, I was scared by the complicated formulas. Actually, I spent much time going over subjects like Linear Algebra and Calculus since I'd already forgotten them. But with time, I feel more and more confident in understanding them, though they sometimes still confuse me. Anyway, in my opinion, there is no need to know every detail about each equation, after all, we are not mathematicians. Instead, learning how to use these math formulas to solve real-wold problems is the key. 



<!--more-->



As we all know, the main effort in machine learning is to find a loss function and optimise it, i.e. find the miminum or maximum point, and this is the question of optimisation. However, we may only find local optimisation because of some constraints. Even without constraints, there is still chance that we can reach local optimisation only. From this, we can see that there are two main situations we need to consider: unconstrained optimisation and constrained optimisation. Furthermore, constrained optimisation can be divided into two parts: euqality constraints and inequality constraints. And today we will talk about all of them in detail.



## Unconstrained Optimisation



TODO





## Equality Constraints



Suppose we want to minimise a function $f(x)$ subject to an equality constraint, $g(x) = 0$. This can be solved by introducing Lagrange multiplier $\alpha$ 


$$
\mathcal{L}(x, \alpha) = f(x) - \alpha g(x)
$$


Then set the gradient of $\mathcal{L}$ equal to the zero vector


$$
\nabla_x \mathcal{L} = \nabla_x f(x) - \alpha \nabla_xg(x) = 0
$$

$$
\frac{\partial \mathcal L}{\partial \alpha}  = -g(x) = 0
$$


Finally, solving the above equations will give us the minimum point we are seeking.

 

### Multiple Constraints



If we have multiple constraints, then multiple Lagrange multipliers are introduced,


$$
\mathcal{L}(x, \alpha) = f(x) - \sum_i^m \alpha_i g_i(x)
$$
And the corresponding solutions are given by,


$$
\nabla_x f(x) = \sum_i^m \alpha_i \nabla_xg_i(x)
$$

$$
\frac{\partial \mathcal L}{\partial \alpha_i}  = -g_i(x) = 0
$$


### Lagrange 

But what's the rationale behind these formulas? Though we are not required to know everything, basic understanding of Lagrange is still needed. Remember that the question is to find a point $x^*$ that satisfies both $g(x^\*) = 0$ and $f(x^\*) =m $, where $m$ is the minimum value of $f(x)$ given the constraint.

 The line represented by $f(x) = c$ is known as a contour line. Since $f(x)$ can have many values, we can plot many contour lines with equal intervals between lines in ascending or descending order from inner to outer. Graphically, we want to find a point that lies on the line of $g(x) = 0$ and the line of $f(x)$ with the minimum value simultaneously as shown in Figure 1. The green point is the point we are looking for.



![](/blog/post/images/lagrange.png "Figure 1: Illustration of equality constraints")



But we still need to figure out equations to calculate the position of the gree point. Let's start from $x_0$, the magenta point in Figure 1. Mathematically, the above process of finding $x^*$ can be described as follows,


$$
f(x_0 + \delta x) < f(x_0)
$$

$$
g(x_0) = g(x_0 + \delta x) = 0
$$


So in which direction should we move at $x_0$? With the aid of Taylor expansion, we have
$$
g(x_0 + \delta x) = g(x_1) = g(x_0) + (x_1 - x_0)^T\nabla_xg(x_0) + \frac{1}{2} (x_1 - x_0)^T H (x_1 - x_0)
$$

$$
= g(x_0) + (x_1 - x_0)^T\nabla_xg(x_0) + O(||x_1 - x_0||^2)
$$


where $x_1 = x_0 + \delta x$ and $H$ is a matrix of second derivative of $g(x)$ known as Hessian. The third term tends to be zero if $\delta x$ is small enough, then we are left with


$$
g(x_0 + \delta x) = g(x_0) + (x_1 - x_0)^T\nabla_xg(x_0) = g(x_0)
$$


Thus,


$$
(x_1 - x_0)^T\nabla_xg(x_0) = 0
$$


which means that the moving direction from $x_0$ should be perpedicular to $\nabla_xg(x_0)$. But we are not done, because not all $(\delta x = x_1 - x_0)$ point to the right direction along which $f(x)$ decreases. In order to ensure this, we require that  $\delta x$ must satisfy


$$
(x_1 - x_0)^T ( - \nabla_x f(x)) > 0
$$


where $- \nabla_x f(x)$ indicates the descent direction. Therefore, as long as the value of the dot product is greater than zero, the moving of point will continue unless the value becomes zero. If so, it means that the direction of $\nabla_x f(x)$ is parallel to $\nabla_x g(x)$, i.e.


$$
-\nabla_x f(x) = \lambda \nabla_x g(x)
$$


We can rewrite this by replacing $\lambda$ with $\alpha = -\lambda $ 


$$
\nabla_x f(x) = \alpha \nabla_x g(x)
$$


Finally, we come to the method of Lagrange multipliers.



## Inequality Constraints



Now we consider another situation where we have inequality constraints, i.e. $g(x) \ge 0$. It looks a little complicated, but if we think about it for a while, we can find that only two things could happen, as shown in Figure 2,

- either the optimum point satisfies $g(x) \gt 0$ or
- the (local) optimum point lies on the boundary, $g(x) = 0$ 



![](/blog/post/images/inequality.png#full "Figure 2: Two possible outcomes of inequality constraints")



Again, we use Lagrange to solve it, 


$$
\mathcal{L}(x, \alpha) = f(x) - \alpha g(x)
$$


In the first case where the optimum point lies in the interior of the constraint, i.e. $g(x) > 0$, which is filled in red in Figure 2 a). We set $\alpha = 0$, which means the constrains has no influence on $f(x)$. 

As for the second case, it is exactly the same as the equality constraints, i.e.


$$
\nabla_x f(x) = \alpha \nabla_xg(x)
$$


but with an additional constraint $\alpha > 0$. So why do we set $\alpha > 0$ here? 

### $\alpha \ge 0$



Visually, it can be seen From Figure 2 b) that both the magenta and blue points seem to be the right point we are seeking. But in fact, only the bule one is in a lower position. And we find that $\nabla_x f(x) $ and $\nabla_x g(x)$ point to the same direction at the blue point. Thus, $\alpha$ is positive. 

In theory, the area defined by $g(x) > 0$ is known as feasible region, i.e. any value of $x$ inside this region is valid. Besides, We can see that $\nabla_x f(x)$ (colored in blue) points in towards the interior of the region. Conversely, the negative gradient $-\nabla_x f(x)$, which is the direction along which $f(x)$ decreases the most quickly, points away from the feasible region(I don't plot it). 

If we are at a point where $-\nabla_x f(x)$ points to the feasible region, it means that a point with smaller value of $f(x)$ can be found in the feasible region, which is contradictory to the assumption that we can only move along the boundary of the region. In other words, this is not the optimal point. 

If $-\nabla_x f(x)$ at some point points to the exterior of the feasible region, then we are in the right position because the outer of the feasible region is invalid and we cannot move forward any further.

It is noticeable that $\nabla_x g(x)$ points in towards the feasible region. Therefore, we conclude that $\nabla_x f(x)$ and $\nabla_x g(x)$ have the same direction. Thus, $\alpha$ is positive. 



From above, we can also draw another conclusion shown below


$$
\alpha \nabla_x g(x) = 0
$$




### KKT Conditions



To sum up, we want to minimise $f(x)$ subject $g(x) \ge 0$, and the Lagrangian function is defined as follows,


$$
\mathcal{L}(x, \alpha) = f(x) - \alpha g(x)
$$


Then we can find a local mininum $x^*$ s.t.

- $\nabla_x f(x) = \alpha \nabla_xg(x)$
- $\alpha \ge 0$
  - $\alpha = 0$, the solution is in the interior or
  - $\alpha \gt 0$ and $g(x) = 0$, i.e. the solution is on the boundary
- $\alpha \nabla_x g(x) = 0$



These are the Karush-Kuhn-Tucker (KKT) conditions.





### Many Inequalities



Once again, if we have many ineuqality constraints, then Lagrangian function is given by, 


$$
\mathcal{L}(x, \alpha) = f(x) - \sum_i^m \alpha_i g_i(x)
$$


And the corresponding solutions are given by,


$$
\nabla_x f(x) = \sum_i^m \alpha_i \nabla_xg_i(x)
$$


plus the constraints that 

- either $\alpha_i = 0$ or
- $\alpha_i \gt 0$ and $g_i(x) = 0$



## Duality





## Convex





## Jensen’s inequality





## References

- https://www.csc.kth.se/utbildning/kth/kurser/DD3364/Lectures/KKT.pdf

