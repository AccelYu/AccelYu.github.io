---
layout: post
title: Python并发编程（一），多进程
categories: [Python]
---

并发指一段时间内处理多个任务，不一定要同时

<!-- more -->
## 孤儿和僵尸
1. 孤儿进程：父进程先于子进程退出，此时子进程称为孤儿进程，init进程自动回收

2. 僵尸进程：子进程先于父进程退出，父进程又没有处理子进程的退出状态，此时子进程称为僵尸进程，浪费系统的内存资源

## fork
1. pid = os.fork()：创建子进程，windows系统不支持。  
创建进程失败返回负数，成功则在原有进程中返回新进程的PID，在新进程中返回0

2. 父进程fork之前开辟的空间子进程同样拥有，父子进程对各自空间的操作相不互影响

3. 常用函数  
os.getpid()：返回当前进程的PID  
os.getppid()：返回父进程PID  
os._exit()：终止当前进程，一般用在退出子进程  
sys.exit()：通过抛出异常的方式来终止当前进程  
os.wait()：在父进程中阻塞等待处理子进程退出，影响父进程运行  
os.waitpid()：在父进程中非阻塞处理子进程退出，需要定时运行
   ```python
   import os
   import signal
   from time import *
   
   # 处理僵尸，方法一
   # 在父进程创建子进程前使用
   # 子进程结束，向父进程发送SIGCHLD信号，通知父进程回收子进程
   # 父进程不操作则子进程变成僵尸
   # 所以将SIGCHLD设置为SIG_IGN将其忽略，由init回收子进程
   signal.signal(signal.SIGCHLD, signal.SIG_IGN)
   
   pid = os.fork()
   
   if pid < 0:
       print("Create process failed")
   elif pid == 0:
       print("The new process")
       print("child pid:", os.getpid())
       # 处理僵尸，方法二
       # 创建二级子进程
       # p = os.fork()
       # if p == 0:
          # print("二级子进程")
       # 子进程退出，使二级子进程变成孤儿，由init进程回收
       # else:
          # 0为正常退出，其他数值（1-127）为不正常
          # os._exit(0)
   else:
       print("The old process")
       print("main pid:", os.getpid())
       sleep(1)
       # 处理僵尸，方法三
       # pid,status = os.wait()
       # 处理僵尸，方法四
       # 使用os.waitpid(pid, options)
       # pid：-1表示等待任意子进程退出，>0表示等待指定的子进程退出
       # options：0表示阻塞等待，WNOHANG表示非阻塞
       # pid, status = os.waitpid(-1, os.WNOHANG)
       # print("pid:", pid)
       # 打印子进程退出状态
       # print("status:", os.WEXITSTATUS(status))
   
       while True:
           pass
   ```

## multiprocessing
1. p = Process()：创建子进程，各系统支持，但无法使用标准输入  

2. 父子进程对各自空间的操作相不互影响，子进程只运行target绑定的函数部分

3. 常用函数  
p.start()：启动线程  
p.join()：阻塞等待回收进程，参数为超时时间  
p.is_alive()：查看子进程生命周期  
p.name：获取进程名称  
p.pid：获取进程PID  
p.daemon：设置父子进程的退出关系，置为True则子进程会随父进程的退出而结束，必须在start()前设置
   ```python
   from multiprocessing import Process
   from time import sleep
   import os
   import signal
   
   
   def th():
       sleep(2)
       print('th:', os.getpid())
       print("吃饭")
   
   
   if __name__ == '__main__':
       signal.signal(signal.SIGCHLD, signal.SIG_IGN)
       print('main:', os.getpid())
       p = Process(target=th)
       # 子进程随父进程的退出而结束，必须在start()之前
       # p.daemon = True
       p.start()
       # 处理僵尸，主进程等待子进程执行完毕再继续
       # p.join()
       print(p.is_alive())
       
       while True:
           pass
   ```

## 进程池
1. 必要性  
当任务量众多，每个任务在很短时间内完成时，需要频繁的创建和销毁进程

2. 原理  
创建一定数量的进程来处理事件，事件处理完进程不退出而是继续处理其他事件，直到所有事件
全都处理完毕统一销毁

3. 实现
   ```python
   from multiprocessing import Pool
   from time import sleep, ctime
   
   
   # 事件
   def worker(msg):
       sleep(2)
       print(msg)
       return ctime()
   
   
   if __name__ == '__main__':
       # 创建进程池，参数为进程数
       pool = Pool(4)
   
       # 向进程池添加事件
       for i in range(9):
           msg = "Hello %d" % i
           r = pool.apply_async(func=worker, args=(msg,))
   
       # 关闭进程池，表示不能再往线程池中添加事件
       pool.close()
   
       # 主线程执行完毕，pool会自动销毁
       # 此处用join可以很好地搭配
       pool.join()
       # 获取事件函数返回值
       print(r.get())
   ```