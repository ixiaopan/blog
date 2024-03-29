---
title: "Naive Bayes Classification"
date: "2021-06-16"
description: ""
# tags: []
categories: [
    "Machine Learning",
]

katex: true
---





In the previous articles, we introduced several classification algorithms like logistic regression. These models are often called discriminative models since they make prediction by calculating \\(P(Y|X)\\) directly. Sometimes it might be hard to compute. Another way to think of this is that samples are generated from the existed distributions. And one of the most popular models is Naive Bayes classification. 



<!--more-->



## Model



Given a sample \\(\bold x = [x_1, x_2, ..., x_d]\\), where \\(x_i\\) represent each individual feature, we want to know the probability of \\(\bold x\\) belonging to each class \\(C_k\\). With the aid of Bayes' inference, we can inverse this problem into the forward problem


$$
P(Y=C_k|X= \bold x) = \frac{P(X= \bold x|Y=C_k)P(Y=C_k)}{P(X= \bold x)}
$$


Then, Naive Bayes makes the following assumptions

- all features are independent — one paticular feature doesn't affect another, though it's not real in practice. That's why it's called naive (lack of experience)
- all features have equal importance



Based on the above assumptions, we can rewrite Bayes' theorem as follows,


$$
P(Y=C_k|X=\bold x) = \frac{P(x_1|Y=C_k)P(x_2|Y=C_k)...P(x_d|Y=C_k)P(Y=C_k)}{P(X=\bold x)}
$$


Since $P(X=\bold x)$ is the evidence, 


$$
P(Y=C_k|X=\bold x) \propto P(Y=C_k) \prod_{i=1}^d P(x_i|Y=C_k)
$$


Now the question is that where \\(P(x_i|Y=C_k)\\) comes from. What distribution does the class \\(C_k\\) follow? In Naive Bayes, there are three typical distributions

- Bernoulli 
- Multinonimal
- Gaussian



## Bernoulli Naive Bayes

In Bernoulli distribution, the outcome of an event, denoted by \\(y_i\\), is either 1 or 0. Let \\(p\\) be the probability of being 1, so we have


$$
P(Y=y_i) = y_ip +  (1- y_i )(1 - p)
$$


Since 0 or 1 can encode the occurrence of an event, we can use this to filter spam email. We don't care how many suspect words occur in an email. Instead, once they occur, this email should gain more attention. Hence, each email can be represented by a word vector with binary element taking value 1 if the corresponding word appears in the email and 0 otherwise,


$$
\bold  x = (x_1, x_2, ..., x_{|v|}), \text{ where } x_i \in {1, 0}
$$


where \\(V\\) is the vocabulary of the training emails. In other words, we do $|V|$ Bernoullis trials for each email. For each word in $V$, if it's present in that email, the outcome is 1, otherwise 0. This repeats \\(|V|\\) times until all words in  \\(V\\) have been visited.


Now suppose there are \\(D\\) emails, including \\(D_s\\) spam emails and \\(D_h\\) ham emails, given a new email \\(\bold  x \\), we want to know whether it's a spam email or not. Based on the model defined above, we need to calculate two things

- the probability of being each class, \\(P(Y = C_k)\\)
- given that class \\(C_k\\), the probability of sampling the new data \\(x\\), \\(P(X=\bold x|Y=C_k) = \prod_{i=1}^V P(x_i|Y=C_k)\\)


The first question can be solved by the following equation,


$$
P(Y=S) = \frac{|D_s|}{|D|}, P(Y=H) = 1 - P(Y=S)
$$


Based on the assumption of Naive Bayes, we consider each word in the vocabulary as a single independent random variable that follows Bernoulli distribution with probability \\(p\\), where $p$ is the probability of a word \\(x_i\\) occurring in an email of class \\(C_k\\) and can be computed as follows,


$$
p(x_i | Y=C_k) = \frac{|D_s(w_i)|}{|D_s|}
$$


where \\(|D_s(w_i)|\\) represent the number of emails that belong to class \\(C_i\\) and contain the word \\(x_i\\). Therefore, the probability of \\(\bold x\\) being classified into the spam category is 


$$
P(Y = S| X = \bold x ) \propto P(Y=S) \prod_{i=1}^V P(x_i|Y = S)
$$

$$
= \frac{|D_s|}{|D|} \prod_{i=1}^V \text{ } y_i  P(x_i|Y=S)  +  (1- y_i )(1 - P(x_i|Y=S))
$$




## Multinomial Naive Bayes



In multinomnial distribution, there are `K` possible outcomes, so the probability is


$$
p(x_k) = \prod_{k}^K \mu_k^{I_{k}}, \text{ where } _{k} = 1 \text{ if } x_k = 1, \text{otherwise } 0
$$


If we do \\(N\\) trials, the likelihood of the observations is


$$
P(m_1, m_2, ..., m_k) = \prod_{n=1}^N \prod_{k}^K \mu_k^{I_{nk}} = \frac{N!}{m_1!m_2!...m_k!}\prod_{k}^K \mu_k^{m_k}
$$


where  \\(m_k=\sum_n^N I_{nk}\\) represents the number of \\(I_{nk} = 1\\) and \\(\sum_{i=1}^k m_i = N\\). Through MLE, \\(\mu_k\\) is estimated as


$$
\mu_k = \frac{m_k}{N}
$$


Sometimes we are more interested in the frequency of a word, for example, sentiment classification. If words about happy occur more frequently, then we're more likely to classify the text as the positive category. In this case, each text can be represented by a word vector with element whose value is the frequency of that word.


$$
\bold  x = (x_1, x_2, ..., x_{|V|}), \text{ where } x_i \in Z
$$


where \\(V\\) is the vocabulary of the training texts. Thus, we can see that a text can be generated by doing \\(|N_x|\\) multinominal trials. 

Suppose there are \\(D\\) texts, including \\(D_p\\) positive texts and \\(D_n\\) negative texts, given a new text \\(\bold x\\), we want to know whether the sentiment that that text shows is positive or negative.



Again, the probability of being a class \\(C_k\\) is given by,


$$
P(Y=\text{Positive}) = \frac{|D_p|}{|D|}, P(Y=\text{Negative}) = 1 - P(Y=\text{Positive}) = \frac{|D_n|}{|D|}
$$


The likelihood of generating the sample \\(\bold x\\) is given by,


$$
P(X=\bold x|Y=C_k) = P(x_1, x_2, ..., x_{|V|}|Y=C_k) \propto \prod_{k}^V \mu_k^{x_k}
$$


where \\(x_k\\) represents the frequency of that word that occurs in the text \\(\bold x\\) and \\(\sum_{i=1}^V x_i = N_x\\). 

For words that not occur in the text, \\(\mu_k^0 = 1\\). Thus, the posterior probability can be rewritten as follows,


$$
P(Y = \text{Positive} | X = \bold x ) \propto P(Y= \text{Positive} ) \prod_{k=1}^{N_x} \mu_k
$$




## Gaussian Naive Bayes

Similarly, Gaussian Naive Bayes assumes that data from each class follows a Gaussian distribution.



## Zero Probability

If a new sample contains a word that is unseen in the training data, the probability is zero.


$$
\mu = \frac{m_k}{N} = 0
$$


If one of the term \\(\mu_k\\) is zero, the whole probability will be zero too. One way to avoid this is to use add-one smoothing, adds a count of one to each unique word.


$$
\mu = \frac{m_k + 1} {N + |V|}
$$




## References

- https://www.inf.ed.ac.uk/teaching/courses/inf2b/learnnotes/inf2b-learn07-notes-nup.pdf
- https://jakevdp.github.io/PythonDataScienceHandbook/05.05-naive-bayes.html
- https://medium.com/atoti/how-to-solve-the-zero-frequency-problem-in-naive-bayes-cd001cabe211

