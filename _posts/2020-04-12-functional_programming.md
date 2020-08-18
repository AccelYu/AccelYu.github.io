---
layout: post
title: Python函数式编程
categories: [Python]
---

函数式编程：数据与数据、类型与类型、函数和函数之间的关系映射

<!-- more -->
## 函数作为参数
### lambda 表达式
1. 定义：是一种匿名方法

2. 作用：  
可作为参数传入函数使用  
随时创建和销毁，减少程序耦合度

3. 语法
   ```python
   # 定义
   变量 = lambda 形参:方法体
   # 调用
   变量(实参)
   ```

4. 说明：  
形参没有可以不填  
方法体只能有一条语句，且不支持赋值语句


### 内置高阶函数
1. map（函数，可迭代对象）：根据条件处理可迭代对象中的元素，返回值为新可迭代对象。

2. filter(函数，可迭代对象)：根据条件筛选可迭代对象中的元素，返回值为新可迭代对象。

3. sorted(可迭代对象，key = 函数,reverse = bool值)：排序，返回值为排序结果。

4. max(可迭代对象，key = 函数)：根据函数获取可迭代对象的最大值。

5. min(可迭代对象，key = 函数)：根据函数获取可迭代对象的最小值。
    ```python
    def fun(operate, arg):
        for item in arg:
            if operate(item):
                yield item
    
    
    if __name__ == '__main__':
        lst = [(1, 2), (3, 1)]
        # 自定义fun()函数处理按lambda筛选
        my_generator1 = fun(lambda tup: tup[0] > 2, lst)
        for item in my_generator1:
            print(item)
        # 内置高阶函数filter按lambda筛选
        my_generator2 = filter(lambda tup: tup[0] > 2, lst)
        for item in my_generator2:
            print(item)
        # 内置高阶函数sorted按lambda排序，默认从小到大
        # 本方法是按照元组的第二位排序
        print(sorted(lst, key=lambda tup: tup[1]))
    ```

## 函数作为返回值
### 闭包
1. 三要素：  
必须有一个内嵌函数  
内嵌函数必须引用外部函数中变量  
外部函数返回值必须是内嵌函数

2. 语法：
   ```python
   # 定义
   def 外部函数名(参数):
       外部变量
       def 内部函数名(参数):
           使用外部变量
       return 内部函数名

   # 调用
   变量 = 外部函数名(参数)
   变量(参数)
   ```

3. 定义：在一个函数内部的函数,同时内部函数又引用了外部函数的变量

4. 本质：闭包是将内部函数和外部函数的执行环境绑定在一起的对象

5. 优点：内部函数可以使用外部变量

6. 缺点：外部变量一直存在于内存中，不会在调用结束后释放，占用内存

7. 作用：实现python装饰器

### 函数装饰器
1. 定义：在不改变原函数的调用以及内部代码情况下，为其添加新功能的函数

2. 语法：
   ```python
   def 函数装饰器名称(func):
       def 内嵌函数(*args, **kwargs):
           # 需要添加的新功能
           temp = 原函数(*args, **kwargs)
           # 处理temp
           return temp
       return 内嵌函数

   @ 函数装饰器名称
   def 原函数(参数):
       函数体

   原函数(参数)
   ```

3. 本质：使用“@函数装饰器名称”修饰原函数，等同于创建与原函数名称相同的变量，关联内嵌函数；故调用原函数时执行内嵌函数

4. 装饰器链：一个函数可以被多个装饰器修饰，执行顺序为从近到远
    ```python
    # 使用装饰器实现重载
    from functools import singledispatch
    
    
    @singledispatch
    # 未定义的类型默认调用此方法
    def fun(obj):
        print('other:', obj)
    
    
    @fun.register(str)
    # 识别第一个参数来决定调用哪个装饰
    def _(obj):
        print('str:', obj)
    
    
    @fun.register
    # Python3.7新增了type annotions来注明第一个参数的类型
    def _(obj: int):
        print('int:', obj)
    
    
    if __name__ == '__main__':
        fun(1.1)
        fun('abc')
        fun(5)
    ```