---
layout: post
title: Python Socket（一），IO阻塞
categories: [Python]
---

Socket其实是一个门面模式，连接应用层和传输层

<!-- more -->
## 分类
1. 流式套接字(SOCK_STREAM)：以字节流方式传输数据，实现tcp网络传输方案

2. 数据报套接字(SOCK_DGRAM)：以数据报形式传输数据，实现udp网络传输方案

## TCP粘包
1. 现象：后一包数据的头紧接着前一包数据的尾

2. 原因：  
   - 发送端需要等缓冲区满才发送出去，造成粘包  
   - 接收方不及时接收缓冲区的包，造成多个包接收

3. 对应方法：发送时添加消息边界或告知接收方数据的大小是多少字节

## 阻塞IO
以下均以TCP为例

### 单进程（线程）
- HTTP服务
- 响应格式：响应行，响应头，空行，响应体
- 缺点：消息过大或网络延迟会一直阻塞，其它请求会一直等待

```python
import socket


# 　处理浏览器的http请求
def handle(connfd):
    print("Request from", connfd.getpeername())
    request = connfd.recv(4096)  # 接收http请求
    # 防止request为空造成下方数组越界报错
    if not request:
        return
    # 将请求按行分割
    request_line = request.splitlines()[0].decode()
    # 获取请求内容，浏览器默认请求/
    info = request_line.split(' ')[1]

    if info == '/':
        # 打开当前目录下的index.html
        f = open('index.html', encoding='UTF-8')
        response = "HTTP/1.1 200 OK\r\n"
        response += "Content-Type: text/html\r\n"
        response += '\r\n'
        response += f.read()
    else:
        response = "HTTP/1.1 404 Not Found\r\n"
        response += "Content-Type: text/html\r\n"
        response += '\r\n'
        response += "<h1>Sorry....</h1>"
    # 向浏览器发送内容
    connfd.send(response.encode())


# 　搭建TCP网络
def TCP():
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind(('0.0.0.0', 8000))
    server.listen(3)
    print("Listen the port 8000...")
    while True:
        conn, addr = server.accept()
        handle(conn)
        conn.close()


if __name__ == "__main__":
    TCP()
```

### 多进程（线程）
- socket服务
- 缺点：并发量较大的情况，频繁创建和销毁进程（线程），开销较大

```python
# 服务端
import os
import signal
import struct
import sys
from socket import *
from threading import Thread


def handle(c):
    print("连接的客户端", c.getpeername())
    while True:
        data = c.recv(1024)
        if not data:
            break
        print(data.decode())
        fmt = struct.Struct('32s')
        c.send(fmt.pack('客户端接收成功'.encode('utf-8')))
    c.close()


def tcp_fork(sock, c):
    signal.signal(signal.SIGCHLD, signal.SIG_IGN)
    pid = os.fork()
    if pid == 0:
        sock.close()  # 子进程不需要sock
        handle(c)  # 具体处理客户端请求
        os._exit(0)  # handle()结束意味着客户端断开连接，此时结束子进程
    else:
        c.close()  # 父进程不需要c


def tcp_thread(c):
    t = Thread(target=handle, args=(c,))
    t.setDaemon(True)  # 分支线程随主线程退出
    t.start()


if __name__ == '__main__':
    s = socket()
    # 设置端口立即重用
    s.setsockopt(SOL_SOCKET, SO_REUSEADDR, 1)
    s.bind(("0.0.0.0", 8888))
    s.listen(3)

    print('正在监听', s.getsockname())

    while True:
        try:
            # 阻塞等待其它客户端连接
            conn, addr = s.accept()
        except KeyboardInterrupt:
            sys.exit("服务器退出")
        except Exception as e:
            print(e)
            continue

        tcp_fork(s, conn)
        # tcp_thread(conn)
```
```python
# 客户端
import socket
import struct
import sys


def socket_client():
    try:
        client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        client.connect(('127.0.0.1', 8888))
    except socket.error as msg:
        print(msg)
        sys.exit(1)
    while 1:
        data = input('please input:')
        if client:
            client.send(data.encode())
            fmt = struct.Struct('32s')
            receive = fmt.unpack(client.recv(1024))
            print(receive[0].strip(b'\x00').decode('utf-8'))
        if data == 'exit':
            break
    client.close()


if __name__ == '__main__':
    socket_client()
```
