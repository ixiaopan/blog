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



As we all know, the main effort in machine learning is to find a loss function and optimise it, i.e. find the miminum or maximum point, and this is the question of optimisation. However, we may only find local optimisation because of some constraints. Even without constraints, there is still chance that we can reach local optimisation only. From this, we can see that there are two main situations we need to consider: unconstrained optimisation and constrained optimisation. Furthermore, constrained optimisation can be divided into two parts: euqality constraints and inequality constraints. And we will focus on constrained optimisation in this article.



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

In theory, if we are at a point where $-\nabla_x f(x)$ points to the feasible region, which is the area defined by $g(x) > 0$, i.e. any value of $x$ inside this region is valid, it means that a point with a smaller value of $f(x)$ could be found in the feasible region. But it is contradictory to the assumption that we can only move along the boundary of the region. In other words, this is not the optimal point. 

If $-\nabla_x f(x)$ at some point points to the exterior of the feasible region, then we are in the right position because the outer of the feasible region is invalid and we cannot move forward any further(we are already on the border of the region).

It is noticeable that $\nabla_x g(x)$ points in towards the feasible region. Therefore, we conclude that $\nabla_x f(x)$ and $\nabla_x g(x)$ have the same direction. Thus, $\alpha$ is positive. 



From above, we can also draw another conclusion shown below


$$
\alpha \nabla_x g(x) = 0
$$




### KKT Conditions



Putting it together, we want to minimise $f(x)$ subject $g(x) \ge 0$, and the Lagrangian function is defined as follows,


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
- $\alpha_i \nabla_x g_i(x) = 0$



## Duality



Let's revisit the above problem again. Consider minimising a function $f(x)$ subject to $g(x) \ge 0$, the lagrangian function is given by


$$
\mathcal{L}(x, \alpha) = f(x) - \alpha g(x)
$$


### Primal Problem



Now consider maximising $\mathcal{L}(x, \alpha)$ w.r.t $\alpha$, 


$$
\text{max}_\alpha  \mathcal{L}(x, \alpha) = \begin{cases} f(x) & \text{if $g(x) \ge 0$} \\\\ \infin & \text{otherwise}\end{cases}
$$




- For any $x'$ that satisfies the constraint $g(x) \ge 0$, we conclude that

  
  $$
  f(x') \ge \mathcal{L}(x', \alpha)
  $$
  

  i.e. the upper bound of  $\mathcal{L}(x, \alpha)$ is $f(x)$. Thus, the maximum value of $\mathcal{L}$ we can obtain is to set $\alpha = 0$ 

- On the contratry, if the constraint is not satisfied, then there exists some $x'$ that satisfies $g(x) \lt 0$. If so, then we can make $\mathcal{L}$ infinite by taking $\alpha \rarr \infin$.



Next we take the minimum of the maximum of $\mathcal{L}$, i.e.


$$
\text{min}\_\text{x} \text{max}_\alpha \mathcal{L}(x, \alpha)
$$


which is equivalent to the problem of minimising $f(x)$ subject to $g(x) \ge 0$. We call the original problem as **primal problem**.



### Dual Problem

Yet we still don't find a solution to $x$. How about reversing the order of max and min like the below formula?


$$
\text{max}_\alpha \text{min}\_\text{x} \mathcal{L}(x, \alpha)
$$


We solve $\text{min}\_\text{x} \mathcal{L}(x, \alpha)$ using the method of Lagrange, and find that $x$ is a function of $\alpha$. Then we plug $x$ into $ \mathcal{L}(x, \alpha)$ and obtain a new function of $\alpha$, say $h(\alpha)$. So the problem becomes to maximise $h(\alpha$), also known as **dual problem**,


$$
\text{max}_\alpha  h(\alpha)
$$


However, is the solution to dual problem the same as the primal problem? Why do we bother solving a dual problem rather than the original problem?



Since $\alpha \ge 0$  and $g(x) \ge 0$, we have,


$$
\text{min}\_\text{x} \mathcal{L}(x, \alpha) \le \text{min}_x f(x) = p^*
$$

$$
d^* = \text{max}_\alpha \text{min}\_\text{x} \mathcal{L}(x, \alpha) \le p^*
$$


where $p^\*$ and $d^\*$ are the optima of the primal and dual problem respectively. 

It can be seen that the solution of dual problem gives us a lower bound on the primal  problem. If possible, we can also have the same solution.

The reason why we solve the dual problem is that the primal problem works in a feature space that may have high dimensions while the dual problems depends on the number of constraints, which is much smaller than the dimensitionality of $x$.



### Linear Programming

In linear programming, we minimise a linear function $c^Tx$ subject to a series of linear constraints $g(x) = Mx - b \ge 0$. The primal problem is described as follows,


$$
\mathcal{L} (x, \alpha) =  c^Tx - \alpha^T(Mx - b)
$$

$$
\text{minimise } c^T x
$$

$$
\text{subject to } Mx \ge b
$$


and the dual problem is defined as,


$$
\mathcal{L} (x, \alpha) =  b^T\alpha - x^T(M^T \alpha - c)
$$

$$
\text{maximise } b^T \alpha
$$

$$
\text{subject to } M^T \alpha\le c
$$


Let's see and example.

Primal problem


$$
\text{minimise } z = 15x_1 + 12x_2\\\\ \text{subject to } x_1 + 2x_2 \ge 3, 2x_1 - 4 x_2 \ge 5
$$


Dual problem


$$
\text{maximise } w = 3y_1 + 5y_2\\\\ \text{subject to } y_1 + 2y_2 \le 15, 2y_1 - 4 y_2 \le 12
$$




### Quadratic Programming

In quadratic programming, we minimise a quadratic function $x^TQx$ subject to a series of linear constraints $g(x) = Mx - b \ge 0$. The primal problem is described as follows,


$$
\text{max}_\alpha  \text{min}\_\text{x} \mathcal{L} (x, \alpha) =  x^TQx - \alpha^T(Mx - b)
$$


Using the method of Lagrange, we have $x^* = \frac{1}{2}Q^{-1}M^T\alpha$, and we substitue it into $\mathcal{L}$


$$
\text{max}_\alpha  -\frac{1}{4} \alpha^TMQ^{-1}M^T\alpha + \alpha^Tb
$$


## Convexity



### Quadratic Form



> Quadratic form is a polynominal function with terms all of degree of two. For example, $4x^2 + 2xy - 3y^2$ 
>
>  —Wikipedia.



For simplicity, we often write it in matrix notation, as shown below


$$
Q(\bold  x) = \bold x^T \bold  M \bold x = \sum_{i,j}^d \bold  M_{ij} \bold x_i \bold x_j
$$


In this example $4x^2 + 2xy - 3y^2$, the quadratic form is given by,


$$
Q(\bold x) = \bold x^TM\bold x = \displaystyle{\begin{bmatrix}x&y\end{bmatrix}
\begin{bmatrix}4&2\\\\0&3 \end{bmatrix}\begin{bmatrix}x\\\\y\end{bmatrix}
}
$$


From this we can see that quadratic form is a mapping from $R^d$ to $R$, so its value could be one of the following cases,

- $Q(\bold x) \gt0$, 
  - Q is positive definite
- $Q(\bold x) \ge 0$, 
  - Q is positive semi-definite
- $Q(\bold x) \lt 0$ , 
  - Q is negative definite
- $Q(\bold x) \le 0$, 
  - Q is negative semi-definite
- $Q(\bold x)$ could be both positive and negative
  - Q is idefinite



Furthermore, quadratic form can also be characterised in terms of eigenvalues. Let $A$ be an $n \times n$ symmetric matrix. Then a quadratic form $x^TAx$ is:

- positive definite if and only if the eigenvalues of $A$ are positive,
- negative definite if and only if the eigenvalues of $A$ are negative, 
- indefinite if and only if A has bothe positive and negative eigenvalues



Here are some proofs. First, we should know that any two eigenvectors from different eigenspaces of a symmetric matrix are orthogonal and any symmetric matrix can be orthogonally diagonalizable. Let $\bold P = [\bold v1, \bold v2, ..., \bold v_n]$ be eigenvectors that correspond to different eigenvalues $\Lambda= \lambda_1, \lambda_2, ..., \lambda_n$ of a symmetric matrix $A$. To show that $v_1 \cdot v_2 = 0$, compute


$$
\lambda_1 v_1 \cdot v_2 = (A v_1)^T v_2 = v_1^T A v_2 = \lambda_2 v_1^Tv_2
$$

$$
(\lambda_1 - \lambda_2) v_1^T v_2 = 0
$$


But $ \lambda_1 \ne \lambda_2$, so $v1 \cdot v2=0$. Furthermore, $\bold P ^T \bold P = I$, so $P^{-1} = P^T$. Then we have


$$
AP = PD\\\\A = PDP^{-1} = PDP^T
$$


 The quadratic form of $A$ can be written as follows,


$$
x^T A x= x^T PDP^Tx = (P^Tx)^T D (P^Tx) = y^TDy = \lambda_1y_1^2 + \lambda_2y_2^2 + ... + \lambda_ny_n^2
$$


If the eigenvalues of $A$ are positive, then $x^TAx > 0$ and $A$ is positive definite. On the other hand, if $A$ is positive definite, then $x^TAx > 0$ for any $x \ne 0$. If we substitute $x$ with an eigenvector $v_1$, we have


$$
v_1^T A v_1 = v_1^T \lambda v_1 = \lambda v_1^Tv_1 = \lambda ||v_1||^2 \gt 0
$$


Thus, $\lambda  > 0$.





### Convex Set

Before we talk about convex function, let's start with convex region and convex set. 

A region $R$  is said to be a convex region if any two points $x$ and $y$ in that region plus any $a \in [0, 1]$ satisfy,


$$
z = a x + ( 1 - a )y \in R
$$


![](/blog/post/images/convex-region.png "Figure 3: Convex region and non-convex region")



Similarly, for any set of points $S$, if for any two points $x, y \in S$ and any $a \in [0, 1]$ satisfy
$$
z = a x + ( 1 - a )y \in S
$$
then $S$ is a convex set. We can prove that the set of positive semi-definite matrices form a convex set. Let $A_1, A_2 \in S$, compute


$$
x^T z x = x^T (a A_1 + ( 1 - a)A_2)x = x^TaA_1x + x^T ( 1-a) A_2 x \ge 0
$$


Thus, $z$ is positive semi-definite and $z \in S$.



### Convex Function





## Jensen’s inequality





## References

- https://www.csc.kth.se/utbildning/kth/kurser/DD3364/Lectures/KKT.pdf
- https://www-cs.stanford.edu/people/davidknowles/lagrangian_duality.pdf



