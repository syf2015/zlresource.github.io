---
layout: post
title:  "Java同步辅助类源码分析-Semaphore"
date:  2016-06-04
categories: JAVA
---

Java同步辅助类源码分析-Semaphore

---

- 目录
  {:toc}

#### Jdk1.5 java.util.concurrent包下引入了同步辅助类有：

1.Semaphore信号量：通常用来限制线程可以同时访问的（物理或逻辑）资源数量。
2.CountDownLatch(n)：在一组线程为完成时，让其一直阻塞等待，直到全部到达(计数器为0)
3.CyclicBarrier:可重置的多路同步点,在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。
                因为该 barrier在释放等待线程后可以重用，所以称它为循环的barrier。
4.Phaser 可重用的同步屏障：可以动态修改同步线程的个数，功能更强大.
         非常适用于在多线程环境下同步协调分阶段计算任务（Fork/Join框架中的子任务之间需同步时，优先使用Phaser）
5.Exchanger：允许两个线程在某个汇合点交换对象，在某些管道设计时比较有用。
             Exchanger提供了一个同步点，在这个同步点，一对线程可以交换数据。

#### 一、Semaphore信号量: 

acquire获取（信号量减1）
release释放（信号量加1）

#### 源码原理分析：

Semaphore内部主要通过AQS（AbstractQueuedSynchronizer）实现线程的管理。
Semaphore构造函数中，参数permits表示许可数，它最后传递给了AQS的state值。
线程在运行时首先获取许可，如果成功，许可数就减1，线程运行，当线程运行结束就释放许可，许可数就加1。
如果许可数为0，则获取失败，线程位于AQS的等待队列中，它会被其它释放许可的线程唤醒。

#### 源码分析：

Semaphore两个构造函数：
非公平：是指在获取许可时先尝试获取许可，而不关心等待队列中是否有线程需要获取许，如果获取失败，才会入列。
而公平的信号量：在获取许可时首先要查看等待队列中是否已有线程，如果有则入列。
​      
1.非公平，传入n个，允许进入资源的线程

```java
          public Semaphore(int permits) {
            sync = new NonfairSync(permits);
          }
```
2.公平

```java
          public Semaphore(int permits, boolean fair) {
            sync = fair ? new FairSync(permits) : new NonfairSync(permits);
          }
          
      public void acquire() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
      }
```
​        

