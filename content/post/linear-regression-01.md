---
title: "Linear Regression 01"
date: "2021-04-14"
description: "There are two main tasks in machine learning: regression and classification. Today we will talk about regression, more specifically, linear regression. Linear regression is simple and easy to understand and maybe it's the first algorithm that most of people learn in the world of machine learning."
# tags: []
categories: [
    "Machine Learning",
]
series: ["Machine Learning"]
katex: true

---



There are two main tasks in machine learning: regression and classification. Today we will talk about regression, more specifically, linear regression. Linear regression is simple and easy to understand and maybe it's the first algorithm that most of people learn in the world of machine learning. So let's go!



<!--more-->



## Problem statement

Suppose you are a teacher, and you record some data about the hours students spent on study and the grades they achieved. Then you want to predict the grade for given hours that someone spent. Here are some sample data you collected:



| Hours | 0.5  | 1    | 2    | 3    | 4    |
| ----- | ---- | ---- | ---- | ---- | ---- |
| Grade | 20   | 21   | 22   | 24   | 25   |

Since there are only two variables, let's plot them.



![scatter-plot](/blog/post/images/simple-linear-grade.png "Figure 1: The scatter plot of hours and grade")



Well, we can see that the variable `grade` is positive related to the variable `hours`. For simplicity, we can use a simple line(the red line in this figure)  to approximate this relationship. And this is exactly our first simple linear model.



## Simple Linear Regression



Remember a line equation is written in this way:


$$
y = ax + b
$$


In this example, $x$ is the variable `hours `and $y$ is the variable `grade` , which we already know. So the problem is how to calculate the parameter $a, b$. Technically, this is called **parameter estimation**. Usually, there are two ways to do this: minimising the loss and maximising likelihood. Now we focus on minimising the loss.



## Loss

What is the loss? Basically, it's the error between the esitmated value and our true value. Minimising the error is simply to make the estimated value as close as possible to the true value.



![simple-linear-grade](/blog/post/images/linear-regression-residual.png "Figure 2: The error for a single data \(Bradthiessen.com 2021\)")



For a single data point, the loss function is defined below, where $y$ is the true value and $y'$ is our estimated value for a given $a, b$.  


$$
error = y_i - y'_i = y_i - ax_i - b
$$


Since we have many data points, we need to sum up them all to evaluate the overall errors. 



### 1. error

Unfortunately, some error terms will cancel out when you do this calculation directly.


$$
L = \sum_i^n (y_i - y'_i) = \sum_i^n (y_i - ax_i - b)
$$


### 2. the absolute value of error

One way to tackle this is taking the absolute value of the error terms.


$$
L =  \sum_i^n |y_i - y_i'|
$$




However, the absoulte value of $x$ is not differentiable at $0$.



### 3. the squared value of error



Instead of taking absolute value, we will square all the errors. One reason is that the errors will become larger and can be distinguished easily when squaring them. It looks like the errors are zoomed in and we can find them and minimize them quickly. It is also known as `Residual Sum of Squares(RSS)` or `Sum of Squared Error (SSE)`


$$
L =  \sum_i^n (y_i - y_i')^2
$$


PS: We will revisit the squared error later from the perspective of MLE.



## Closed-form solution

Okay, finally we find a function to measure the loss. Next we need to find the parameters that minimize the squared error. Good news is that our loss function is differentiable and convex! It means that we have a global minimial value and can be calculated directly by taking derivatives.



Let's take the first derivatve of $b$


$$
\frac{\partial L}{\partial b} = \sum_i^n -2(y_i - ax_i-b)
$$


and then set this equation to $0$,




$$
-2(\sum_i^ny_i -a\sum_i^nx_i - \sum_i^nb) = -2(n\overline y-an\overline x - nb) = 0
$$

$$
b = \overline y - a\overline x
$$




Let's take the first derivatve of $a$


$$
\frac{\partial L}{\partial a} = \sum_i^n -2x_i(y_i - ax_i-b)
$$


and then plug $b$ into this equaiton and set this equation to 0 again,


$$
\sum_i^n -2x_i(y_i - ax_i-\overline y+a\overline x) = \sum_i^n -2x_i[(y_i-\overline y)- a(x_i -\overline x)]
$$

$$
a = \frac{\sum_i^nx_i(y_i-\overline y)}{\sum_i^nx_i(x_i -\overline x)}
$$


Here, we use a slight algebra trick,


$$
a\sum_i^n(x_i - \overline x_i) = 0
$$


Then we plug this into the previous equation


$$
a = \frac{\sum_i^nx_i(y_i-\overline y)}{\sum_i^nx_i(x_i -\overline x)} = \frac{\sum_i^nx_i(y_i-\overline y) - \sum_i^n\overline x(y_i - \overline y)}{\sum_i^nx_i(x_i -\overline x) - \sum_i^n\overline x(x_i - \overline x)}
$$

$$
= \frac{\sum_i^n(x_i-\overline x)(y_i-\overline y)}{\sum_i^n(x_i -\overline x)^2}
$$

$$
= \frac{Cov(x, y)}{Var(x)}
$$


Finally, we find the best estimators for simple linear regression.



## $R^2$



So how to evaluate our model? How good is it? We can use $R^2$ to measure this. Let's rewrite the previous euqation by multiplying both the denominator and numerator by $\sqrt {\sum_i^n(y_i-\overline y)^2}$


$$
a = \frac{\sum_i^n (x_i - \overline x)(y_i - \overline y) \sqrt {\sum_i^n(y_i-\overline y)^2}}{\sqrt {\sum_i^n(x_i-\overline x)^2} \sqrt {\sum_i^n(x_i-\overline x)^2} \sqrt {\sum_i^n(y_i-\overline y)^2}}
$$

$$
a = R\frac{s_y}{s_x}
$$


where 


$$
R = \frac{Cov(x, y)}{\sqrt{var(x)} \sqrt{var(y)}}
$$

$$
s_y =  \sqrt{Var(y)}
$$

$$
s_x = \sqrt{Var(x)}
$$




Remember that the error is defined as $e_i = y_i' - y_i$, so the mean of $e$ is


$$
E(e) = \frac{1}{N} \sum_i^n e_i =  \frac{1}{N} \sum_i^n b + ax_i - y_i =b + a\overline x - \overline y = 0
$$


and the variance is given by


$$
var(e) = \sum_i^n (e_i - \overline e)^2 = \sum_i^n  (y_i - b - ax_i)^2 = \sum_i^n (y_i - \overline y + a\overline x - ax_i)^2
$$


Let's plug $a$ into this equation


$$
var(e) = \sum_i^n [(y_i - \overline y) - R\frac{s_y}{s_x}( x_i - \overline x)]^2 = var(y) (1-R^2)
$$


Or you might be more familiar with this equation




$$
R^2 = 1 - \frac{var(e)}{var(y)} = 1 - \frac{RSS}{TSS}
$$




Therefore, $R^2$ tells us how much variance of $y$ has been explained by our models. The higher the $R^2$ is, the better our model is.





## Reference

[1] *Bradthiessen.com*, 2021. [Online]. Available: https://www.bradthiessen.com/html5/docs/ols.pdf. [Accessed: 14- Apr- 2021].

[2] http://mlwiki.org/index.php/Linear_Regression



