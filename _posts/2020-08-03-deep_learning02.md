---
layout: post
title: Python深度学习（二），TensorFlow
categories: [Python]
mathjax: true
---

TensorFlow是一个端到端开源机器学习平台

<!-- more -->
## 常用API
```python
import numpy as np
import tensorflow as tf

a1 = np.arange(6).reshape(2, 3)
print(a1.dtype)
# 1.numpy类型转换为tensor类型
a2 = tf.convert_to_tensor(a1)
print(a2.dtype)
# 2.取回numpy类型数据
a3 = a2.numpy()
# 3.cast转换数据类型
b = tf.cast(a2, dtype=tf.float32)
# 4.包装tensor，使其具有trainable属性，可以求导及优化
c = tf.Variable(a2)
# 5.以正态分布初始化矩阵，可指定mean、stddev
d = tf.random.normal([2, 3], mean=0, stddev=1)
print(d)
# 6.以截断正态分布初始化矩阵
e = tf.random.truncated_normal([2, 3], mean=0, stddev=1)
print(e)
# 7.以均匀分布初始化矩阵
e = tf.random.uniform([2, 3], minval=0, maxval=1)
print(e)
# 8.随机打散顺序
f = tf.random.shuffle(tf.range(6))
print(f)
# 9.独热编码numpy.ndarray
g = tf.keras.utils.to_categorical([1, 3])
print(g.__class__)
# 10.独热编码tensor
h = tf.one_hot([1, 3], 10)
print(h)
# 11.根据索引提取数据
i1 = tf.gather(e, axis=0, indices=[1])
print(i1)
i2 = tf.gather_nd(e, [[0, 1], [1, 2]])
print(i2)
# 12.维度转换
j = tf.transpose(a2, [1, 0])
print(j)
# 12.矩阵合并
k = tf.concat([a2, a2], axis=0)
print(k)
# 13.矩阵扩充（增加维度）
l = tf.stack([a2, a2])
print(l)
# 14.矩阵分割
m = tf.split(a2, axis=0, num_or_size_splits=2)
print(m)
# 15.矩阵转置
n = tf.transpose(a2, [1, 0])
print(n)
# 16.ord为范数
o = tf.norm(tf.cast(a2, tf.float32), ord=2, axis=1)
print(o)
# 17.获取最大k个值的索引、值
p = tf.math.top_k(a2, 2)
print(p.indices, p.values)
# 18.限幅
q1 = tf.clip_by_value(a2, 2, 4)  # 根据值限幅
q2 = tf.clip_by_norm(tf.cast(a2, tf.float32), 1)  # 根据范数限幅
# q3 = tf.clip_by_global_norm(grads)  # 等比范数限幅
print(q1, q2)
```

## 数据加载
```python
import os
import tensorflow as tf
from tensorflow.keras import layers, optimizers, Sequential


# 预处理方法
def preprocess(x, y):
    x_pre = tf.cast(x, dtype=tf.float32) / 255
    x_pre = tf.reshape(x_pre, [-1])
    y_pre = tf.cast(y, dtype=tf.int32)
    # 独热编码
    # y_pre = tf.one_hot(y_pre, depth=10)
    return x_pre, y_pre


# 数据集设置
def MyDataset(ds, shuffle_buffer_size):
    # 大内存操作之前执行小内存操作
    # 预取buffer_size个批次的数据
    ds = ds.prefetch(buffer_size=2)
    # shuffle打乱顺序
    ds = ds.shuffle(shuffle_buffer_size)
    # 并行预处理，batch_size设置每批大小
    ds = ds.apply(tf.data.experimental.map_and_batch(
        map_func=preprocess, batch_size=100, num_parallel_calls=tf.data.experimental.AUTOTUNE))
    # 预处理之后进行缓存，内存不够可以缓存至文件
    ds = ds.cache()
    return ds


if __name__ == '__main__':
    # 关闭显存贪婪占用
    os.environ["TF_FORCE_GPU_ALLOW_GROWTH"] = "true"

    # 加载mnist
    mnist = tf.keras.datasets.fashion_mnist
    # 加载训练数据和测试数据
    (x_train, y_train), (x_valid, y_valid) = mnist.load_data()

    # from_tensor_slices将特征与标签组合
    train_ds = tf.data.Dataset.from_tensor_slices((x_train, y_train))
    del x_train, y_train  # 删除使用过的数据

    validate_ds = tf.data.Dataset.from_tensor_slices((x_valid, y_valid))
    del x_valid, y_valid

    train_ds = MyDataset(train_ds, 60000)
    validate_ds = MyDataset(validate_ds, 10000)
```