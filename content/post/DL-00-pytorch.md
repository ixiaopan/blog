---
title: "Deep Learning - PyTorch"
date: "2021-07-19"
description: ""
# tags: []
categories: [
    "Deep Learning",
]
series: ["Machine Learning"]
katex: true
---

 

The most popular deep learning frameworks so far are Tensorflow and PyTorch. Well, during my study, I use PyTorch more often. Recently, I am building the classic BiLSTM-CRF model using PyTorch. It's a bit hard for me when operating matrices since it uses various advanced techniques about indexing and slicing. I think it's necessary to explain these amazing functions. Therefore, I am going to write this post to revisit the most important aspects of PyTorch for future references.



<!--more-->



## Tensor



Tensor is the basic data structure in PyTorch. Simply put, a tensor means a multi-dimensional array where you can access an individual element by indexing. The dimension could be zero, one, two, three and so on. The most common tensors are scalars, vectors, and matrices shown below.



```python
torch.tensor(1.) # scalar

torch.tensor([1, 2, 3]) # vector

torch.arange(12).reshape(3, 4) # matrix
torch.arange(12).reshape(2, 3, 2) # matrix
```



### storage

In fact, the tensor declared above is just a view of the underlying data. View means a kind of way to look at data. For example, you can look at an image from the top direction or the right direction. In the case of data, you could look at data in the original order or you could skip a fixed number of elements. 

No matter how you view it, the data in memory stay the same. In fact, the real values are stored in a contiguous block of memory. In PyTorch, we can access it by invoking `tensor.storage()`. For example,



```python
x = torch.tensor([[0, 1, 2], [3, 4, 5]])
x.storage()
```

The result is shown below. You might find that the elements are sorted along the rows of the tensor $x$. Tthe difference is that they are stored in a one-dimensional array.



![](/blog/post/images/torch_storage.png "Figure 1: The underlying data beneath the tensor x")



Here comes a question: How does the indexing operation $x[0, 1]$ work?



### stride

In oder to index an element, PyTorch needs to know meta data about a tensor, such as stride, which indicates the number of elements to be skipped along each dimension. In the above example, the stride is



```python
x.stride()
# (3, 1)
```



$3$ means we need to skip 3 elements to get to the next row, 1 means we just move one step so as to reach the next column. Mathematically, the indexing operation in 2D tensor are described as follows, where offset is usually zero


$$
x[i, j] = i * \text{stride}[0]  + j * \text{stride}[1] + \text{offset}
$$


Why is it designed like this? It's less expensive for some operations like transpose since there is no need to reallocate memory space. Instead, we just modify the stride.



For instance, $x^T$ is shown below

```
0 3 
1 4
2 5
```



If we look at the underlying data of $x^T$, we will find that $x^T$ and $x$ have the same storage shown in Figure 1. 



```python
x.data_ptr() == xt.data_ptr() # True
```



Let's have a look at the stride of $x^T$



```python
xt.stride()
#(1, 3)
```



In this case, we simply move one step forward if we want to reach the next row while we need to move 3 steps to get to the next column.



Apart from this, knowing how storage and stride work will also help understand other PyTorch functions' behaviour. [This article](https://zhang-yang.medium.com/explain-pytorch-tensor-stride-and-tensor-storage-with-code-examples-50e637f1076d) discusses the difference between `torch.expand()` and `torch.repeat()`. Accoding to the PyTorch document



> torch.expand()
>
> ​	Expanding a tensor does not allocate new memory, but only creates a new view on the existing tensor where a dimension of size one is expanded to a larger size by setting the `stride` to 0. Any dimension of size 1 can be expanded to an arbitrary value without allocating new memory.
>
> torch.repeat()
>
> ​	Unlike expand(), this function copies the tensor’s data.



```python
x = torch.arange(3)
x.stride() # 1

y = x.expand(2, 3)
y.stride() # (0, 1)

z=x.repeat(2, 1)
z.stride() # (3, 1)

```



From the results of strides, we can see that `torch.tensor.expand()` does not require extra memory space. This is essential when working with large data set.



### contiguous

In Pytorch, there is a concept called **contiguous**, indicating whether a tensor has the same values as the storage when counting from the innermost dimension. In other words, when moving along the rows in 2D tensors, the values sorted in this way are exactly the order of the underlying data. Visually, such an order is more comfortable and consistent.

For example, if we flatten $x$ along the innermost dimension, we will get 
$$
0, 1, 2, 3, 4, 5
$$


If we flatten $x^T$ along the innermost dimension, we will get 


$$
0, 3, 1, 4, 2, 5
$$


which is different from the storage shown in Figure 1. Therefore, we say $x$ is contiguous while $x^T$ is not. We can also check it in Pytorch shown below.

```python
x.is_contiguous() # True
xt.is_contiguous() # False
```



#### Why

But wait, why do we need contiguous tensor?

One reason is that we can exploit the benefit of [cache](https://stackoverflow.com/questions/16699247/what-is-a-cache-friendly-code/16699282#16699282) to improve the speed of fetching data. In short, when fetching an element of a matrix, we can also get the neighboring elements at the same time, requiring less memory accesses.

Another reason is that some functions only work in the contiguous tensor, such as `view()`

```python
xt.view(1, 6)
# RuntimeError: view size is not compatible with input tensor's size and stride (at least one dimension spans across two contiguous subspaces). Use .reshape(...) instead.
```



There are two approaches to fix it: call `contiguous()` to make the tensor contiguous or call `reshape()`



```python
xt.contiguous().view(1, 6)
xt.reshape(1, 6)
```



However, `reshape()` could return a view (if the tensor is contiguous) or a new tensor (if not). 



#### How

How do we make $x^T$ contiguous too? 



```python
xtc = xt.contiguous()
```



![](/blog/post/images/x_T.png "Figure 2: The underlying data beneath the tensor $x^T$")



Figure 2 shows the new storage. Thus, `contiguous()` reallocates a new memory and copy the original tensor into that memory space. We can check it using the code below.

```python
xt.data_ptr() == xtc.data_ptr() # False
```



Meanwhile, the stride has also been changed to adapt to this new storage. 



```python
xT.stride()
# (2, 1)
```





## torch.nn



### nn.module

`torch.nn` is the submodule that contains everything needed to build neural networks in PyTorch. We know that a neural network is composed of a stack of layers. In PyTorch, we call these layers **modules** and they are all subclasses of `nn.Module`. 

I think we call them modules because a single layer is a layer while multiple layers can also be considered as a big layer. Thus, we refer to both of them as modules regardless how many layers there are. Moreover, a module can also have other submodules (subclasses of `nn.Module`) as their attributes and thus track their parameters automatically. 

When I built the BiLSTM-CRF for NER, I split LSTM and CRF apart. However, I noticed that the BiLSTM model can still retrieve the parameters of the CRF layer. Now I found the reason.

First, both the `nn.Linear()` and `CRF()` are moules in PyTorch. Second, when the instance of such a module is assigned to an attribute of another `nn.Module` ( in this case, it's BiLSTM ), this module will be automatically registered as the submodule of  `BiLSTM`, and thus `BiLSTM` have access to the parameters of its submodules.



```python
class CRF(nn.Module):
  def __init__(self):
		super(CRF, self).__init__()


class BiLSTM(nn.Module):
  def __init__(self):
    super(BiLSTM, self).__init__()
    self.fc = nn.Linear(10, 2)
  	self.crf = CRF(self.num_of_tag)

```



### forward



When I first used PyTorch, I often confused why this model doesn't invoke `forward()`. For example, I have a simple forward network, I have seen two options shown below to move forward.



```python
self.fc = nn.Linear(5, 4)

self.fc(inputs)  # option 1, the right way
self.fc.forward(inputs) # option 2, wrong
```



Well, the reason is simple. In fact, the built-in PyTorch subclasses of `nn.Module` allows themselves to be called as a simple function call. In the `__call__`, it calls `forward()`. So what's the difference? The difference is that PyTorch will do other operations by provided hooks before calling `forward()`. Thus, it's possible to call `forward()` directly, but it shouldn't do this unless we don't provide any hooks. ( More details can be seen [here](https://github.com/pytorch/pytorch/blob/master/torch/nn/modules/module.py#L1057) )



### parameter

We have built our models, so how to access to the parameters? PyTorch provides `parameters()` function, which allows us to collect all parameters from the first layer to the last layer.



```python
for param in model.parameters():
	print(param)
```



However, it's a bit hard to distinguish them when you have many layers. Thus, PyTorch provides another function `named_parameters()`



```python
for name, param in model.named_parameters():
	print(name, param)
```



With the name of each layer, it's easy to retrieve individual parameter by accessing the corresponding name directly, such as

```python
model.out_linear.weight
model.out_linear.bias
```

 

So far so good. But where are the names from?



### sequential

PyTorch provide two ways to create neural networks. The first one is to use `nn.Sequential()`, as shown below



```python
model = nn.Sequential(
  nn.Linear(5, 4),
  nn.Tanh(),
  nn.Linear(4, 2)
)

'''
Sequential(
  (0): Linear(in_features=5, out_features=4, bias=True)
  (1): Tanh()
  (2): Linear(in_features=4, out_features=2, bias=True)
)
'''
```



Well, the names in this example are just numbers. It would be fine if there are few layers. To build a more semantic model structure, we can pass name for each layer,



```python
from collections import OrderedDict

model = torch.nn.Sequential(OrderedDict([
  ('hidden_layer', torch.nn.Linear(5, 4)),
  ('active_func', torch.nn.Tanh()),
  ('out_layer', torch.nn.Linear(4, 2))
]))

model

'''
Sequential(
  (hidden_layer): Linear(in_features=5, out_features=4, bias=True)
  (active_func): Tanh()
  (out_layer): Linear(in_features=4, out_features=2, bias=True)
)
'''
```



As mentioned earlier, we can retrieve the parameters for an individual layer by accessing its name, such as



```python
model.hidden_layer.bias
'''
Parameter containing:
tensor([ 0.1177, -0.2876, -0.3861, -0.3367], requires_grad=True)
'''
```



Though `nn.Sequential()` is handy, the order of layers is fixed. That's where `nn.Module` comes into play. We rewrite the above code using `nn.Module`



```python
class SimpleNet(nn.Module):
	def __init__(self):
    super(SimpleNet, self).__init__()

    self.hidden_layer = nn.Linear(5, 4)
    self.out_layer = nn.Linear(4, 2)
   
  def forward(self, inputs):
  	outs = self.hidden_layer(inputs)
  	outs = torch.tanh(outs)
  	outs = self.out_layer(outs)
  	return outs
  
net = SimpleNet()

'''
SimpleNet(
  (hidden_layer): Linear(in_features=5, out_features=4, bias=True)
  (out_layer): Linear(in_features=4, out_features=2, bias=True)
)
'''
```



## Advanced Techniques



### torch.scatter_

This function is often used to implement one-hot encoding. As its name suggests, **scatter** means to distribute a list of values over another tensor.

In the case of one-hot encoding, the matrix is a parse matrix whose element value is either zero or one. So one is the scalar to be scattered at the corresponding column index for each observation.



```python
label = torch.tensor([2, 3, 4, 0]).view(4, 1)
one_hot = torch.zeros(len(label), 5)
one_hot.scatter_(1, label, 1)
'''
tensor([[0., 0., 1., 0., 0.],
        [0., 0., 0., 1., 0.],
        [0., 0., 0., 0., 1.],
        [1., 0., 0., 0., 0.]])
'''

```

Further, the scatterd item could be any multi-dimensional array rather than a simple scalar, for example,

```python
labels = torch.tensor([[2, 3], [4, 0], [1, 5]]) # 3 observations
one_hot = torch.zeros(len(labels), 6).long() # 6 labels in total
source = torch.arange(1, 7).long().view(3, 2)

one_hot.scatter_(1, labels, source)
'''
tensor([[0, 0, 1, 2, 0, 0],
        [4, 0, 0, 0, 3, 0],
        [0, 5, 0, 0, 0, 6]])
'''
```

From the above examples, we conclude that 

- `labels` and `source` should have the same size along the specified dimension (the first argument)
- All tensors (one_hot, lables, source) should keep the same shape except the specified dimension



### torch.squeeze()

`squeeze()` and `unsqueeze()` often appear in pairs to expand dimension of a tensor for broadcasting.

- `squeeze()` remove all dimensions with the value of 1

  ```python
  
  x = torch.arange(8).view(1,2,4)
  '''
  tensor([[[0, 1, 2, 3],
           [4, 5, 6, 7]]])
  '''
  
  x.squeeze(0)
  '''
  tensor([[0, 1, 2, 3],
          [4, 5, 6, 7]])
  '''
  ```

  

- `unsqueeze()` add one dimension along the specified dimension

  ​	

  ```python
  x = torch.tensor([1, 2, 3])
  
  x.unsqueeze(dim=0) # tensor([[1, 2, 3]])
  x.unsqueeze(dim=1) 
  '''
  tensor([[1],
          [2],
          [3]])
  '''
  ```



### torch.gather()

I happened to meet this function when building BiLSTM+CRF NER models. It was hard to implement batch training for CRF layers until I found `torch.gather()`. Well, it's a bit similar to `torch.scatter_()` except that `torch.gather()` aims to fetch data while `toch.scatter_()` is used to write values.



For BilSTM+CRF models, we need to calculate the sentence score as follows,


$$
\text{Score} (D_j) = \sum_{i=0}^{|D_j|} T(y^{w_i} \rarr y^{w_{(i+1)}}) + E(w_{i+i}|y^{w_{(i+1)}})
$$


where emission scores are the outputs of BiLSTM and the transition scores are the parameters of the CRF layer. For each batch training, we need to calculate each sentence score in this batch.

```python
emission score: 
	(batch_size, max_seq_len, num_of_tag)
	[
		[
                   O   B   I
            [ w00 0.1 0.2 0.7 ]
            [ w01 0.2 0.4 0.4 ]
            ...
		], # sentence 1

		[
                  O   B   I
            [ w10 0.1 0.2 0.7 ]
            [ w11 0.2 0.4 0.4 ]
            ...
		] # sentence 2
	]
tags: (batch_size, max_seq_len)
	[ 
		[ w00=>B, w01=>I, ...], # sentence 1
		[ w10=>B, w11=>O, ...], # sentence 2
            ...
	]
```

It's easy to fetch data from the specified index along the desired dimension using `torch.gather()`. In this case, we want to fetch data from the index that the true tag of each word belongs to along the innermost dimension of emission_score. For example, sentence 1 has two words with the labels `B` and `I`, so the corresponding scores in the emission score are `emission_score[0][0][1]` and `emission_score[0][1][2]`.



```python
emission_score = torch.tensor([
    [

        [  0.1, 0.2, 0.7 ],
        [  0.2, 0.3, 0.5 ]

    ], 

    [

        [ 0.35, 0.25, 0.4 ],
        [ 0.6, 0.25, 0.15 ]

    ]
])
true_labels = torch.tensor([ 
    [ 1, 2],
    [ 1, 0],
    
])

torch.gather(emission_score, 2, true_labels.unsqueeze(-1)).squeeze(-1).sum(-1)
'''
tensor([[0.2000, 0.5000],
        [0.2500, 0.6000]]) => tensor([0.7000, 0.8500])
'''
```



which is exactly the second term of $\text{Score} (D_j) $.



## References

- https://zhang-yang.medium.com/explain-pytorch-tensor-stride-and-tensor-storage-with-code-examples-50e637f1076d



