---
title: "Leetcode - 102"
date: "2021-09-10"
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



## Traverse a Tree



Three strategies to visit a tree depends on the order of visiting the root

- pre-order
- in-order
- post-order



Two methods to implement this

- Recursive
- Iterative
  - The idea is first-in-last-out, so we use stack to keep nodes.



### Pre-order Traversal



```python
class Solution:
    def preorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        if root is None:
          return []

        # recursive
        l_n = self.preorderTraversal(root.left)
        r_n = self.preorderTraversal(root.right)
        return [ root.val ]  + l_n + r_n
      
      	# iterative
      	val = []
        stack = [ root ]
        while len(stack) > 0:
          node = stack.pop()
          val.append( node.val )

          if node.right:
            stack.append( node.right )
          
          if node.left:
            stack.append( node.left )
        return val
```





### In-oder Traversal 

```python
class Solution:
    def inorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        if root is None:
          return []
        
        # recursive
        l_n = self.inorderTraversal(root.left)
        r_n = self.inorderTraversal(root.right)
        return l_n + [ root.val ] + r_n
      
      	# iteratively
        stack = []
        val = []
        while True:
          while root:
            stack.append(root)
            root = root.left

          if len(stack) == 0:
            return val

          # once end, it must be the local root node without left children
          # then we pop it out to visit this node
          # next we visit its right child
          # if the right child is None, we pop up its parent node
          node = stack.pop()
          val.append(node.val)
          root = node.right
```





### Post-order Travesal

```python
class Solution:
    def postorderTraversal(self, root: Optional[TreeNode]) -> List[int]:
        if root is None:
            return []
        
        # recursive
        l_n = self.postorderTraversal(root.left)
        r_n = self.postorderTraversal(root.right)
        return l_n + r_n + [ root.val ]
```





## Level-order Traversal



### BFS

Breadth-First Search visit nodes level by level. We use a queue to keep nodes. 

- Each time we are goingt to visit a node, we pop it out of the queue
- At the same time, we add its left node and right node into the queue



![](/blog/post/images/BFS-queue.png#full "Figure 1: We use queue to store nodes in BFS ($\text{LeetCode}^{[1]}$)")



```python
def BFS(root):
  if root is None:
  	return []

  ret = []
  queue = [ root ]
  while len(queue) > 0:
    ret.append( [ n.val for n in queue ] )
   
  	next_level = []
    for n in queue:
    	next_level += [ n.left, n.right ] 

    queue = [ n for n in next_level if n ]    	
  
```



### DFS



## Tree problem



### Top-down Solution

- visit a node and get some values
- pass the values to its children



```python
return specific value for null node
left_ans = top_down(root.left, some_val_of_node)
right_ans = top_down(root.right, some_val_of_node)
```



### Bottom-up Solution

- get the answer from the children nodes
- then update the answer for the parent node



```python
class Solution:
    def maxDepth(self, root: Optional[TreeNode]) -> int:
        if root is None:
            return 0

        left_depth = self.maxDepth(root.left)
        right_depth = self.maxDepth(root.right)
        return max(left_depth, right_depth) + 1
```





## Reference

- [[1] Binary Tree - LeetCode](https://leetcode.com/explore/learn/card/data-structure-tree/134/traverse-a-tree/992/)

