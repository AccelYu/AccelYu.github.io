---
layout: post
title: Python深度学习（三），Pytorch
categories: [Python]
mathjax: true
---

PyTorch很简洁、易于使用、支持动态计算图而且内存使用很高效

<!-- more -->
## 常用API
```python
import torch
import numpy as np

a = np.array([
    [1, 2, 3],
    [4, 5, 6]
])
# 1.numpy类型转tensor
a = torch.from_numpy(a)
# 获取tensor的形状
print(a, a.shape, a.size())

# 设置全局默认tensor类型
torch.set_default_tensor_type(torch.DoubleTensor)
# 2.以数据生成tensor，指定dtype
b = torch.tensor([1, 2], dtype=torch.float32)
# 获取tensor的type
print(b.type())

# 3.从0到1均匀初始化
print(torch.rand(2, 2))
# 以b的形状及类型初始化
print(torch.rand_like(b))

# 4.从low到high，均匀取整数，形状为size
print(torch.randint(low=1, high=10, size=[2, 2]))

# 5.标准正态分布
print(torch.randn([2, 2]))
# 正态分布
print(torch.normal(mean=0, std=1, size=[2, 2]))

# 6.全部为fill_value
print(torch.full([2, 2], fill_value=1.))

# 7.等差数列[start,end)
print(torch.arange(start=0, end=10, step=2))

# 8.等分点[start,end]
print(torch.linspace(start=0, end=10, steps=4))

# 9.随机打散
print(torch.randperm(10))
```

## 数据加载
```python

```