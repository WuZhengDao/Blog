---
title: locks接口及实现
date: 2019-12-09 20:03:30
tags:
- locks
categories:
- technology
- java
---

- Locks包类层次结构

  - 实现类：

    - ReentrantLock
    - ReentrantReadWriteLock

  - 方法：

    - lock():void
    - lockInterruptibly():void
    - tryLock()：boolean
    - tryLock(long time, TimeUnit unit):  boolean
    - unlock(): void

    ```java
    import java.util.concurrent.locks.Lock;
    import java.util.concurrent.locks.ReentrantLock;
    
    public class Demo1_GetLock {
        //公平锁
        //static Lock lock = new ReentrantLock(true);
    
        //非公平锁
        static Lock lock = new ReentrantLock();
    
        public static void main(String args[]) throws InterruptedException {
            //主线程 拿到锁
            lock.lock();
    
            Thread th = new Thread(new Runnable() {
                @Override
                public void run() {
     /*               System.out.println("begain to get lock...");
                    lock.lock();
                    System.out.println("interrupted...");*/
    
                    //子线程 获取锁(浅尝辄止)
    /*                boolean result = lock.tryLock();
                    System.out.println("是否获得到锁：" +result);*/
    
                    //子线程 获取锁(点到为止)
    /*                try {
                        boolean result1 = lock.tryLock(5, TimeUnit.SECONDS);
                        System.out.println("是否获得到锁：" +result1);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }*/
    
                    //子线程 获取锁（任人摆布）
               /*     try {
                        System.out.println("start to get lock Interruptibly");
                        lock.lockInterruptibly();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                        System.out.println("dad asked me to stop...");
                    }*/
    
                    //子线程 获取锁（不死不休）
                    System.out.println("begain to get lock...");
                    lock.lock();
                    System.out.println("succeed to get lock...");
    
                }
            });
            th.start();
            Thread.sleep(10000L);
            lock.unlock();
        }
    }
    ```

    - new Condition(): Condition

    ```java
    public class Demo3_Condition_deadLock {
        private static Lock lock = new ReentrantLock();
        private static Condition condition = lock.newCondition();
    
        public static void main(String[] args) throws InterruptedException {
    
            Thread th = new Thread(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(2000L);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    lock.lock();
                    try {
                        System.out.println("获得锁，调用condition.await()\n");
                        condition.await();      // waiting  park
                        System.out.println("唤醒了...\n");
    
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    } finally {
                        lock.unlock();
                    }
                }
            });
    
            th.start();
    
            lock.lock();
            condition.signal();
            lock.unlock();
        }
    }
    //condition的await()和signal()方法必须在lock()和unlock之间，类似于sync锁，不然会报无monitor的错。
    //condition和sync的区别是，condition可以实现多个等待池
    ```

    - condition实现阻塞队列

    ```java
    //condition实现阻塞队列
    
    
    ```

    

  