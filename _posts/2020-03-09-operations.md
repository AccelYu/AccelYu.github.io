---
layout: post
title: Python数据基本运算
categories: [Python]
---

python数据类型注意点

<!-- more -->
## 注意要点
1. None  
作用：变量解除绑定或用来占位  
在if判断中其结果为false

2. int  
CPython中整数-5 到 256，永远存在小整数对象池中，不会释放

3. del  
作用：删除变量