---
title: "Deep Learning - Softmax"
date: "2021-06-21"
description: ""
# tags: []
categories: [
    "Deep Learning",
]
series: ["Machine Learning"]
katex: true
draft: true
---



From now on, 



<!--more-->





## Linear Regression



```python
# lr = 0.001
lr = 0.01
epoches = 5000

def model(w, b, x):
    return w * x + b

def loss_fn(y_true, y_pred):
    ''' means squared error'''
    return ((y_true - y_pred)**2).mean()

def grad_w(x, y_true,  y_pred):
    return ((y_pred - y_true) * x).mean()

def grad_b(y_true,  y_pred):
    return (y_pred - y_true).mean()

def grad_param(x, y_true, y_pred):
    dw = grad_w(x, y_true,  y_pred)
    db = grad_b(y_true,  y_pred)
    return torch.stack([dw, db])

def training(x, y, params, epoches, lr):
    for epoch in range(epoches):
        w, b = params

        y_pred = model(w, b, x)
        loss = loss_fn(y, y_pred)
        grad = grad_param(x, y, y_pred)

        params = params - lr * grad
        
        if epoch % 1000 == 0:
            print('epoch: ', epoch, 'loss: ', loss.item())
            print('[w, b]:', params.numpy())
    return params

params = training(x, y, torch.tensor([1., 0.]), epoches, lr)
# params = training(x*0.1, y, torch.tensor([1., 0.]), epoches, lr)

```



learning rate



gradient



## Logistic Regression







## Binary Cross-Entropy







## Softmax



## References



- Pytorch 

