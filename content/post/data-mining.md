---
title: "Information Theory"
date: "2021-06-07"
description: ""
# tags: []
categories: [
    "Machine Learning",
]
series: ["Machine Learning"]
katex: true
---



I am not sure what I've learned from the module 'Data Mining' is truly useful to me. It seems that the lecturers tried to cover everything, but in fact, they only scratched the surface due to time limits. Besides, the content they taught were less integrated. Half of the lectures revolved around the graph theory, the left were a mix of various techniques of machine learning. I am not going to do works related to graph, so I won't bother to learn such stuff. What I found really practical are information theory, association analysis, regression and recommender systems. And the first two are the main topics of this article.



<!--more-->



## Information Theory



I remembered that I'd learned a little stuff about information when I was at my undergraduate. Well, I am not sure...It's been almost 10 years. But as a CS student, I know that when it comes to information theory, we will definitely talk about a person — Shannon. So what's the information theory? What problems does it help to solve?



### Information

First, let's understand what information is. Consider a guessing game, Alice randomly chooses a number between 1 and 100, then we need to make a guess what that number is. An intuitive method is to use dichotomy search. We may ask 'is it larger or smaller than 50?'. If it's less than 50, we continue to ask 'is it larger or smaller than 25?' We do this iteratively until we find the right number. Mathematically, the number of searches is $log_2(100)$. If the number Alice chooses is small, say 3, then it would take many steps to get the correct answer.

If we know that Alice has some preferences for some particular numbers, say she likes 88 most. Then the first question we ask at this time would be 'is it 88?'. If we are luckly enough, we will get the number once.

On the other hand, since 88 is the number that Alice likes most, there is no doubt that she chooses 88 as the riddle. Conversely, if Alice is going to suprise us, she is likely to use numbers that she doesn't like very much. If so, it would also take us many guesses.



The number of questions we ask or the degree of surprise is information, which is measured by



$$
I(x) = -log(p(x))
$$



where $p(x)$ is a probablity distribution of a random variable $x$. It's clear that a rare event will cause much surprise and consequently contain much information. 



### Entropy



### Cross-entropy







## Market Basket Analysis





## References

