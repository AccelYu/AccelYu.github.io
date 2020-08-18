---
layout: post
title: Python深度学习（一），流程
categories: [Python]
mathjax: true
---

深度学习的大致流程

<!-- more -->
## 概念
1. 线性问题：低维的线性不可分可转换为高维的线性可分

2. 大致过程：前向传播通过激活函数获取输出，损失函数计算损失，
达到条件则停止，否则后向传播更新参数从而减小损失，再次前向传播

## 模型
1. 线性模型(Liner Regression)：连续值的预测

2. 二分类模型(Logistic Regression)：判断预测，输出1或2个值

3. 多分类模型(Classification)：分类预测

4. 生成模型(Generative Model)：生成不包含在训练数据集中的新数据

## 标准化数据(normalization)
1. 归一化：$x_i=\frac{x_i-x_{min}}{x-x_{max}}$

2. BN(Batch normalization)：
   1. 将当前数据转换为标准正态分布，进行数据归一化
   2. $x_i^\prime=\gamma x_i + \beta$，还原数据分布

## 激活函数
为网络提供非线性建模能力
1. sigmoid：$f(x)=\frac{1}{1+e^{-x}}$将数据压缩到0到1之间

2. tanH：$f(x)=\frac{e^x-e^{-x}}{e^x+e^{-x}}$将数据压缩到-1到1之间

3. relu：$f(x)=\begin{cases}0,&\text{x<0}\\\x,&\text{x>0}\end{cases}$  
激活函数之前使用BN算法的必要性：sigmoid和tanH等两端趋于饱和的激活函数需要$\gamma$进行数据缩放，缓解梯度弥散；
relu等线性转折类激活函数需要$\beta$进行偏置，否则会出现均值为固定值的现象

## 损失函数
2. 均方差：一般在meta-learning中使用

3. 交叉熵：一般在分类问题中使用

## 优化方法
1. 梯度下降法：标准梯度下降（GD）、批量梯度下降（BGD）、随机梯度下降（SGD）

2. 动量优化法：Momentum、NAG

3. 自适应学习率优化法：AdaGrad、RMSProp、AdaDelta、Adam