---
title: "An E2E Project - EDA"
date: "2021-04-24"
description: ""
# tags: []
categories: [
    "Machine Learning",
]
series: ["Machine Learning"]
katex: true
---



We have discussed many algorithms, such as linear regression and ensemble methods. It's time to kick off a project from scratch to learn the pipeline of a machine learning project.



<!--more-->



## Overview



The book, "Hands-on Machine Learning", has summarised the below 8 steps as a guidance to do a end-to-end machine learning project,

- Frame the problem
- Get the data
- Explore the data
- Prepapre the data
- Explore many different models
- Fine-tune
- Deploy



In this post, we will cover the first 3 parts.



## Frame the problem



The first step is to define you problem. People are unlikely to do things without reasons, right?. So ask yourself, what problem do you want to solve? Why are you interested in them? What's your objective? Are there any solutions out there?

As an example, we are going to predict used car prices because accurate prices prediction can help both buyers and sellers. For buyers, it can ensure the money that customers invest on used cars to be worthy. For used car dealers, they might want to know which factors influence car prices most so as to adjust sales strategy and offer a better prediction to customers. Therefore, there is a necessity for building a used car price prediction system.



## Get the data

There are many ways to get the desired data. You can write a web crawler to download the data from related websites or you can get data freely from some public data platforms.Among them, Kaggle is one of the most popular platforms that enables us to achieve our data science goals. In this example project, the data is downloaded from [Used Cars Dataset - Kaggle](https://www.kaggle.com/austinreese/craigslist-carstrucks-data).



```python
!kaggle datasets download -d austinreese/craigslist-carstrucks-data -p data

!unzip data/craigslist-carstrucks-data.zip -d data

```



## EDA



EDA stands for Exploratory Data Analysis, which is an important part throughout the project. The goal of EDA is to get insights from the data so as to clean and prepare data for building models. Generally speaking, it involves the following steps,

- Identify variables
- Examine the quality of the data
- Univariate analysis
- Bi-variate analysis



### Identify variables



Before any further analysis, we need to have a look at the data generally. More specifically, you need to answer the following questions,

- How many observations and variables do you have?

- Which variables are your predictors and target?

- What's the data type and the memory usage?

- What's the descriptive statistics about the data?

  

Luckily, Pandas provides convenient functions to answer these questions.



```python
df_vehicles.shape

df_vehicles.info()

df_vehicles.describe()

```



In our example, we have `458,213` rows and `25` attributes. The `price` is our target variable and the remaining are our predictors. The following table shows the distribution of data types and Figure 1 shows some example data.



| Data Type     | Variable Name                                                |
| ------------- | ------------------------------------------------------------ |
| Numerical (5) | id, price, odometer, lat, long                               |
| Object (20)   | 'url', image_url, region_url, 'manufacturer',  model, 'condition', 'cylinders',   'fuel', 'title_status', 'transmission',    'drive',  'type', 'paint_color', 'region', 'state',posting_date, description, image_url, size, VIN, |



![Example Data](/blog/post/images/used-car-toydata.png#full "Figure 1: Some example data")



### Examine data quality

The raw data is unlikely to use directly because it's inevitable to introduce errors like duplication, null values and extreme values when collecting data. The higher the data quality is, the better the model performance is. So we need to identify and solve these problems to obtain a clean data set.



### Univariate analysis





### Bi-variate analysis





## References



[1] https://www.kaggle.com/austinreese/craigslist-carstrucks-data



