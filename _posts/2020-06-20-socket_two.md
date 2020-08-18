---
layout: post
title: Python Socket（二），IO多路复用
categories: [Python]
---

IO多路复用能同时监听多个种类的多个IO事件，当其就绪就执行它

<!-- more -->
## 注意点
1. 优点：它是单线程的，不需要建立进程（线程）去处理多个请求，节省系统资源

2. 缺点：事件探测和事件响应夹杂在一起，一旦事件响应的执行体过于庞大，整个程序就会出现卡住的情况。
因为需要通过循环去遍历事件，再去执行操作。

## 方案
1. select方法：windows、linux、unix

2. poll方法：linux、unix

3. epoll方法：linux

## select
实现
```python
"""
IO多路复用select实现多客户端通信
重点代码
"""

from socket import *
from select import select

# 　设置套接字为关注IO
s = socket()
s.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)
s.bind(('0.0.0.0', 8888))
s.listen(5)

# 设置关注的IO
rlist = [s]
wlist = []
xlist = []

while True:
    rs, ws, xs = select(rlist, wlist, xlist)
    # 遍历三个返回值列表，判断哪个IO发生
    for r in rs:
        # 如果是套接字就绪则处理连接
        if r is s:
            c, addr = r.accept()
            print("Connect from", addr)
            rlist.append(c)  # 加入新的关注IO
        else:
            data = r.recv(1024)
            if not data:
                rlist.remove(r)
                r.close()
                continue
            print(data.decode())
            wlist.append(r)

    for w in ws:
        w.send(b'Ok,Thanks')
        wlist.remove(w)
    for x in xs:
        pass
```

## poll
实现
```python
from socket import *
from select import *

s = socket()
s.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)
s.bind(('0.0.0.0', 8888))
s.listen(5)

p = poll()

# 建立查找字典
fdmap = {s.fileno(): s}

# 设置关注IO及事件类型
# POLLIN读取、POLLOUT输出、POLLERR错误发生
p.register(s, POLLIN | POLLERR)

while True:
    events = p.poll()  # 阻塞等待IO发生
    print(events)
    # events返回(文件编号,事件类型)
    for fno, event in events:
        # 如果是对象s，则说明有新的连接
        if fno == s.fileno():
            c, addr = fdmap[fno].accept()
            print(addr, '连接')
            # 添加新的关注事件
            p.register(c, POLLIN)
            fdmap[c.fileno()] = c
        elif event & POLLIN:  # 位运算，确认是否为POLLIN
            data = fdmap[fno].recv(1024)
            # 断开发生时data得到空此时POLLIN也会就绪
            if not data:
                print(fdmap[fno].getpeername(), '断开连接')
                p.unregister(fno)  # 取消关注
                fdmap[fno].close()
                del fdmap[fno]
                continue
            print(data.decode())
            fdmap[fno].send(b'OK')
```


## epoll
- select和poll从内核拷贝回所有对象，再去检查就绪状态
- epoll只返回就绪对象，而且可监控的对象数目更多

实现：与poll流程一致  
特有事件类型：EPOLLET，边缘触发。第一次读写事件触发，通知一次，
如果没有完成，不会像select和poll一样一直通知，直到第二次读写事件触发
