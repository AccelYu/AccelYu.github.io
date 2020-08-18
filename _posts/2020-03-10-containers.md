---
layout: post
title: Python容器
categories: [Python]
---

python容器包括字符串str、列表list、元组tuple、字典dict、集合set和frozenset

<!-- more -->
## 通用操作
1. 成员运算  
数据 in(not in) 容器

2. 索引  
正向索引从0开始，反向索引从-1开始

3. 切片  
语法：容器[开始索引:结束索引:步长]，从0开始，左闭右开  
反向切片：[::负整数]，-1开始

## 字符串str
1. 定义：不可变序列容器

2. 常见函数  
    ord(字符串)：返回该字符串的Unicode码  
    chr(Unicode码)：返回对应的字符串

3. 注意点  
    字符串前加r：去除转义字符  
    字符串前加f：相当于 format()函数  
    字符串前加b：转bytes类型  
    字符串前加u：以Unicode格式进行编码

## 列表list
1. 定义：可变序列容器

2. 常见函数  
将多个字符串拼接为一个：str01 = '连接符'.join(list01)  
将一个字符串拆分为多个：list01 = str01.split('分隔符')  
在列表末尾添加新的对象：list01.append()  
在列表末尾添加新的列表：list01.extend(list02)

3. 拷贝  
直接赋值：复制了所有数据的内存地址引用（即所有数据都共用）  
浅拷贝：只有第一层才是拷贝者自己的，所有嵌套层复制了内存地址引用  
list02 = list01[:]或list02 = list01.copy()  
深拷贝：拷贝者将所有数据拷贝下来，操作独立，互不干扰  
list02 = deepcopy(list01)

## 元组tuple
1. 定义：不可变序列容器

2. 按子元素的某个元素从小到大排序：
   ```python
   data = ()
   # 不改变data
   sorted(data, key=itemgetter(第几个))
   # 改变data
   data.sort(key=itemgetter(第几个))
   ```

## 字典dict
1. 定义：可变映射容器，键不重复

2. 常见函数：  
update()：添加一条或多条记录
item()：获取键值对元组组成的列表
collections.defaultdict：为所有值赋初值

3. key保留，value相加（仅限value为数字类型）：
   ```python
   x = {}
   y = {}
   X, Y = Counter(x), Counter(y)
   z = dict(X + Y)
   ```


## 集合set和frozenset
1. 定义：set为可变映射容器，frozenset为不可变映射容器，无序不重复

2. 常见函数：  
添加元素：~~set01.update()~~或set01.add()  
移除元素：~~set01.remove()~~或set01.discard()

3. 用法  
A ∩ B：A & B  
A ∪ B：A | B  
A \ B：A - B  
A Δ B：A ^ B
