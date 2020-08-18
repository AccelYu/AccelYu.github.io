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

### 模型
1. 线性模型(Liner Regression)：连续值的预测

2. 二分类模型(Logistic Regression)：判断预测，输出1或2个值

3. 多分类模型(Classification)：分类预测

4. 生成模型(Generative Model)：生成不包含在训练数据集中的新数据

### 标准化数据(normalization)
1. 归一化：$x_i=\frac{x_i-x_{min}}{x-x_{max}}$

2. BN(Batch normalization)：
   1. 将当前数据转换为标准正态分布，进行数据归一化
   2. $x_i^\prime=\gamma x_i + \beta$，还原数据分布

### 激活函数
为网络提供非线性建模能力
1. sigmoid：$f(x)=\frac{1}{1+e^{-x}}$将数据压缩到0到1之间

2. tanH：$f(x)=\frac{e^x-e^{-x}}{e^x+e^{-x}}$将数据压缩到-1到1之间

3. relu：$f(x)=\begin{cases}0,&\text{x<0}\\\x,&\text{x>0}\end{cases}$  
激活函数之前使用BN算法的必要性：sigmoid和tanH等两端趋于饱和的激活函数需要$\gamma$进行数据缩放，缓解梯度弥散；
relu等线性转折类激活函数需要$\beta$进行偏置，否则会出现均值为固定值的现象

### 损失函数
2. 均方差：一般在meta-learning中使用

3. 交叉熵：一般在分类问题中使用

### 优化方法
1. 梯度下降法：标准梯度下降（GD）、批量梯度下降（BGD）、随机梯度下降（SGD）

2. 动量优化法：Momentum、NAG

3. 自适应学习率优化法：AdaGrad、RMSProp、AdaDelta、Adam

## 网络
### 卷积神经网络(CNN)
CNN每一次Conv2D操作都是权值共享的，大大减少了参数量，而且还保留了数据的结构信息  

ResNet(Residual Neural Network)：
   1. BasicBlock  
   $Conv2D(可降维) \to Activation  \to Conv2D \to \cdots$
   
   2. ResBlock  
   $x_1=[x_0(可降维) +BasicBlock] \to Activation$  
   $x_2=[x_1(可降维) +BasicBlock] \to Activation$  
   $\cdots$
   
   3. ResNet  
   $ResBlock \to ResBlock \to \cdots$
   
   ResNet中BasicBlock拟合$f(x)$，ResBlock拟合$H(x)$，$f(x)=H(x)-x$，所以$f(x)$是一个残差块。  
   即使$f(x)$再差也能拟合到对loss影响为0，只要有一丝提升就是赚到，而且$x$贯穿网络，缓解了梯度弥散，所以理论上残差块可以无限叠加。

### 循环神经网络(RNN)  
RNN每一个时间步上权值共享，大大减少了参数量

1. LSTM(Long Short-Term Memory)：
   1. 遗忘门  
   $f_t = \sigma (W_f[h_{t-1},x_t] + b_f)$，遗忘的逻辑（门操作）
   
   2. 输入门  
   $i_t = \sigma (W_i[h_{t-1},x_t] + b_i)$，更新的逻辑（门操作）  
   $\widetilde{C_t} = tanh (W_C[h_{t-1},x_t] + b_C)$，候选内容
   
   3. 输出门  
   $o_t = \sigma (W_o[h_{t-1},x_t] + b_o)$，输出的逻辑（门操作）  
   $C_t = f_t \cdot C_{t-1} + i_t \cdot \widetilde{C_t}$，遗忘部分之前内容，更新部分候选内容  
   $h_t = o_t \cdot tanh(C_t)$，最终输出
   
   $C_t$始终贯穿整个网络，远处信息与近处信息相当于同权相加，所以远处信息不会被稀释。
   而且梯度相当于是累加的，缓解了梯度弥散的问题。

2. GRU(Gated Recurrent Unit)：
   1. 重置门  
   $r_t = \sigma (W_r[h_{t-1},x_t] + b_r)$，重置的逻辑（门操作）
   2. 更新门  
   $z_t = \sigma (W_z[h_{t-1},x_t] + b_z)$，更新的逻辑（门操作）  
   $\widetilde{h_t} = tanh (W[r_t \cdot h_{t-1},x_t])$，重置之前内容，再加上当前内容，得出候选内容  
   $h_t = (1-z_t) \cdot h_{t-1} + z_t \cdot \widetilde{h_t}$，遗忘部分之前内容，更新部分候选内容，最终输出  
   
   $h_t$始终贯穿整个网络，相当于LSTM中的$C_t$
   
### 对抗生成网络

### 自编码器(Auto-Encoders)
观测样本 $x\to$ 隐含变量 $z\to$ 生成样本 $\hat x$
1. AAE(Adversarial Auto-Encoder)：
   
2. VAE(Variational Auto-Encoder)：
   1. $q(z \vert x)$变分函数代替后验概率$p(z \vert x)$，$q(z \vert x)$越接近$p(z \vert x)$，则$KL(q(z \vert x) \Vert p(z \vert x))$越小。  
   由贝叶斯展开$KL(q(z \vert x) \Vert p(z \vert x))=\log{p(x)} - \int{q(z \vert x) \log{p(x \vert z)} dz} + KL(q(z \vert x) \Vert p(z))$
   
   2. $\-int{q(z \vert x) \log{p(x \vert z)} dz}=-E_{q(z \vert x)}\log p(x \vert z)=H(x,\hat x)$且$\log{p(x)}$确定，  
   所以损失函数设计为$loss=H(x,\hat x)+KL(q(z \vert x) \Vert p(z)))$
   
   3. 假设x每一维特征都符合正态分布，计算x的均值$\mu$和方差$\sigma$，并进行随机采样，此过程不可导。  
   所以使用重参数方法，从N(0,1)上随机采样$\epsilon$，计算$z=\mu+\sigma\cdot\epsilon$，采样结果$z\sim(\mu,\sigma)$可导。
   
   4. 因为$q(z \vert x) \sim N(\mu,\sigma)$，$p(z) \sim N(0,1)$，所以$KL(q(z \vert x) \Vert p(z)))=KL(N(\mu,\sigma) \Vert N(0,1))$  
   由$KL(p1 \Vert p2)=-\frac{1}{2}\sum\limits_{i=1}^n[\log \frac{\sigma1^2}{\sigma2^2}-\frac{\sigma1^2}{\sigma2^2}-\frac{(\mu1-\mu2)^2}{\sigma2^2}+1]$  
   得$KL(q(z \vert x) \Vert p(z)))=-\frac{1}{2}\sum\limits_{i=1}^n(\log\sigma^2 - \sigma^2 - \mu^2 + 1)$
   
3. β-VAE(Disentangled VAE)：
   1. z的各维特征解耦
   
   2. $loss = H(x,\hat x) + \beta KL(q(z \vert x) \Vert p(z))),\beta>1$
   
   3. 用信息瓶颈来解释，将$\beta$值设为一个较大值，减少$q(z \vert x)$和$p(z)$的互信息，
   剔除了$q(z \vert x)$中对于重建无用的或者说是弱相关的信息，达到各维特征解耦的目的
   
4. β-TCVAE(Total Correlation Variational Auto-Encoder)：
   1. 效果优于β-VAE
   
   2. 拆解$$\begin{align}KL(q(z \vert x) \Vert p(z))) &= KL(q(z,x)\Vert q(z)p(x)),(1)\\\
   &+KL(q(z)\Vert\prod_j q(z_j)),(2)\\\
   &+\sum_j KL(q(z_j)\Vert p(z_j)),(3)\end{align}$$
   
   3. 第一项是索引编码互信息(index-code MI)，与数据变量和潜变量之间的互信息相关  
   第二项是全相关(total correlation)，潜变量空间中变量之间的相互依赖程度  
   第三项维度KL散度(dimension-wise KL)，每个潜变量的维度与先验潜变量的维度的偏离
   
   4. 损失函数设计为$loss = H(x,\hat x)+\alpha\(1)+\beta(2)+\gamma(3)$
   将$\beta$值设为一个较大值，$(2)$会向0收缩，达到各维特征解耦的目的