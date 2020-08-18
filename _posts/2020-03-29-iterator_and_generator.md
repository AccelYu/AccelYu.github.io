---
layout: post
title: Python迭代器和生成器
categories: [Python]
---

python迭代器和生成器注意点

<!-- more -->
## 迭代器

1. 可迭代对象：必须具有__iter__函数，可以返回迭代器对象

2. 迭代器对象：通过__next__函数获取聚合对象中下一个元素
   
   ```python
   class 迭代器类名:
       def __init__(self, 聚合对象):
           self.聚合对象 = 聚合对象
    
       def __next__(self):
           if 没有元素:
               raise StopIteration
           return 聚合对象元素
   ```



## 生成器

1. 定义：能够动态(循环一次计算一次返回一次)提供数据的可迭代对象

2. 作用：在循环过程中，按照某种算法推算数据，不必创建容器存储完整的结果，从而节省内存空间。数据量越大，优势越明显


### 生成器函数
```python
# 一般用法
def generator_fun():
    print('a')
    p = yield '1'
    print('b')
    if p == 'hello':
        print('p是send传过来的')
    yield '2'


if __name__ == '__main__':
    # for _ in generator_fun():
    #     print('获取', _)
    
    g = generator_fun()  # 生成器函数对象
    y1 = next(g)  # 运行第一个yield之前的内容
    print(y1)  # 打印第一个yield返回值
    # 给当前yield赋值并运行下一个yield之前的内容
    y2 = g.send('hello')
    print(y2)  # 打印第二个yield返回值
```
```python
# 遍历可迭代对象
class MyObject:
    def __init__(self, content):
        self.content = content

    def __iter__(self):
        for item in self.content:
            yield item


if __name__ == '__main__':
    myObject = MyObject([1, 2, 3])
    myIter = myObject.__iter__()
    while True:
        try:
            print(myIter.__next__())
        except StopIteration as s:
            break
```

### 内置生成器

1. enumerate：遍历可迭代对象时，可以将索引与元素组合为一个元组
   ```python
   for 索引, 元素 in enumerate(可迭代对象):
   ```

2. zip：将多个可迭代对象中对应的元素组合成元组
   ```python
   for item in zip(可迭代对象1, 可迭代对象2, ...):
   ```

### 生成器表达式
```python
变量 = (表达式 for 变量 in 可迭代对象 [if 真值表达式])
```