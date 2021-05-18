---
title: "Singular Value Decomposition"
date: "2021-05-04"
description: "Singular Value Decomposition(SVD) is an important concept in Linear Algebra. Any matrix can be decomposed into three matrices using SVD. In machine learning, SVD is typically used to reduce dimensionality. Another popular dimension reduction technique is PCA. We will cover both of them in this post."
# tags: []
categories: [
    "Machine Learning",
]
series: ["Machine Learning"]
katex: true

---



Singular Value Decomposition(SVD) is an important concept in Linear Algebra. Any matrix can be decomposed into the multiplication of three matrices using SVD. In machine learning, SVD is typically used to reduce dimensionality. Another popular dimension reduction technique is PCA. We will cover both of them in this post.



<!--more-->



## Change of Basis



Suppose there is a point in the 2D space, how do you describe it? The common way is to use the Cartesian coordinate system, which is composed of two fixed perpendicular oriented axes, measured in the same unit of length. The two perpendicular axes are just a special set of vectors served as the basis of the 2D space. Actually, there are many other sets of vectors that can be the basis for the 2D space. For example, in Figure 1, the position of the red point is `(-4, 1)` when using the standard basis `(1, 0), (0, 1)` (colored in grey). If we change the basis to `(2, 1), (-1, 1)` (colored in blue), the position is `(-1, 2)`. 



![](/blog/post/images/change-basis-example.png "Figure 1: The same point with different coordinates in two different coordinate spaces")



From Figure 1, we can see that the absolute position of the red point always stay the same. However, the relative positions to the different bases are different. Mathematically, the red point can be described from the perspective of basis as follows,


$$
x = P_b[x]_b = c_1 \bold b_1 + c_2 \bold b_2 + ... + c_n \bold b_n
$$


$$
P_b = [\bold b_1, \bold b_2,  ... , \bold b_n ]
$$


$$
[x]_b = [c_1, c_2, ... c_n]
$$



where

- $\[x\]_b$ is a set of scalars, which represent the length of projection onto each axis of the current coordinate system
- $P_b$ is the corresponding basis of the current coordinate system



Let's plug the above point and the basis `(1, 0), (0, 1)`  (colored in grey) into the equation,



$$
P_b = [ (1, 0), (0, 1)]
$$

$$
[x]_b = (-4, 1)
$$

$$
x_b = -4 \begin{bmatrix}1\\\\ 0 \end{bmatrix} + 1 \begin{bmatrix}0\\\\ 1 \end{bmatrix} = \begin{bmatrix}-4\\\\ 1 \end{bmatrix}
$$



Let's do the same calculation with another basis (colored in blue),



$$
P_b = [ (2, 1), (-1, 1)]
$$

$$
[x]_b = (-1, 2)
$$

$$
x_b = -1 \begin{bmatrix}2\\\\ 1 \end{bmatrix} + 2 \begin{bmatrix}-1\\\\ 1 \end{bmatrix} = \begin{bmatrix}-4\\\\ 1 \end{bmatrix}
$$


As expected, they yield the same result. And the second one essentially changes the basis of $R^2$ from `(2,1),(-1,1)` to`(1,0),(0,1)`, which is the standard basis of $R^2$.



Actually, this example is a special case of the change of basis, where the new basis is the standard basis. More generally, $P_{c \larr b}$ is known as **the change of coordinate matrix from the old basis $b$ to the new basis $c$, which we are going to switch to** in $R^n$. 



Say we are in the basis $b$ and $[x]_b$ is known, the corresponding coodinates of $x$ under the new basis $c$ can be computed as follows,



$$
x_c = P_{c \larr b} x_b
$$



Since $P_{c \larr b}$ is invertible, we have


$$
(P_{c \larr b})^{-1}x_c = x_b
$$



which is the inverse operation of change of basis from $b$ to $c$. We can generalize this to any number of points and dimensions.



$$
A=US\\\\(D,N)= (D, M) \times (M, N)
$$




$$
U^{-1} A  = S\\\\(M, D) \times (D, N) = (M, N)
$$


where 

- $A$ is a $D\times N$ matrix with $D$ dimensions and $N$ points 
- $S$ is a  $M\times N$ matrix with $M$ dimensions and $N$ points described in a new vector space decided by another basis
- $U$ is the the change of coordinate matrix from $S$ to $A$
- $U^{-1}$ is the the change of coordinate matrix from $A$ to $S$



If we do some transformation for a point in the standard coordinate system, what're the new coordinates of the same point in another system? This problem can be solved by the following equation,



$$
x_s' = U^{-1}TUx_s
$$



where $T$ represents the transformation matrix. If $T=I$,  $x_s'$ is exactly $x_s$.



## SVD



### Linear Regression





### Principal Component





## PCA



Principal component analysis(PCA) is  often used to reduce dimentionality. The idea of PCA is to find directions along which data has the largest variation. The variation can be computed by projecting data onto that direction. Mathematically, we want to find a vector $v$ with $||v||=1$ to maximise


$$
\sigma^2 = \frac{1}{n-1}\sum_i^n(\bold v^T (\bold x_i - \bold \mu))^2
$$


There are two things to notice here:

- $v$ is an unit vector since we are care about the direction only
- data are centralized first for simple computation; centralizing data doesn't change the distribution of data



We can solve the above equation by introducing Lagrange multiplier


$$
L = \frac{1}{n-1}\sum_i^n(\bold v^T (\bold x_i - \bold \mu))^2 - \lambda (||v||^2 - 1)
$$



$$
= \frac{1}{n-1}\sum_i^n \bold v^T (\bold x_i - \bold \mu)(\bold x_i - \bold \mu)^T\bold v - \lambda (||v||^2 - 1)
$$

$$
= \bold v^T \bold C \bold v - \lambda (\bold  v^T\bold v - 1)
$$


where $\bold C$ is the covariance matrix of $X$. Then we take the derivative of $L$ w.r.t $\bold v$ , and then set it to $0$


$$
\frac{\partial L}{\partial \bold v} = 2(C \bold v - \lambda \bold v) = 0
$$


so $\bold v$ is the eigenvector of $\bold C$, and the variance along this direction is,


$$
\sigma^2 = \frac{1}{n-1}\sum_i^n(\bold v^T (\bold x_i - \bold \mu))^2 = \bold v^T \bold C \bold v = \lambda \bold v^T  \bold v = \lambda
$$


### Properties of Covariance Matrix



we know that the quadratic form of a vector and a matrix is defined as


$$
x^T M x
$$


Thus, the quadratic form of $C$ is


$$
x^TCx = x^T XX^Tx=u^Tu \ge 0
$$


which is also known as positive semi-definite. What does this mean? Well, it tells us that all eigenvalues of $C$ are greater than or equal to zero. Suppose $\mu$ is an eigenvector of $C$, since $\mu^TC\mu\ge0$ and $||\mu||>0$, then we have


$$
\mu^TC\mu = \mu^T \lambda \mu => \lambda = \frac{\mu^TC\mu }{||\mu||^2 } \ge 0
$$


### Geometry of PCA



Since $C$ is a $p \times p$ symmetric matrix, it can be orthogonally diagonalized as $C = P^TDP$, where $P$ is an orthogonal matrix and $D$ is a diagonal matrix. Thus, the above formula can also be written as follows, 


$$
\sum_i^n(\bold v^T (\bold x_i - \bold \mu))^2 = \sum_i^n \bold v^T (\bold x_i - \bold \mu)(\bold x_i - \bold \mu)^T\bold v = \bold v^T C \bold v
$$

$$
= \bold v^T P^TDP \bold v = y^TDy
$$




where $y = Pv$. Mathematically, $\bold v^T C \bold v$ and $y^TDy$ yield the same result.  They both compute the length of projection of each data point onto some vector, which is also the deviation of each data point from the original point along the direction determined by this vector, as shown in Figure 2. 



![](/blog/post/images/change-variable.png "Figure 2: Change of variable in $x^T A x$ (Introduction to Linear Algebra[1])")



Geometrically,  $P_{y \larr v}$ is the change coordinate matrix from $v$ to $y$, which finally transform the shape of the quadratic form $\bold v^TC\bold v$ into the standard position because $D$ is a diagonal matrix, such as in the figure below.



![](/blog/post/images/ellipse-standard.png "Figure 3: An ellipse and a hyperbola in standard position (Introduction to Linear Algebra[1])")



### Projection



For a data set with $p$ features, we want to reduce its dimension to $k$, there are a few steps to follow



- Construct a covariance matrix $C$ using centralized data
- Find  all the eigenvectors $v_i$ and eigenvalues $\lambda_i$ of $C$
- Keep $k$ eigenvectors with the $k$ largest eigenvalues (principal components)
- Project the original data into the space spanned by the principal components


$$
\bold  z = P(\bold  x - \bold \mu)
$$


where $z$ is our new inputs.



### Reconstruction



From the perspective of projection, the projection onto a vector $\bold v_j$ can be seen as approximating the inputs by


$$
\hat {\bold x_i} = \bold \mu + \sum_j^k z_j^i \bold v_j
$$

$$
z_j^i = \bold v_j^T(\bold x_i - \bold \mu)
$$


So our goal is to minimize the error 


$$
E[ ||\hat {\bold x_i} - \bold x_i ||^2] = \frac{1}{n} \sum_{n=1}^n ||\sum_{i=1}^k \bold u_i^T\bold x_n \bold u_i + \sum_{i=k+1}^p \bold u_i^T\bold x_n \bold u_i - \sum_{j=1}^k \bold v_j^T\bold x_n \bold v_j = \frac{1}{n} \sum_{n=1}^n \sum_{i=k+1}^p ||\bold u_i^T\bold x_n \bold u_i||^2
$$

$$
= \frac{1}{n}\sum_{n=1}^n\sum_{i=k+1}^p (\bold u_i^T\bold x_n) \bold u_i^T \cdot (\bold u_i^T\bold x_n) \bold u_i =\frac{1}{n} \sum_{n=1}^n\sum_{i=k+1}^p (\bold u_i^T\bold x_n)^2
$$

$$
= \frac{1}{n} \sum_{n=1}^n\sum_{i=k+1}^p \bold u_i^T\bold x_n \bold x_n^T \bold u_i= \sum_{i=k+1}^p \bold u_i^T (\frac{1}{n} \sum_{n=1}^n\bold x_n \bold x_n^T )\bold u_i
$$

$$
\sum_{i=k+1}^p \bold u_i^T S \bold u_i = \sum_{i=k+1}^p \lambda_i \bold u_i^T  \bold u_i = \sum_{i=k+1}^p \lambda_i 
$$




We can see that the error is exactly the sum of the eigenvalues in the directions that are discarded.





### PCA for images





## References



[1]	G. Strang, Introduction to Linear Algebra, 5th ed. Wellesley, MA: Wellesley-Cambridge Press, 2016.



