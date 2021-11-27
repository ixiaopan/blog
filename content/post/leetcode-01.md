---
title: "Leetcode - 101"
date: "2021-09-06"
description: ""
# tags: []
categories: [
    "Utilities",
]
series: ["Computer Science"]
katex: true
---



Practice! Practice! Practice!



<!--more-->



## Math



### Reverse an integer

- the key is to multiply each remainder by 10 in each division
- $a\*10\*10+b*10+c$



```python
reverse_x = 0

while x > 0:
  mod_x = x % 10 # mod operation
  x = x // 10  # floor division  
  # reverse
  reverse_x = reverse_x * 10 + mod_x
```



### Plus one

- if $d==9$  ( $v==0$ ), we need to increment 1 for the next digit
- otherwise we are done

```python
def plusOne(digits):
  d = digits[-1]
  v = (d + 1) % 10 
  if v == 0:
    return self.plusOne(digits[:-1]) + v
  else:
    digits[-1] = v
    return digits
```



### Binary addition 



- elementwise addition between numbers with variable length
- addition starts from right to left

```python
inc = 0
while l >= 0 and r >= 0: # a,b should have the same length
  t = a[l] + b[r] + inc
  v = t % 2 # the result of the current position
  if t >= 2:
    inc = 1
  else:
    inc = 0
  l -= 1
  r -= 1

# which is equivalent to 
inc = 0
while l >= 0 or r >= 0 or inc > 0:
  if l >= 0: # no need to ensure that len(a) == len(b)
    inc +=a[l]
    l -= 1

  if r >=0:
    inc += b[r]
    r -= 1

  v = inc % 2
  inc = inc // 2
```



### The Euclidean Algorithm



```python
# find the greatest common divider of A and B
def findGCD(A, B):
  while B > 0:
    A, B = B, A % B
  return A
```



## Array



### The second largest number

```python
# the largest number is unique
max_v, sec_max_v = -999, -999
for n in nums:
	if n > max_v:
		sec_max_v = max_v
		max_v = n
	
	elif n > sec_max_v:
		sec_max_v = n
```





## String





## Two pointer approach

### The same direction

- one slow-runner and one fast-runner, $+=1$
- the slow-runner points at the placeholders
- the fast-runner is used to loop each element
- the key is to determine the movement strategy



```python
def removeDuplicate(nums):
  start = 0
  end = 1

  while end < len(nums):
    if nums[end] == nums[start]:
      end += 1
    else:
      if end - start != 1:
        nums[start + 1], nums[ end ] = nums[ end ], nums[start + 1]

      start += 1
      end += 1

  return start + 1

```



### The opposite direction

- the left pointer $+=1$
- the right pointer $-=1$
- This technique is often used in a sorted array



```python
def reverseString(s):
  l, r = 0, len(s) - 1
  while l < r:
    s[l], s[r] = s[r], s[l]
    l += 1
    r -= 1
  return s
```



Two sum (sorted array)

```python
def twoSum(nums):
	l, r = 0, len(nums) - 1
	while l < r:
		t = nums[l] + nums[r]
		if t > target:
			r -= 1
		elif t < target:
			l += 1
		else:
			return [l, r]
```





## Binary Search

An effective way to find an element in a **sorted array**.

- if not sorted, we can always sort the collection and apply binary search
- divide search space in two spaces each time
- converge at some position



### Template 1

- basic form of binary search
- Loop ends when the two pointers meet
- $ \text{left} <= \text{right} $



```python
def findElement(nums, target):
  i, j = 0, len(nums) - 1

  while i <= j:
    # mid = (i + j) // 2
    mid = i+ (j - i) // 2 # better in this way due to overflow
    if target > nums[mid]:
      i = mid + 1
     elif target < nums[mid]:
      j = mid - 1
     else:
      return mid

  return -1
```



### Template 2

- Loop ends when there is only one element left
- search left: $right = mid$
- search right: $ left =  mid + 1$
- $ \text{left} < \text{right}$



```python
class Solution:
    def firstBadVersion(self, n, pick):
        """
        :type n: int
        :rtype: int
        """
        l, r = 1, n
        while l < r:
          mid = (l+r)//2
          if isBadVersion(mid, pick) is False: # search right
            l = mid + 1
          else: # search left
            r = mid

        # if l <= n and isBadVersion(l, pick) is True:
        return l

        # return -1
```



### Template 3







## Hash Table

Keep the corresponding value of a specified key in advance

Two sum

- if the next value is the desired value, then it must be in the dict
- otherwise, we update the dict



```python
hashmap = {}
for i in range( len(nums) ):
	if nums[i] not in hashmap:
		hashmap[  target - nums[i] ] = i
	else:
		return [ hashmap[ nums[i] ], i ]
```





## Refer



- [Array and String - LeetCode](https://leetcode.com/explore/learn/card/array-and-string/)
- [Binary Search - LeetCode](https://leetcode.com/explore/learn/card/binary-search/138/background/974/)
- [The Euclidean Algorithm](https://www.khanacademy.org/computing/computer-science/cryptography/modarithmetic/a/the-euclidean-algorithm)

