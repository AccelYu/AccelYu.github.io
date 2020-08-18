---
layout: post
title: Java线程间通讯
categories: [Java]
---
多个线程并发执行时，在默认情况下CPU是随机切换线程的，有时我们希望CPU按我们的规律执行线程，此时就需要线程之间协调通信。
<!-- more -->
### 休眠唤醒方式
- object wait()必须在synchronized（同步锁）下使用，必须要通过notify()、notifyAll()方法进行唤醒
- notify()用于一个生产者线程，一个消费者线程，notifyAll()用于多个生产者线程，多个消费者线程

   ```java
   public class WaitNotifyRunnable {
       private final Object obj = new Object();
       private Integer i = 0;

       public void odd() {
           while (i < 10) {
               synchronized (obj) {
                   if (i % 2 == 1) {
                       System.out.println(Thread.currentThread().getName() + ":" + i);
                       i++;
                       obj.notify();
                   } else {
                       try {
                           obj.wait();
                       } catch (InterruptedException e) {
                           e.printStackTrace();
                       }
                   }
               }
           }
       }

       public void even() {
           while (i < 10) {
               synchronized (obj) {
                   if (i % 2 == 0) {
                       System.out.println(Thread.currentThread().getName() + ":" + i);
                       i++;
                       obj.notify();
                   } else {
                       try {
                           obj.wait();
                       } catch (InterruptedException e) {
                           e.printStackTrace();
                       }
                   }
               }
           }
       }

       public static void main(String[] args) {
           final WaitNotifyRunnable runnable = new WaitNotifyRunnable();
           Thread t1 = new Thread(runnable::odd, "奇数线程");
           Thread t2 = new Thread(runnable::even, "偶数线程");

           t1.start();
           t2.start();
       }
   }
   ```

- condition await() 必须和Lock（互斥锁）配合使用，必须通过 signal()、signalAll() 方法进行唤醒
- signal()用于一个生产者线程，一个消费者线程，signalAll()用于多个生产者线程，多个消费者线程

   ```java
   public class AWaitSignalRunnable {
       private Lock lock = new ReentrantLock();
       private Condition condition = lock.newCondition();
       private Integer i = 0;

       public void odd() {
           while (i < 10) {
               lock.lock();
               try {
                   if (i % 2 == 1) {
                       System.out.println(Thread.currentThread().getName() + ":" + i);
                       i++;
                       condition.signal();
                   } else {
                       condition.await();
                   }
               } catch (InterruptedException e) {
                   e.printStackTrace();
               } finally {
                   lock.unlock();
               }

           }
       }
   }
   ```

### CountDownLatch
- 使一个线程等待其他线程完成各自的工作后再执行

   ```java
   public class CountDownLatchDemo {

       static class TaskThread extends Thread {
           CountDownLatch latch;
           int time;

           public TaskThread(CountDownLatch latch, int time) {
               this.latch = latch;
               this.time = time;
           }

           @Override
           public void run() {
               try {
                   System.out.println(getName() + " start");
                   //模拟不同线程的执行时间
                   Thread.sleep(1000 * time);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               } finally {
                   System.out.println(getName() + " Done");
                   latch.countDown();
               }
           }
       }
    
       public static void main(String[] args)  {
           int threadNum = 5;
           CountDownLatch latch = new CountDownLatch(threadNum);

           for (int i = 1; i < threadNum + 1; i++) {
               TaskThread task = new TaskThread(latch, i);
               task.start();
           }
    
           latch.await();
           System.out.println("All Task is Done!");
       }
   }
   ```

### CyclicBarrier
- 所有线程会等待全部线程到达栅栏后才会继续执行，并且最后到达的线程会完成 Runnable 的任务
- CyclicBarrier 是可循环利用的

   ```java
   public class CyclicBarrierDemo {

       static class TaskThread extends Thread {
           CyclicBarrier barrier;

           public TaskThread(CyclicBarrier barrier) {
               this.barrier = barrier;
           }

           @Override
           public void run() {
               try {
                   Thread.sleep(1000);
                   System.out.println(getName() + " 到达栅栏 A");
                   barrier.await();
                   System.out.println(getName() + " 冲破栅栏 A");

                   Thread.sleep(2000);
                   System.out.println(getName() + " 到达栅栏 B");
                   barrier.await();
                   System.out.println(getName() + " 冲破栅栏 B");
               } catch (Exception e) {
                   e.printStackTrace();
               }
           }
       }

       public static void main(String[] args) {
           int threadNum = 5;
           CyclicBarrier barrier = new CyclicBarrier(threadNum, () ->
                   System.out.println(Thread.currentThread().getName() + " 完成最后任务"));

           for (int i = 0; i < threadNum; i++) {
               new TaskThread(barrier).start();
           }
       }
   }
   ```

### Semaphore
- 控制线程的并发数量

   ```java
   public class SemaphoreDemo {

       static class myTask implements Runnable {
           private final int num;
           private final Semaphore semaphore;
           private static final SimpleDateFormat sf = new SimpleDateFormat("HH:mm:ss");

           public myTask(int num, Semaphore semaphore) {
               this.num = num;
               this.semaphore = semaphore;
           }

           public void run() {
               try {
                   semaphore.acquire();
                   //acquireUninterruptibly无需抛出异常，但不可中断，应避免使用
                   //semaphore.acquireUninterruptibly();
                   System.out.println("任务" + this.num + " 请求资源 " + sf.format(new Date()));
                   Thread.sleep(2000);
               } catch (InterruptedException e) {
                   e.printStackTrace();
               } finally {
                   semaphore.release();
                   System.out.println("任务" + this.num + " 释放资源 " + sf.format(new Date()));
               }
           }
       }

       public static void main(String[] args) {
           int task = 7;
           //每次只允许3个线程并发处理
           Semaphore semaphore = new Semaphore(3);
           for (int i = 0; i < task; i++) {
               new Thread(new myTask(i, semaphore)).start();
           }
       }
   }
   ```