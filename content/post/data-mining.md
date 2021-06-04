---
title: "Data Mining"
date: "2021-06-02"
description: ""
# tags: []
categories: [
    "Machine Learning",
]
series: ["Machine Learning"]
katex: true
draft: true
---



I am not sure what I've learned from this module is truly helpful for me because the content of this module is a hodgepodge of many subjects. The lecturers indeed delivered something useful, but the knowledge is rather scattered and less integrated. It seems that they tried to cover everything, but in fact, they only scratched the surface due to time limits. Anyway, I gained some insights from this module, and I will talk about some topics I found interesting and useful in this post.



## Information Theory



## Entropy



### Information



For example

>Sun rose (probability=1) | Information = 0 bits
>
>this message was obvious and gave you no new information.
>
>
>
>Sun did not rise (probability=0) | Information = ∞ bits
>
>You came out shocked and saw the world had gone crazy. This was breaking news — lots of information.



From this example, we can see that the information contained in a message about an event is related to the uncertainty and surprise-value of the event. ==An occurrence of an unlikely event gives you more information than the occurrence of a likely event.== 

用数学公式表达就是


$$
I(x) = -log(p(x)) 
$$


### Entropy



The entropy of X is then defined as the *expected value* of the information conveyed by an outcome in X.



> *Expected Value or Expectation of random variable X, written as E[X], is the average of all values of X weighted by the probability of their occurrence*



![image-20210312151117869](/Users/wuxiaopan/Library/Application Support/typora-user-images/image-20210312151117869.png)







![image-20210319084208384](/Users/wuxiaopan/Library/Application Support/typora-user-images/image-20210319084208384.png)



在 Speech and Language Processing 中提到



> Entropy is a measure of information. Given a random variable X ranging over whatever we are predicting (words, letters, parts of speech, the set of which we’ll call χ) and with a particular probability function, call it p(x), the entropy of the random variable X is:
>
> 
>
> One intuitive way to think about entropy is as a lower bound on the number of bits it would take to encode a certain decision or piece of information in the optimal coding scheme.



### Cross-entropy 

Cross-entropy measures the relative entropy between two probability distributions over the same set of events. Intuitively, to calculate cross-entropy between P and Q, you simply calculate entropy for Q using probability weights from P



![image-20210312151555839](/Users/wuxiaopan/Library/Application Support/typora-user-images/image-20210312151555839.png)



> The cross-entropy is useful when we don’t know the actual probability distribution p that generated some data.
>
> - Speech and Language Processing




$$
H(P, Q) \ge H(P)
$$


> The more accurate m is, the closer the cross-entropy H(p,m) will be to the true entropy H(p). Thus, the difference between H(p,m) and H(p) is a measure of how accurate a model is. 
>
> Between two models m 1 and m 2 , the more accurate model will be the one with the lower cross-entropy. (The cross-entropy can never be lower than the true entropy, so a model cannot err by underestimating the true entropy.)



在机器学习中，X 就可以理解为比如分类问题的 class, 对一个input来说，模型会对所有的class预测其属于这个class的概率，实际上，我们观察到的概率分布是 `0, 1, 0, 0` （假设有4个分类，这个input属于第二个class的概率是100%)，



![image-20210312151652383](/Users/wuxiaopan/Library/Application Support/typora-user-images/image-20210312151652383.png)





然后我们对其计算 cross-entropy 得到值 0.736 ；如果概率是 `0.05, 0.90, 0.05` 那么值是 0.15。说明我们的模型越来越接近真实的概率分布了，这也是 minimize loss function的地方



此外，我们还知道在 Logistic Regression 中， Maximize LIkelihood 其实就是 Minimize entropy，因为激活函数是 sigmoid 通过计算就能得到这个结果





## Market Basket Analysis





## Network





## Community Detection





## Link Prediction





## Pagerank

