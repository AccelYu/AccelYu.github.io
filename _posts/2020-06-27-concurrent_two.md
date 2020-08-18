---
layout: post
title: Python并发编程（二），进程间通信（IPC）
categories: [Python]
---

进程间空间独立，资源不共享，此时在需要进程间数据传输时就需要特定的手段进行数
据通信

<!-- more -->
## 管道
1. 在内存中开辟管道空间，生成管道操作对象，多个进程使用同一个管道对象进行读写即可实现通信

2. 实现

    ```python
    import time
    from multiprocessing import Process, Pipe
    
    
    def fun1():
        time.sleep(2)
        # 向管道写入内容
        fd2.send("msg")
    
    
    def fun2():
        # 阻塞形式接收
        print(fd1.recv())
    
    
    if __name__ == '__main__':
        # 创建管道
        # Pipe()参数默认为True，双向管道均可读写
        # False表示是单向管道，fd1只读，fd2只写
        fd1, fd2 = Pipe()
    
        p1 = Process(target=fun1)
        p1.start()
        p1.join()
    
        p2 = Process(target=fun2)
        p2.start()
        p2.join()
    ```


## 消息队列
1. 在内存中建立队列模型，进程通过队列将消息存入或取出

2. 实现
    ```python
    # 使用Queue
    from multiprocessing import Queue, Process
    import time
    
    
    def producer(name, queue):
        for value in ['A', 'B', 'C']:
            time.sleep(1)
            product = name + '的' + value
            queue.put(product)
            print('%s已被添加至队列中' % product)
    
    
    def consumer(queue):
        while True:
            product = queue.get()
            # 检测到结束标志后跳出死循环
            # 这种方法在某些情况下不是很可靠
            if not product:
                break
            time.sleep(1.2)
            print(product + '已被取出')
    
    
    if __name__ == '__main__':
        que = Queue(2)
    
        p1 = Process(target=producer, args=('p1', que))
        p2 = Process(target=producer, args=('p2', que,))
        c1 = Process(target=consumer, args=(que,))
        p1.start()
        p2.start()
        c1.start()
        p1.join()
        p2.join()
    
        # 放入消费者结束标志
        que.put(None)
    ```
    
    ```python
    # 使用JoinableQueue
    # 无需判断queue是否为空，十分可靠
    from multiprocessing import JoinableQueue, Process
    import time
    
    
    def producer(name, queue):
        for value in ['A', 'B', 'C']:
            time.sleep(1)
            product = name + '的' + value
            queue.put(product)
            print('%s已被添加至队列中' % product)
        # 阻塞到queue中所有任务完成
        queue.join()
    
    
    def consumer(queue):
        while True:
            product = queue.get()
            time.sleep(1.2)
            print(product + '已被取出')
            # 标识当前任务完成，通知给消费者
            queue.task_done()
    
    
    if __name__ == '__main__':
        que = JoinableQueue(2)
    
        p1 = Process(target=producer, args=('p1', que))
        p2 = Process(target=producer, args=('p2', que,))
        c1 = Process(target=consumer, args=(que,))
        # 设置守护进程，主线程完成时消费者进程终止
        c1.daemon = True
        p1.start()
        p2.start()
        c1.start()
        # 主线程阻塞到生产者进程完成
        p1.join()
        p2.join()
    ```


## 共享内存
1. 在内中开辟一块空间，进程可以写入内容和读取内容完成通信，但是每次写入会覆盖之前内容

2. 实现
    ```python
    from multiprocessing import Value,Array
    
    # 开辟单个共享内存空间
    # ctype为C结构缩写
    # 共享对象的值可通过obj.value进行修改
    obj = Value(ctype,data)
    
    # 开辟数组共享内存空间
    # data为列表，列表内各元素数据类型一致
    # 共享对象的值只能通过索引修改，长度不能变
    obj = Array(ctype,data)
    ```


## 信号量
1. 内置的计数器，用来控制同时运行的进程数量

2. 实现
    ```python
    from multiprocessing import Semaphore
    
    # num代表并发数量
    sem = Semaphore(num)
    
    # 将信号量减1 当信号量为0时阻塞
    sem.acquire()
    # 将信号量加1
    sem.release()
    # 获取信号量数量
    sem.get_value()
    ```


## 本地套接字
1. 本地两个程序之间通过套接字进行数据的收发

2. 实现
    ```python
    # server
    from socket import *
    from time import ctime
    import os
    
    file = './msg.sock'
    # 做个判断，如果文件存在则删除
    if os.path.exists(file):
        os.remove(file)
    
    socketfd = socket(AF_UNIX, SOCK_STREAM)
    socketfd.bind(file)
    socketfd.listen(5)
    
    while True:
        connfd, addr = socketfd.accept()
        while True:
            data = connfd.recv(1024)
            if not data:
                break
            print(data.decode())
            connfd.send('copy! {}'.format(ctime()).encode())
    
        connfd.close()
    ```
    
    ```python
    # client
    from socket import *
    
    file = './msg.sock'
    clientfd = socket(AF_UNIX, SOCK_STREAM)
    clientfd.connect(file)
    
    while True:
        msg = input('>>>')
        if not msg:
            break
        clientfd.send(msg.encode())
    
        data = clientfd.recv(1024)
        print(data.decode())
    
    clientfd.close()
    ```