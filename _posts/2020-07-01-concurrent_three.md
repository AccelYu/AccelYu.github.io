---
layout: post
title: Python并发编程（三），多线程
categories: [Python]
---

线程是系统分配内核的最小单元，被称为轻量级的进程

<!-- more -->
## 线程GIL（全局解释器锁）
1. CPython解释器同一时刻只能解释执行一个线程，大大降低了线程的执行效率

2. 遇到阻塞时线程会主动让出解释器，所以执行多阻塞高延迟IO时可以提升效率

## 创建
1. threading模块创建线程

2. 自定义线程类创建线程

## 同步互斥
### 线程Event
```python
from threading import Thread, Event

s = None  # 全局变量用于通信
e = Event()  # 事件对象


def myThread():
    print("分支线程开始")
    global s
    s = 1
    e.set()  # 共享资源操作完毕


t = Thread(target=myThread)
t.start()

e.wait()  # 阻塞等待
if s == 1:
    print("分支线程先执行")
else:
    print("主线程先执行")

t.join()
```

### 线程锁Lock
```python
from threading import Thread, Lock

a = b = 0  # 共享资源
lock = Lock()


def value():
    while True:
        lock.acquire()  # 上锁
        if a != b:
            print("a = %d,b = %d" % (a, b))
        lock.release()  # 解锁


t = Thread(target=value)
t.start()
while True:
    # 上锁，代码块结束自动解锁
    with lock:
        a += 1
        b += 1
```

## 死锁
1. 两线程互相等待资源释放  
避免方法：  
   - 两线程运行同一方法  
   不让两线程同时运行
   - 两线程运行不同方法  
   传入两边锁，一个方法顺序上锁，一个方法逆序上锁

2. 本线程重复上锁  
避免方法：使用递归锁RLock()，在一个线程内可以对同一个锁重复上锁