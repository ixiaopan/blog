---
title: "NLP - CRF"
date: "2021-11-22"
description: ""
# tags: []
categories: [
    "NLP",
]
series: ["Machine Learning"]
katex: true
draft: true
---



## Direced Graphical Model

In a direction graph, if there is a link starting from a to b, we say a is the parent of b and b is the child of a. The joint distribution over all variables $\bold x=\{x_1, x_2, ..., x_N\}$ in this graph is described as 
$$
p(\bold x) = \prod_i^N p(x_i|pa_i)
$$
where $N$ is the number of nodes, and $pa_i$ indicates the set of parents of $x_i$. For example, the joint distribution of the graph in Figure 1 is 
$$
p(a, b, c) = p(a)p(b|a)p(c|a,b)
$$
![](/blog/post/images/directed-graph.png "Figure 1: A directed graph representing the joint probability distribution over a, b, and c.")



## Undireced Graphical Model



### Conditional Independence

![](/blog/post/images/con-indep.png "Figure 2: ")



### Random Field



### Markov Random Field

#### Global Markov



#### Pairwise Markov



#### Local Markov





### Conditional Random Feild





### Linear CRF



## Matrix Notion



## Inference



## Parameter Estimation



## Decoding



## Refer

- [Performing sequence Labelling using CRF](https://albertauyeung.github.io/2017/06/17/python-sequence-labelling-with-crf.html/)
- [Introduction to Undirected Models – Markov Random Fields](https://web.ics.purdue.edu/~ibilion/www.zabaras.com/Courses/BayesianComputing/Intro2MarkovRandomFields.pdf)
- [Conditional Random Fields: Introduction](https://people.cs.umass.edu/~wallach/technical_reports/wallach04conditional.pdf)
- [NLP - CRF](https://www.cnblogs.com/Determined22/p/6915730.html)
- [NLP From HMM to CRF](https://anxiang1836.github.io/2019/11/05/NLP_From_HMM_to_CRF/)
- [Conditional Random Fields](https://www.davidsbatista.net/blog/2017/11/13/Conditional_Random_Fields/)
- [An introduction to Conditional Random Fields](https://homepages.inf.ed.ac.uk/csutton/publications/crftut-fnt.pdf)

