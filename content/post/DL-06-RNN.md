---
title: "Deep Learning - RNN"
date: "2021-09-26"
description: ""
# tags: []
categories: [
    "Deep Learning",
]
series: ["Machine Learning"]
katex: true
---



When I first learned RNN, I found lots of articles about it, that's good. But most of them focus on the design of the architecture. Well, the network itself is not difficult to understand, the real problem is why. What's the intuition behind it? Why do we need "recurrence"? Unfortunately, few people explain it clearly. Luckily, the book [Deep Learning](https://www.amazon.co.uk/Deep-Learning-Adaptive-Computation-Machine/dp/0262035618) gives the answers.



<!--more-->



## RNN

### Sequence Modeling 

Recurrent neural networks (RNN) is another kind of neural networks, which are good at processing sequential data, such as stock prices, text, and audio. Machine translation and speech recognition are typical applications. So, what is special about sequential data? Why not still use MLP? Why do we bother designing another type of network to suit it?

Q1: the characteristics of sequential data

Sequential data consists of a series of data points across time, which have a dependency on each other. In CNN, we feed a single image into the network each time, but the order of these images doesn't matter. Each image is an independent individual. However, when we process text, the order of words matters. For example, "I like running" is a correct sentence while "Running like I" is wrong. More importantly, there are dependencies between words. We understand the meaning of a piece of text or audio based on the understanding of the previous words.



Q2: Why not MLP?

Remember that there are two main reasons why we use CNN rather than MLP

- MLP cannot capture spatial information
  - Flattening will cause the loss of spatial information of image pixels, especially there are multiple channels (depth). For example, the colour of a pixel can be estimated by considering the nearest neighbouring pixel's value
- Repeated pattern is hard to detect
  - The object to be detected (e.g. Cats) can appear anywhere in the image. Actually, we do not care about locations but the pattern.

Well, the reasons why MLP is not suitable for sequential data are similar.

- Spatial information in sequential data means the order of data and dependencies among data.
  - Each word is considered as an input feature when using MLP, but the representation of each word does not contain dependencies between words.
- A sentence can be written in different ways using the same words. 
  - For example, "Today, I am going to watch a movie." is the same as "I am going to watch a movie today." When we feed each word into MLP, "Today" will be assigned with different weights.
- Sentence length is variable.

Q3: What kind of networks is suitable for sequential data?

From the above analysis, we conclude that a network that is capable of modeling sequential data should be able to

- handle with variable sequence length
- retain the order of data and dependencies between data
- detect repeated patterns



### Architecture

So, how RNN works? Figure 1 shows that RNN is somehow a kind of state machine, which only accepts two inputs:

- the previous system state $h_{t-1}$
- the current input $x_t$



![](/blog/post/images/rnn-unfold.png#full "Figure 1: The architecture of RNN.")



Specifically, RNN is defined as


$$
h_t = W_x x_t + W_h h_{t-1} + b_h \\\\ o_t = W_y h_t + b_y \\\\ \hat y_t = \sigma(o_t)
$$
First, unlike the Feed Forward Networks that requires fixed length of the inputs, RNN can accormadate variable sequence length by looping. Second, the input data are fed into the network sequentially to maintain the order. At each time $t$, the system will reach a new state $h_t$ by receiving the previous state $h_{t-1}$ and the current input $x_t$. And the new state will be fed into the network recursively. In other words, at each time $t$, the system has already known what happend before through $h_{t-1}$. Finally, the weights of RNN are shared ( $W_t, W_h, W_y, b_h, b_y$ ) each time step to capture the repeated pattern.



### BPTT

Traning a RNN is nothing special. First, let's define the loss function. For a sequence whose length is $T$, the loss is computed as


$$
L = \sum_{t=1}^T L_t (\hat y_t, y_t) = \sum_{t=1}^T -y_t \text{log}\hat y_t = \sum_{t=1}^T -y_t \text{log} \sigma(o_t)
$$
where $\sigma$ is the softmax function. The computation graph is shown below.



![](/blog/post/images/rnn-graph.png "Figure 2: Computation Graph of RNN")



#### $W_y$



First, we compute the gradient of $L$ w.r.t $W_y$. From Figure 2, we can see that $W_y$ has three parents $o_0, o_1, o_2$, so we can generalize it to $T$ time steps
$$
\frac {\partial L}{\partial W_y} = \sum_{t=1}^T \frac {\partial o_t}{\partial W_y} \frac {\partial L}{\partial o_t} = \sum_{t=1}^T \frac {\partial o_t}{\partial W_y} \frac {\partial \hat y_t}{\partial o_t} \frac {\partial L}{\partial \hat y_t}
$$


So what's the derivative of $L$ w.r.t $o_t$? 


$$
\frac {\partial L}{\partial o_t} = -y_t \frac{1}{\sigma(o_t)} \frac{\partial \sigma(o_t)}{\partial o_t} - \sum_{i \neq t} ^N y_i \frac{1}{\sigma(o_i)} \frac{\partial \sigma(o_i)}{\partial o_t}
$$




Generally, $\sigma$ is the softmax function. Thus,


$$
\frac{\partial \frac{e^{o_t}}{\sum_{k=1}^K e^{o_k}}}{\partial o_i} = \frac{ \frac{\partial e^{o_t}}{\partial o_i} \sum_{k=1}^K e^{o_k} - e^{o_i} e^{o_t} }{ [\sum_{k=1}^K e^{o_k} ]^2}
$$


due to the derivative of $f(x) = \frac{g(x)}{h(x)}$




$$
f'(x) = \frac{g'(x)h(x) - h'(x)g(x)}{h^2(x)}
$$


Thus, for $i = t$, we have


$$
\frac{\partial \sigma(o_t)}{\partial o_t} = \sigma(o_t) (1 - \sigma(o_t))
$$
for $i \ne t$, we have


$$
\frac{\partial \sigma(o_i)}{\partial o_t} = - \sigma(o_i) \sigma(o_t)
$$
Therefore,


$$
\frac {\partial L}{\partial o_t} = -y_t \frac{1}{\sigma(o_t)} \sigma(o_t) (1 - \sigma(o_t)) + \sum_{i \neq t} ^N y_i \frac{1}{\sigma(o_i)} \sigma(o_i) \sigma(o_t) = -y_t (1 - \sigma(o_t)) + \sum_{i \neq t} ^N y_i \sigma(o_t) \\\\ = -y_t + y_t \sigma(o_t) + \sum_{i \neq t} ^N y_i \sigma(o_t) = \sigma(o_t) - y_t
$$


Finally, we have
$$
\frac {\partial L}{\partial W_y} = \sum_{t=1}^T \frac {\partial o_t}{\partial W_y} \frac {\partial L}{\partial o_t} = \sum_{t=1}^T  (\sigma(o_t) - y_t) \otimes h_t
$$


#### $W_h$





### Truncated BPTT



## Gated RNN

### Vanishing Gradient



### LSTM



### GRU



## References

- [Sequence Modeling with Neural Networks – Part I](https://www.kdnuggets.com/2018/10/sequence-modeling-neural-networks-part-1.html)
