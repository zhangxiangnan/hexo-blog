layout: title
title: java线程协作通信
date: 2019-03-19 23:15:00
tags:
- 线程
categories:
- 线程
---

java线程协作即2个线程如何交互，发信号，告诉对方一些信息，触发对方做一些事情。

### 一道题说起：
```
        public static void main(String[] args) {
            System.out.println("step1");
            Object obj = new Object();
            magic(obj);
            System.out.println("step2");
            synchronized (obj){
                System.out.println("step3");
            }
        }

```

问如何实现magic方法，使输出按如下顺序，且线程停留在为synchronized (obj)这行（注意：不输出step3）：

```
    step1
    step2
```

很明显，magic方法里需要获取obj的锁，因为需要返回到main函数，没法在magic方法里用主线程一直锁定obj，所以需要单独开一个线程来锁定obj。那么，子线程锁定obj，锁定多长时间呢，两个线程运行具有不确定性，既然不确定，就让子线程一直锁。如下：

```
     private static void magic(Object obj)  {
            Thread thread = new Thread(() -> {
                synchronized (obj){
                    while (true){
                    }
                }
            });
            thread.setName("xx");
            thread.start();
        }
```

magic启动子线程后，方法返回，main函数继续执行，此时可能子线程还没有锁定obj，导致和预期不一致。即我们要做的是子线程必须获取到obj锁，才能让magic返回，怎么让子线程告诉主线程我已经获取到了obj锁，你可以继续执行呢？

#### 主线程睡眠一定时间？？
这种不靠谱，极端情况下子线程可能在设置的休眠时间内仍然没有得到运行；另外子线程极端时间内得到了锁，但是主线程确休眠，造成cpu浪费

```
    private static void magic(Object obj)  {
            Thread thread = new Thread(() -> {
                synchronized (obj){
                    while (true){
                    }
                }
            });
            thread.setName("xx");
            thread.start();
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
```

#### 共享变量
```
    private static void magic(Object obj)  {
        final boolean[] getLock = {false};
        Thread thread = new Thread(() -> {
            synchronized (obj){
                getLock[0] = true;
                while (true){
                }
            }
        });
        thread.setName("xx");
        thread.start();

        while (!getLock[0]){//不同处理器可能主线程始终读取到的值为false
        }
    }
```

改进如下：
```
    private static void magic(Object obj)  {
        AtomicBoolean getLock = new AtomicBoolean(false);
        Thread thread = new Thread(() -> {
            synchronized (obj){
                getLock.set(true);
                while (true){
                }
            }
        });
        thread.setName("xx");
        thread.start();

        while (!getLock.get()){
        }
    }
```

#### wait&notify/notifyall 

  wait是已经获取锁的线程释放锁，然后一直阻塞等待其他线程调用相同锁的notify/notifyall方法；notify/notifyall是已经获取锁的线程释放锁，通知其他阻塞在相同锁的wait方法上的其他一个或多个线程中的一个线程继续执行。

  wait、notify、notifyall与synchronized配合使用

  可以得出如下方式：

  ```
    private static void magic(Object obj)  {
        Object obj2 = new Object();

        synchronized (obj2){
            Thread thread = new Thread(() -> {
                synchronized (obj){
                    // 锁定obj后，获取obj2的锁，获取成功即可nofity主线程继续执行
                    synchronized (obj2){
                        obj2.notify();
                    }
                    while (true){
                    }
                }
            });
            thread.setName("xx");
            thread.start();
            try {
                obj2.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
  ```

#### Condition&Lock
java的ReentrantLock结合Condition，跟notify/wait机制类似.

```
    private static void magic(Object obj)  {
        ReentrantLock reentrantLock = new ReentrantLock();
        Condition childThreadHasGetLock = reentrantLock.newCondition();

        reentrantLock.lock();

        Thread thread = new Thread(() -> {
            synchronized (obj){
                reentrantLock.lock();
                childThreadHasGetLock.signal();
                reentrantLock.unlock();

                while (true){
                }
            }
        });

        thread.setName("xx");
        thread.start();

        try {
            childThreadHasGetLock.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        reentrantLock.unlock();
    }
```

#### CountDownLatch
CountDownLatch，闭锁，基于AQS（AQS基于volatitle&CAS），阻塞的线程（一个或多个）一旦等闭锁的初始设置的计数值变为0就会恢复执行，最常用场景就是主线程等待多个外部调用都完成后（n个外部调用，闭锁的计数值为n），恢复执行。

```
    private static void magic(Object obj)  {
        CountDownLatch countDownLatch = new CountDownLatch(1);
        Thread thread = new Thread(() -> {
            synchronized (obj){
                countDownLatch.countDown();
                while (true){

                }
            }
        });
        thread.setName("xx");
        thread.start();
        try {
            countDownLatch.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```

#### CyclicBarrier
CyclicBarrier，循环屏障，可以重复使用，基于Lock&Condition（Lock，Condition基于volatitle&CAS）,常用场景是一组线程在一个屏障点集合，所有线程到达后才会恢复线程的执行，可以设置在最后一个线程到达后所有线程恢复执行前的执行操作（一个屏障点只执行一次，可以更新一些状态，共享变量等）

```
    private static void magic(Object obj)  {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(2);
        Thread thread = new Thread(() -> {
            synchronized (obj){
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException |BrokenBarrierException e) {
                    e.printStackTrace();
                }
                while (true){
                }
            }
        });
        thread.setName("xx");
        thread.start();

        try {
            cyclicBarrier.await();
        } catch (InterruptedException | BrokenBarrierException e) {
            e.printStackTrace();
        }
    }
```

#### Phaser

阶段器，基于volatitle&CAS，多个阶段，多个线程分阶段共同开始去完成每一个阶段（每一个阶段完成线程等待未完成线程）的场景，在这大材小用
```

    private static void magic(Object obj)  {
        Phaser phaser = new Phaser(2);
        Thread thread = new Thread(() -> {
            synchronized (obj){
                phaser.arriveAndAwaitAdvance();
                while (true){
                }
            }
        });
        thread.setName("xx");
        thread.start();

        phaser.arriveAndAwaitAdvance();
    }
```

#### Semaphore
信号量，基于AQS，对有限的资源的并发访问控制，相比于加锁性能更高.

```
    private static void magic(Object obj) {
        Semaphore semaphore = new Semaphore(0);
        Thread thread = new Thread(() -> {
            synchronized (obj) {
                semaphore.release();// 子线程释放一个信号量，主线程才会继续执行
                while (true) {
                }
            }
        });
        thread.setName("xx");
        thread.start();

        try {
            semaphore.acquire();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```

#### 同步队列
SynchronousQueue简单实现，其他类似Queue也行。

```
  private static void magic(Object obj) {
        SynchronousQueue<Boolean> synchronousQueue = new SynchronousQueue();
        Thread thread = new Thread(() -> {
            synchronized (obj) {
                synchronousQueue.offer(true);
                while (true) {
                }
            }
        });
        thread.setName("xx");
        thread.start();

        try {
            synchronousQueue.take();//获取不到一直阻塞
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```

#### LockSupport
lock辅助工具类

```
     private static void magic(Object obj) {
        Thread mainThread = Thread.currentThread();
        Thread thread = new Thread(() -> {
            synchronized (obj) {
                LockSupport.unpark(mainThread);
                while (true) {
                }
            }
        });
        thread.setName("xx");
        thread.start();
        LockSupport.park();
    }
```

### 总结
线程协作（单进程里的多个线程），通信的方式无非是共享变量即共享内存，锁（java提供的线程通讯机制，wait&notify，lock&condition，CountDownLatch等几个并发辅助类都是），管道（SynchronousQueue是队列，也可以理解为管道一种，PipedInputStream实现类似）
