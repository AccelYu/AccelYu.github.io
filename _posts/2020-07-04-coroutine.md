---
layout: post
title: Python协程
categories: [Python]
---

在应用层通过函数的暂停跳转实现多任务同步操作，消耗较少资源，但不能运用多核优势

<!-- more -->
## asyncio
```python
import asyncio
import time


async def do_work(arg, s):
    print(arg, 'start')
    await asyncio.sleep(s)
    print(arg, 'end')
    return arg


async def main():
    res = await asyncio.gather(do_work('cor1', 2), do_work('cor2', 4))
    return res

if __name__ == "__main__":
    start = time.time()
    result = asyncio.run(main())
    print("Time:", time.time() - start)
    print(result)
```

## greenlet
```python
from greenlet import greenlet


def fun1(param1, param2):
    print("start fun1")
    z = gr2.switch(param1 + param2)  # 切换到协程二执行并传参
    print(z)
    print("end fun1")  # 执行结束，返回主线程


def fun2(param3):
    print("start fun2")
    print(param3)
    gr1.switch(1)  # 切换到协程实例一中执行,传递参数给上一个运行节点
    print("end fun2")  # 不会执行该行代码


if __name__ == "__main__":
    gr1 = greenlet(fun1)  # 协程实例一
    gr2 = greenlet(fun2)  # 协程实例二
    gr1.switch("hello", ' world')  # 切换到协程一执行，传入两个参数
    print("end main")
```

## gevent
```python
import gevent


def fun1(arg):
    print("start fun1", arg)
    gevent.sleep(1)
    print("end fun1")


def fun2():
    print("start fun2")
    gevent.sleep(2)
    print("end fun2")


if __name__ == "__main__":
    # 将函数封装为协程，遇到gevent阻塞自动执行
    g1 = gevent.spawn(fun1, 'hello')
    g2 = gevent.spawn(fun2)

    gevent.joinall([g1, g2])  # 等待g1，g2结束
```
```python
import gevent
from gevent import monkey

monkey.patch_all()  # 该句执行在导入socket前
from socket import *


# 　处理客户端请求
def handle(c):
    while True:
        data = c.recv(1024)
        if not data:
            break
        print(data.decode())
        c.send(b'OK')
    c.close()


if __name__ == '__main__':
    s = socket()
    s.bind(('0.0.0.0', 8888))
    s.listen(5)
    while True:
        conn, addr = s.accept()
        print("Connect from", addr)
        # handle(c)   # 循环方案
        gevent.spawn(handle, conn)  # 协程方案
```