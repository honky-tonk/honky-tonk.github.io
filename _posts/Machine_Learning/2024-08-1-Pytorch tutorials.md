---
layout: post
title: Pytorch tutorials.md
subtitle: 
gh-repo: honky-tonk/honky-tonk.github.io
gh-badge: [star,follow]
tags: [Stat]
comments: true
---

<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>


# Why pytorch
pytorch is widely use in machine learning and other field, so this why i learning this tool 

# How to Create a tensor?

you could input a list to ```torch.tensor``` for create new tensor
```
>>> t1 = torch.tensor([1,2,3,4,5,6,7,8,9])
>>> t1
tensor([1, 2, 3, 4, 5, 6, 7, 8, 9])
>>>
```

you could create a tensor with **random int**(which follow by normal distribution)

```
>>> torch.randn(3,2,1)
tensor([[[-1.5739],
         [-1.0333]],

        [[ 0.7810],
         [ 0.8433]],

        [[ 2.4181],
         [ 1.2268]]])
>>>
```
> in torch.randn(3,2,1) we create 3 matrix, each matrix has 2 row 1 col

we also can create a tensor by order(1, 2, 3, ..., n)
```
>>> torch.arange(3)
tensor([0, 1, 2])
>>>
>>> torch.arange(0, 5, step = 2) # from 0 to 5(not include 5), and step is 2
tensor([0, 2, 4])
>>>
```

# how to check tensor storage?
we could check tensor storage
```
>>> t1 = torch.randn(3,2,1)
>>> t1
tensor([[[1.2661],
         [0.5274]],

        [[0.2601],
         [0.9098]],

        [[1.4194],
         [0.2295]]])
>>> t1.storage()
 1.2661268711090088
 0.5274471044540405
 0.2600526809692383
 0.909845769405365
 1.4194166660308838
 0.2295401394367218
[torch.storage.TypedStorage(dtype=torch.float32, device=cpu) of size 6]
```
in pytorch, we colud check if tensor is contiguous
```
>>> t1.is_contiguous()
True
```
in torch memory storage and the way of read tensor is different, liek t1 above, the way read storage in memory is follow by order, so t1 is **contiguous**, but if we change (3,2,1) tensor to (3, 2) matrix then we transipose this, we will find they are same in sotrage, but the way of read is different
```
>>> t1 = torch.randn(3,2,1)
>>> t1
tensor([[[1.2661],
         [0.5274]],

        [[0.2601],
         [0.9098]],

        [[1.4194],
         [0.2295]]])
>>> t1.storage()
 1.2661268711090088
 0.5274471044540405
 0.2600526809692383
 0.909845769405365
 1.4194166660308838
 0.2295401394367218
>>> t1.view(3,2) # change (3,2,1) to (3,2)
tensor([[1.2661, 0.5274],
        [0.2601, 0.9098],
        [1.4194, 0.2295]])
>>> t1.view(3,2).T
tensor([[1.2661, 0.2601, 1.4194],
        [0.5274, 0.9098, 0.2295]])
>>> t1.view(3,2).T.storage() # same as t1 in memory
 1.2661268711090088
 0.5274471044540405
 0.2600526809692383
 0.909845769405365
 1.4194166660308838
 0.2295401394367218
>>> t1.view(3,2).T.is_contiguous()
False
```

# matrix or vector operation
multiply two vector
```
>>> t1 = torch.tensor([1,2,3]).view(3,1)
>>> t1
tensor([[1],
        [2],
        [3]])
>>> t2 = torch.tensor([1,2,3])
>>> t2
tensor([1, 2, 3])
>>> t2 @ t1
tensor([14])
```

add two vector
```
>>> t1 = torch.tensor([3,4,5])
>>> t2 = torch.tensor([1,1,1])
>>> t1 + t2
tensor([4, 5, 6])
```
all vector multiply a number
```
>>> t2 = torch.tensor([1,1,1]) * 3
>>> t2
tensor([3, 3, 3])
```

multiply  col vector and row vector to a matrix
```
>>> t1 = torch.tensor([1,2,3]).view(3,1)
>>> t2 = torch.tensor([4,5,6]).view(1,3)
>>> t1 @ t2
tensor([[ 4,  5,  6],
        [ 8, 10, 12],
        [12, 15, 18]])
>>>
```
> t2 = torch.tensor([4,5,6]) is not (1,3) but (3,) so t2 can not multiply with t1 directly

we could operation a matrix like sum, mean etc...
```
>>> t2
tensor([[ 4,  5,  6],
        [ 8, 10, 12],
        [12, 15, 18]])
>>> t2.sum(dim=1)
tensor([15, 30, 45])
>>> t2.sum(dim=0)
tensor([24, 30, 36])
```
> dim=1 is sum by row(sum each row), dim=0 is sum by col(sum each col) in case above, but dim=1 actually erase dim 1, t2's shape is (3,3) dim 0 mean 3(first 3 of shape), dim 1 also mean 3(last 3 of shape), so the dim = 1 in sum() mean erase the shape of last 3( last 3 mean row) so we will sum all row

same in matrix

```
>>> t1 = torch.randn(3,2,2)
>>> t1
tensor([[[-1.3154, -0.8733],
         [ 2.0236, -1.7698]],

        [[ 1.5449, -0.4286],
         [ 0.2091,  0.7474]],

        [[ 1.4878, -0.6629],
         [ 1.9587,  0.3707]]])
>>> t1.sum(dim=1)
tensor([[ 0.7082, -2.6431],
        [ 1.7540,  0.3188],
        [ 3.4465, -0.2922]])
```
ti's shape is (3, 2, 2), dim = 1 mean the second number of shape(2 in middle, also is first 2 in shape), the first 2 mean all col in each matrix, so t1.sum(dim=1) mean add all col in each matrix 
# 