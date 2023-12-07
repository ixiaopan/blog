---
title: "K-means"
date: "2021-06-14"
description: ""
# tags: []
categories: [
    "Machine Learning",
]
katex: true
---



So far, we've talked much about supervised learning. Aside from it, there exist many other learning types such as unspervised learning, semi-supervised learning and so on. This post will introduce one of the most widely used unsupervised clustering algorithms — K-means. We will cover the implementation of K-means algorithm, the limitation of this algorithm as well as its applications.



<!--more-->



## Problem Formulation



Given an unlabelled dataset $D =[x_1, x_2, ..., x_N] $, we want to explore whether there is a pattern in the data and what pattern is. In the case of clustering, we want to know which samples are close to each other so as to form a cluster. To achieve this, we assign to each sample a membership indicator denoted by  $r_{nk}=1$ if the sample is in cluster $k$, otherwise $r_{nj}=0$ for $j \ne k$. Thus, 

- there could be $N$ clusters, i.e. each sample forms a cluster. 
- there could be only one cluster, i.e. the whole dataset forms a cluster.

So what's the optimal value for $K$? 



## Objective Function



The idea behind K-means is intutive to understand. Similar people or people with similar interests tends to form a group. Such a group also has a small variation. Based on this idea, the aim of K-means is to find $r_{nk}$ to minimise the within cluster variation,


$$
J = \sum^N_n\sum^K_k r_{nk} || x_n - \mu_k ||^2
$$


where $\mu_k$ is the center of the cluster that a sample $x_n$ belongs to. 



We should notice that this objective also limits the ability of K-means somehow. You can imageine $\mu_k$ as the center of a circle, and the average within cluster variation is the maximum radius of that circle. Any point outside that circle doesn't belong to this cluster. So K-means works better for circular clusters.



![](/blog/post/images/k-means-shape.png#full "Figure 1: K-means works better for data with circular clusters (Ref[2])")





## Approach



Since  $\mu_k$ and $r_{nk}$ are both unknown, how to solve it? Well, we solve this by repeating the following two steps,

- Step 1: Randomly choose $K$ cluster centers, and then minimise $J$ w.r.t $r_{nk}$. In this step, we assign to each sample a membership indicator given the fixed cluster centers.


$$
r_{nk} =\begin{cases} 1 & \text{if $ k =\text{argmin}_j ||x_n - \mu_j||^2$} \\\\ 0 & \text{otherwise}\end{cases}
$$


- Step 2: Keep $r_{nk}$ fixed and then calculate the new cluster centers — $\mu_k$ is equal to the mean of all samples assigned to that cluster $k$. This is because we minimise $J$ w.r.t $\mu_k$ by taking the derivative of $J$ and setting it to zero

  
  $$
  \frac{\partial J}{\partial \mu_k} = 2 \sum^N_{x_n \in C_k}  r_{nk} || x_n - \mu_k || =0
  $$
  
  $$
  => \mu_k = \frac{\sum_{x_n \in C_k} r_{nk} x_n}{\sum_{x_n \in C_k} r_{nk}}
  $$
  

So when will this process stop? There are some stopping criterions

- The centroids remain the same
- Within cluster variance reaches below the threshold you've set
- Fixed number of iterations have been reached



## Choosing the optimal K



Like the method used in K-NN, we plot the metric as a function of $k$ shown below.



![](/blog/post/images/kmeans-k.png#full "Figure 2: The optimal K lies at the elbow point (Hands-on machine learning)")



We can see that inertia decreases greatly as k increases up to 4, after that, it decreases slowly. On the other hand, increasing k indeed decrease the sum of squared distance from each sample to its closest centroid. This is because the more clusters k there are, the more cohesive each cluster is, and therefore the lower the within cluster variation is.



## Limits of K-means



### Specify K explicitly

Obvisously, we have to explicitly specify $K$. But more often, we don't what the right $K$ is.



### Sensitive to shape



As said earlier, K-means performs poorly when the clusters vary in sizes or densities, or have nonspherical shape. Figure below shows how K-means deals with a data set containing three ellipsoida clusters with different densities and orientation. Though the solution on the right has a lower inertia, it obviously breaks the inner structure of real clusters. Mabybe we could try Gaussian mixture models for this case.



![](/blog/post/images/kmeans-failure.png#full)





### Centroids initialization

Although K-means algorithm is guaranteed to converge, it might not coverge to the right solution. And this depends on the initialization of the centroids, which can also be seen in above Figure. To avoid this, we'd better run the algorithm several times and select the solution that has the minimum within cluster variation.



### Hard classification

Clustering assignment used in K-means is called hard classification since each sample is assigned to a cluster indicator. But sometimes we would expect probability to estimate uncertainty in clustering assignment. For example, if two clusters overlap, it's hard to have 100% confidence to say that some sample belongs to one of the two clusters.





## References

[1] A. Géron, *Hands-on machine learning with Scikit-Learn and TensorFlow*. Sebastopol (CA): O'Reilly Media, 2019.

[2]	J. VanderPlas, Python Data Science Handbook. Sebastopol, CA: O’Reilly Media, 2016.

