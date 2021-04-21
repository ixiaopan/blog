---
title: "Descriptive Statistics"
date: "2021-04-13"
description: "In this series, I'm going to go through the basic concepts of statistics required in Data Science."
# tags: []
categories: [
    "Machine Learning",
]
series: ["Machine Learning"]
katex: true

---



In this post , I'm going to go through some basic concepts of  statistics required in Data Science. 



<!--more-->



There are two main branches of statistics :

- descriptive statistics tells us the statistics about the data like mean, mode and standard deviation, which you've learned in high school. 

- inferential statistics, on the other hand, uses a random dataset sampled from population to make inferences about population. 

We will firstly focus on descriptive statistics, specifically, the central tendency and dispersion. Central tendency measures the center of the data while dispersion measures how spread out a given data is.



## Central tendency

Mean, mode and median are three mainly used measures of central tendency.

### Mean

Mean is the average of the data and calculated by summing up all data values and then dividing them by the number of data.


$$
\overline x = \frac{\sum_i^nx_i}{n}
$$



For instance, say we have a group of data `1,2,3,4,5,6`, the mean is `3.5`.



### Median

Though mean is widely used, it's sensitive to outliers.  Suppose you have a set of data `1,2,3,4,5,6,100`, after some calculations, you find that the mean is `17.28`. However, it seems a bit strange since most of the data values is less than `10` except one extreme value `100`, which stretches the distribution of the whole data set to the right. This is why median comes.

To get the median, firstly we need to sort the data in ascending order and then find the middle number that separate the data into two groups with the same size. In this example, the median is `4`.



### Mode

Mode is the most frequent value. There could be one, two or more data values that have the same frequency and that freqency is the highest.



## Dispersion

### Range

Range is the distance between the maximum value and the minimum value. Again, range is sensitive to outliers.


$$
r=max - min
$$


### Quantile

Quantiles are used to divide data into several equal-sized groups. The most widely used cut points are `0, 25, 50, 75, 100`, denoted by `min, Q1, Q2, Q3, max` respectively.



IQR or interquartile range measures where the central 50% of the data is.


$$
IQR = Q3 - Q1
$$


IQR can be used to detect outliers. Data that is greater than **upper boundary** or less than **lower boundary** can be considered as an outlier.


$$
upper \ boundary = Q3 + 1.5*IQR
$$

$$
lower \ boundary = Q1 - 1.5*IQR
$$


### Variance

Deviation is the distance between a given data point and the mean. Since there are many data points, the variance calculates the average deviation from the mean. 



But when you do this, you will always get a zero due to the definition of the mean. 


$$
\sum_i^n d_i = \sum_i^n (x_i - \overline x) = \sum_i^nx_i - n\overline x = n\overline x - n\overline x  = 0
$$


For simplicity, I omit the denominator `n`.

Okay,  let's ignore the sign and use the absolute value of the deviation. 


$$
d_i = |x - \overline x|
$$


Though it works, the most popularly used method is calculate the square of the deviation.


$$
s^2 = \frac{\sum_i^n (x_i - \overline x)^2}{n-1}
$$


Though variance works fine, the value is a bit hard to interpret. For instance, we have 15 records of fish size measured in kilogram: 



 `[ 2.1, 2.4, 2.4, 2.4, 2.4, 2.6, 2.9, 3.2, 3.2, 3.9, 4.5, 6.3, 8.2, 12.8, 23.5 ]`. 



The variance is `30.97`.  It means that if we randomly catch a fish, its weight would be `30.97 squared kilogram` far away from the average weight. Emm...squared kilogram is an odd unit.



### Standard Deviation

It's simple to solve this problem by taking the square root of the variance, which is standard deviation.


$$
s = \sqrt{\frac{\sum (x - \overline x)^2}{n-1}}
$$


 So the standard deviation in the previous example is `5.56kg`, which makes much sense.



## Distribution



### Skewness

Skewness measures the symmetry of a distribution.  It's quite common to have non-symmetric distributions.



![skewness](/blog/post/images/skewness.png)



A left-/negative-skewed distribution 

- it has a long left tail
- the mean is on the left of the median



A right-/positive-skewed distribution 

- it has a long right tail
- the mean is on the right of the median



A skewed distribution implies that there are some special values that are larger/smaller than the common values.  Let's see an example.



![skewness](/blog/post/images/fish-skew.png)



Figure 3.2 shows the histogram of the fish sizes gathered from a fisherman. We can see that this distribution is right-skewed. The majority of fish sizes are between 0 and 3kg and there are a few special fishes that weigh over 3.5kg. It also can be seen that the average weight(1.67kg) is a bit higher than the median(1.62kg) shown in Table 3.2.



## References

[1] B. al., "Introduction to Statistics | Simple Book Production", *Courses.lumenlearning.com*, 2021. [Online]. Available: https://courses.lumenlearning.com/introstats1. [Accessed: 14- Apr- 2021].

