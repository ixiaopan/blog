---
title: "Recommender Systems"
date: "2021-11-07"
description: ""
# tags: []
categories: [
    "Machine Learning",
]
series: ["Machine Learning"]
katex: true
---



Recommender system is one of the most popular studying fields in machine learning due to its wide application in our daily life. When you shop online, such as Amazon, you can see similar items just below the item you are looking at. The goal of a recommender system is to make recommendations to people based on their preferences.



<!--more-->



## Overview

Recommendations can be classified as two categories, as Figure 1 shows,

- non-personalised
  - recommendations are the same for all users
  - e.g. the most popular movies, the top ten hotels, ...
- personalised
  - recommendations vary by users



Obviously, we focus on the latter one. The mainstream algorithms includes 

- content-based filtering
  - recommend items based on the attribtues of items, users, or both (feature engineering)
  - e.g. when you buy the novel "The ABC Murders ", the system will also recommend other novels written by Agatha Christie to you.
- collaborative filtering 
  - employ the wisdom of the crowd instead of extracting good attributes manually



There are three main types of collaborative algorithms, namely,

- user-based filtering
- item-based filtering
- matrix factorisation



![](/blog/post/images/recom.png#full "Figure 1: Different algorithms for recommender systems.")



## Data



Data is all we need. Figure 2 shows that a recommender system needs three types of data,

- items data
- user data
- the interaction between items and users



![](/blog/post/images/input-recom.png#full "Figure 2: Data involved in recommender systems.")

Each type of data has its own properties. Items like clothes have attributes of size and colour. Users have age and gender. The interactions between items and users include contextual attributes, such as weather, geographical location, day of the week, and so on.



### Item Content Matrix (ICM)

As mentioned earlier, items have attributes. Mathematically, we can describe that relation using Item Content Matrix or ICM, where each row indicates an item and each column represent an attribute. The value of ICM is either 1 or 0. If an item contains a specific attribute listed in ICM, the corresponding cell will be set to one, zero otherwise. But you can also set a value between 0 and 1 to describe the importance of that attribute.



### User Rating Matrix (URM)

Similarly, the opinion of users on items can also be described mathematically through the User Rating Matrix or URM. Each row represents a user, and each column represents an item. If we have no idea the opinion of someone on an item, the corresponding value is zero. 



#### Implict Ratings

Ratings can be implicit or explicit. Implicit ratings deduce your taste by looking at how you interact with the system, for instance, the time-on-page, the viewing time of a movie, clicks, purchases, and so on. If one stops watching a movie after 10 minutes, we assume that that user does not like that movie. However, one may have to stop watching a film because of an important call. 



#### Explict Ratings

Explicit ratings are done by asking users to rate items on a rating scale, for example, five stars for Educated.

| Large rating scale               | Smaller rating scale | Even rating scale                                            | Odd rating scale                                             |
| -------------------------------- | -------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| complex and requires more effort | easy to understand   | either negative or positive ratings, forcing people to express their opinions | introduces the neutral rating, which makes people feel more comfortable |
| fewer ratings                    | more ratings         | fewer ratings                                                | people tend to choose the neutral rating unless they have a strong positive or negative opinion |



## User Behavior Collection



## Ratings Collection



## Users From the Cold



## Non-personalised recs

### Top 10

As we said before, the top 10 perhaps is the most common type of non-personalised recs. You can see Top 10 everywhere on the Internet. 



#### Method 1: Top Popular

An intuitive method to get the top 10 items is to order items by the number of users that have bought, seen, or given their opinions.

```python
# top 10 movies
movies_has_seen = Log.objects.values('content_id').filter(event='seen').annotate(Count('user_id'))

sorted_items = sorted(movies_has_seen, key=lambda item: -float(item['user_id__count']))

sorted_items[:10]
```



#### Method 2: The Best Rated Items

Another technique is to calculate the average rating for each item and retrieve the top 10 items ranked by the average rating in descending order.



```python
movies_avg_rated = Rating.objects.values('movie_id').annotate(Avg('rating'))

sorted(movies_avg_rated, key=lambda item: -float(item['rating__avg']))[:10]
```



To calculate the average rating, we need the help of URM. Let $r_{ui}$ be the non-zero rating given by user $u$ to item $i$, and $N_i$ be the number of users who rated item $i$,  then the average rating of item $i$ is given as


$$
b_i = \frac{\sum_u r_{ui} }{N_i}
$$


### Market Basket Analysis

Apart from the "Top 10", another type of recommendations also appears often when you look at an item at Amazon. That is "Frequently Bought Together (People who bought this item also bought)". Basically, this kind of recommendations are based on association rule analysis (market basket analysis), which is a technique widely used by retailers to discover associations between items. They want to know what items are frequently bought together so that they can   place them in a similar manner.



Let $I = \{ I_1, I_2, ..., I_m \}$ be the items and $T = \{ t_1, t_2, .., t_N \}$ a list of transactions. 



#### Rule

Association rules can be written in the form of "IF-THEN",
$$
\\\{{ A \\\}} \rarr \\\{{ B \\\}}
$$
where $A \sub I$, $ B \sub I$, and $A \cap B = \empty$. The above rules means that if a customer buys $A$, then he is likely to buy $B$. A and B can include many items,
$$
\\\{{\text{bread},\text{milk}\\\}}\rarr\\\{{\text{carrots},\text{yogurt}\\\}}
$$


#### Support

Support tells us how frequently A and B appear together by calculating the fraction of transactions that contains A and B.


$$
\text {supp} (A \rarr B) = \frac{|A \cup B|}{|T|}
$$
Thus, we can filter out the item sets that occur less frequently by setting a minimum support.



#### Confidence

Confidence shows the percentage in which B is bought with A.


$$
\text {conf} (A \rarr B) = \frac{\text{supp}(A \rarr B)}{\text{supp}(A)}
$$

#### Lift

Lift indicates the strength of a rule.


$$
\text {list} (A \rarr B) = \frac{\text{supp}(A \rarr B)}{\text{supp}(A)\text{supp}(B)}
$$


With these terms, now we can make an association analysis using the Apriori algorithm.



```python
data = {
    '1': [1,0,1,0],
    '2': [0,1,1,1],
    '3': [1,1,1,0],
    '4': [1,0,0,0],
    '5': [0,1,1,1]
}
index = [ 'InvoiceNo_' + str(i) for i in range(4) ]
basket_set = pd.DataFrame(data=data, index=index)

freq_itemsets = apriori(basket_set, min_support=0.5, use_colnames=True)
freq_itemsets['length'] = freq_itemsets['itemsets'].apply(lambda x: len(x))
freq_itemsets

# 0.25 (4) 1 # since 0.25 < 0.5, so it's not included.
support	itemsets	length
0	0.50	(1)	        1
1	0.75	(2)       	1
2	0.75	(3)       	1
3	0.75	(5)	        1
4	0.50	(3, 1)     	2
5	0.50	(2, 3)    	2
6	0.75	(2, 5)    	2
7	0.50	(3, 5)    	2
8	0.50	(2, 3, 5)	3

association_rules(freq_itemsets, metric='confidence', min_threshold=0.75)

antecedents	consequents	antecedent_support	consequent_support	support confidence	lift
(1)	         (3)	       0.50	                0.75	               0.50	   1.0	1.333333
(2)	         (5)	       0.75	                0.75	               0.75	   1.0	1.333333
(5)	         (2)	       0.75	                0.75	               0.75	   1.0	1.333333
(2, 3)	     (5)	       0.50	                0.75	               0.50	   1.0	1.333333
(3, 5)	     (2)	       0.50	                0.75	               0.50	   1.0	1.333333
```



PS: [Here](https://github.com/ixiaopan/DataScience/blob/master/MachineLearning/15-MarketBasketAnalysis.ipynb) is the complete code.



## Evaluation

### Quality Indicators

Relevance、 Coverage、Novelty、Diversity、Consistency、Confidence、Serendipity



### Online Evaluation

#### Direct user feedback

- ques should be meaningful and non--biased
- opinion not reliable

#### A/B test

- monitoring user behavior
- difficult to set up
- result might be difficult to interpret

#### Controlled experiments



#### Crowdsourcing

 

### Offline evaluation



## Content-based Filtering





## Collaborative Filtering





## Matrix Factorisation





## Ranking





## References

- [Apriori Algorithm: Know How to Find Frequent Itemsets](https://www.edureka.co/blog/apriori-algorithm/)
- [Market Basket Analysis](https://www.kaggle.com/xvivancos/market-basket-analysis)

