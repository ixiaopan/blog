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



Singular Value Decomposition(SVD) is an important concept in Linear Algebra. Any matrix can be decomposed into three matrices using SVD. In machine learning, SVD is typically used to reduce dimensionality. Another popular dimension reduction technique is PCA. We will cover both of them in this post.



<!--more-->



## Change of Basis



Suppose there is a point in the 2D space, how do you describe it? The common way is to use the Cartesian coordinate system, which is composed of two fixed perpendicular oriented axes, measured in the same unit of length. The two perpendicular axes are just a special set of vectors served as the basis of the 2D space. Actually, there are many other sets of vectors that can be the basis for the 2D space.



In Figure 1, the position of the red point is `(-4, 1)` when using the standard basis `(1, 0), (0, 1)` (colored in grey). If we change the basis to `(2, 1), (-1, 1)` (colored in blue), the position of the same point is `(-1, 2)`. 



![](/blog/post/images/change-basis-example.png "Figure 1: The same point in two different coordinate spaces")



From Figure 1, we can see that the absolute position of the red point always stay the same. However, the relative positions to the different bases are different. Mathematically, the red point can be described from the view of a coordinate system as follows,

$$
x = P_b[x]_b
$$


which is equivalent to 


$$
x = c_1 \bold b_1 + c_2 \bold b_2 + ... + c_n \bold b_n
$$

$$
P_b = [\bold b_1, \bold b_2,  ... , \bold b_n ]
$$


$$
[x]_b = [c_1, c_2, ... c_n]
$$



where $[x]_b$ is a set of scalars which represent the length along each axis of the coordinate system, and $P_b$ is the **change of coordinate matrix from the current basis to the basis to which we are going to switch**. 



Let's plug the above point and  the basis `(1, 0), (0, 1)`  (colored in grey) into this equation,



$$
P_b = [ (1, 0), (0, 1)]
$$

$$
[x]_b = (-4, 1)
$$

$$
x_b = -4 \begin{bmatrix}1\\\\ 0 \end{bmatrix} + 1 \begin{bmatrix}0\\\\ 1 \end{bmatrix} = \begin{bmatrix}-4\\\\ 1 \end{bmatrix}
$$


Let's do the same calculation with another basis(colored in blue).


$$
P_b = [ (2, 1), (-1, 1)]
$$

$$
[x]_b = (-1, 2)
$$

$$
x_b = -1 \begin{bmatrix}2\\\\ 1 \end{bmatrix} + 2 \begin{bmatrix}-1\\\\ 1 \end{bmatrix} = \begin{bmatrix}-4\\\\ 1 \end{bmatrix}
$$
From the above equations, we can see that $x_b$ is simply a combination of a set of vectors consisting of the basis for a vector space. 



Moreover, if we want to know the position of the red point under the standard basis when we are currently using another basis (colored in blue), we must know the matrix $P_b$ ( $[x]_b$ is already known). On the contrary, if we've already known the coordinates of the red point in our standard coordinate system, i.e. `(-4, 1)`,  we must know the inverse of $P_b$ ( $[x]_b$ is already known).


$$
x_c = P_{cb} x_b
$$

$$
(P_{cb})^{-1}x_c = x_b
$$




We can generalize this to any number of points and dimensions.



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





## References





