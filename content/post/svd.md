---
title: "Singular Value Decompsition"
date: "2021-04-27"
description: ""
# tags: []
categories: [
    "Machine Learning",
]
series: ["Machine Learning"]
katex: true

---



## Change of Basis



Suppose there is a point in the 2D space, how do you describe it? One common way is to use Cartesian coordinate system that is defined by x and y axes by listing a pair of numbers, such as `(x, y)`. Perhaps there are some people who don't know Cartesian coordinates, and they might choose a random set of vectors as their reference coodinate. For example, the red point in Figure 1 is `(-4, 1)` from our standard view(in gray) while the coordinate is `(-1, 2)` if we are in the view of another person(in blue).



![](/blog/post/images/change-basis-example.png "Figure 1: The same point in two different coordinate spaces")



Mathematically, the red point can be described from the view of new coordinate system represented in blue as follows,


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


where  $P_b$ the **change of coordinate matrix from the new basis to the standard basis** and $[x]_b$ is the coordinates in the new coordinate system. 



In this example, $P_b = [(2, 1),(-1, 1)]$, $[x]_b = (-1, 2)$ and then we have


$$
\displaystyle{\begin{bmatrix}-4\\\\ 1 \end{bmatrix} = \begin{bmatrix}2&-1\\\\ 1&1\end{bmatrix}
\begin{bmatrix}-1\\\\ 2 \end{bmatrix}
}
$$


Basically, $P_b$ is the coordinates of the blue vectors described from our standard coordinate system and also is the basis of the new coordinate system. The above equation tells us where the red point is from our standard view if we know its coordinates in other system $[x]_b$ and the corresponding change of coordinate matrix $P_b$.



On the other hand, if we know the coordinates in our system, how to find the corresponding coordinates in another system? In Figure 2, the red point $(-4, 1)$ lies in the standard coordinate system, and we want to know where it is from the view of the red coordinate system defined by the two red vectors? In other words, we want to know the length along each axis in the red coordinate system to arrive at the red point.



![](/blog/post/images/projection-example.png "Figure 2: Project onto the new coordinate system")



Suppose the two red vectors are defined as $v_1, v_2$, then we have




$$
\begin{bmatrix}v_{11}&v_{12}\\\\ v_{21}&v_{22}\end{bmatrix}
\begin{bmatrix}-4\\\\ 1 \end{bmatrix} = V^T x = x_{b}
$$




Well, you might find that it looks like the above equation. Actually, $V^T$ is just the change of coordinate matrix from the standard basis to the new basis and $x$ is the coordinate in the standard basis. The result is the new coordinate in the new basis. So it's an inverse transformation of the previous transformation.



We can generalize this to any number of points and dimensions, where



- $A$ is the $D\times N$ matrix with $N$ points and $D$ dimensions
- $V^T$ is the the change of coordinate matrix from $A$ to $S$
- $U$ is the the change of coordinate matrix from $S$ to $A$
- and $S$ is the new coordinates of all the points after some transformations.


$$
V^T A  = S\\\\(M, D) \times (D, N) = (M, N)
$$



$$
A=US\\\\(D,N)= (D, M) \times (M, N)
$$


If we do some transformation in the standard coordinate system, what's the new coordinates in another system? This can be solved by the following equation, 


$$
x_s' = V^TTUx_s
$$


- $x_s$ is the coodinates of $x$ in the coordinate system $S$, 
-  $Ux_s$ is the corresponding coodinates in the coordinate system $A$, 
- then we do some transformation by applying a matrix $T$, 
- finally we transform the coordinates back to the system $S$



If $T=I$, then $x_s'$ is exactly the same as $x_s$.





## SVD



### Linear Regression



### Principal Component





## PCA









## References





