---
title: "Sorting"
date: "2021-07-11"
description: ""
# tags: []
categories: [
    "Algorithm",
]
series: ["Computer Science"]
katex: true

---



Data structure and algorithm are two important courses in Computer Science. Knowing how to utilise appropriate data structures and algorithms to fit our problems can greatly decrease memory space and improve performance. Besides, the ideas behind the classical algorithms are also worth learning to strengthen our logical thinking skills. Today, I will introduce four common sorting algorithms. Well, I am not going to repeat things we've known, so this is simply a quick note to help refresh my knowledge for future reference.

<!--more-->



## Bubble Sort



### Idea



Bubble sort is the first sorting algorithm I learned in the module 'Data Structure'. The idea is simple,



> each time, we compare numbers in pairs and make an exchange until we put the largest number at the rightmost position



Say we have $n = 6$ numbers  `9, 4, 2, 16, 10, 12`, 



In the first iteration, there are `n = 6 ` items left to sort, we need to compare `n - 1 = 5` pairs of elements



```python
(9, 4), 2, 16, 10, 12
4, (9, 2), 16, 10, 12
4, 2, (9, 16), 10, 12
4, 2, 9, (16, 10), 12
4, 2, 9, 10, (16, 12)
```



Similarly, for the second loop, there are `n - 1 = 5` items left to sort, we need to compare `n - 2 = 4` pairs of elements



```python
(4, 2), 9, 10, 12, 16
2, (4, 9), 10, 12, 16
2, 4, (9, 10), 12, 16
2, 4, 9, (10, 12), 16
```



From this we can conlude that, for the $i\text{th}$ loop, 

- there are $n - i$ items left to sort
- consequently, we need to compare $n - i - 1$ times

Since we place the largest number at the rightmost position each loop, so there are $ n - 1$ loops in total. The above procedure are shown in the following table.

| iterations | times we need to compare |
| ---------- | ------------------------ |
| 0          | n - 1                    |
| 1          | n - 2                    |
| 2          | n - 3                    |
| ...        |                          |
| n - 1      | 1                        |



### Code



```python
def bubbleSort(nums):
    n = len(nums)

    for i in range(n - 1): # each loop
        for j in range(n - i - 1):
            if nums[j] > nums[j+1]:
                nums[j], nums[j+1] = nums[j+1], nums[j]

    return nums        

bubbleSort([9, 4, 2, 16, 10, 12])
# [2, 4, 9, 10, 12, 16]
```



### Analysis

**Stable - Yes**

Since we swap values only when the previous item is greater than the latter one, the same value won't swap each other.



**Complexity - O($n^2$)**

From the table above, we can see that, 

- in the worst case, we need to run comparison $1 + 2 + ... + (n - 1) = \frac{(n-1)n}{2}$ times
- in the best case, we still need to compare $\frac{(n-1)n}{2}$ times



In the given example, we notice that `2, (4, 9), 10, 12, 16` has been sorted, but the comparison still continues. One way is to set a flag to notice it. We can do this because Bubble Sort is an inplace sorting, which means the list has sorted in an ascending order while comparing.



```python
def bubbleSort(nums):
    n = len(nums)
    
    has_sorted = False
    for i in range(n - 1): # loop passes
        if has_sorted:
            break

        for j in range(n - i - 1):            
            if nums[j] > nums[j+1]:
                nums[j], nums[j+1] = nums[j+1], nums[j]
                has_sorted = False
            else:
                has_sorted = True

    return nums        

```



Now the complexity of the enhanced Bubble Sort for the bset case is $O(n)$ while the worset case still stays the same.





## Selection Sort



## Insertion Sort



## Quick Sort



## References

- [Problem Solving with Algorithms and Data Structures using Python](https://runestone.academy/runestone/books/published/pythonds/SortSearch/TheQuickSort.html)

