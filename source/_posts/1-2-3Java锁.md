---
title: Java锁与优化
tags:
  - java锁
categories:
  - technology
  - java
date: 2019-12-18 06:11:43
---

### 常见锁分类

![](https://i.loli.net/2019/11/28/8eOnldMyEDgmfwU.png)

### 自旋锁

> 当一个线程在获取锁的时候，如果锁已经被其他的线程获取，那么该线程将循环等待，然后不断地判断锁是否能被成功获取，知道获取锁才会退出循环。

- 自旋锁有2种表现形式
  - CAS自旋锁

    ```java
    //类似于AtomicIneger
    ```

  - SpinLock
  
    ```java
    //利用自选实现一把锁
    public class Demo2_SpinLock {
        private AtomicReference<Thread> owner = new AtomicReference<Thread>();
        public void lock() {
            Thread current = Thread.currentThread();
            // 利用CAS
            while (!owner.compareAndSet(null, current)) {
                // DO nothing
            }
        }
    
        public void unlock() {
            Thread current = Thread.currentThread();
            owner.compareAndSet(current, null);
        }
    
    
        public static void main(String args[]){
            Demo2_SpinLock lock = new Demo2_SpinLock();
            lock.lock();
    
            lock.unlock();
        }
    }
    ```
  
   - 区别：通过CAS自旋类的仅能保证锁住一个变量，而使用spinLock的可以保证业务范围内被锁住。

### 乐观锁和悲观锁

> 乐观锁：假定没有冲突，在修改数据时如果发现数据阿之前获取的不一致，则读取最新数据，修改重试修改。
>
> 悲观锁：假定会发生并发冲突，同步所有对数据的相关操作，从读数据就开始上锁。

### 独占锁和共享锁

> 独占锁（写）：给资源加上写锁，线程可以修改资源，其他线程不能再加锁。（单写）
>
> 共享锁（读）：给资源加上锁后只能读不能改，其他线程也只能加读锁，不能加写锁；（多读）<常用于限流>

### 可重入锁和不可重入锁

> 线程拿到一把锁之后，可以自由进入同一把锁同步的其他代码。

### 公平锁和非公平锁

> 争抢锁的顺序，如果是按先来后到，则为公平反之则为非公平锁

### synchronized

> 不公平锁、可重入锁、排他锁（独占锁）、悲观锁
>
> 锁优化：
>
> 锁消除（开启锁消除参数：-XX:+DoEscapeAnalysis-XX:+EliminateLocks)
>
> 锁粗化：JDK做了锁粗化的优化，但我们可以从代码层面优化。

- 用于实例方法、静态方法时，隐式指定锁对象
- 用于代码块时，显示指定锁对象

- 锁消除实例：

  ```java
  //JIT即时编译时，进行了锁消除
  public class Demo4_LockElimination {
  
      public void test1(Object arg) {
  
          //StringBuilder线程不安全，StringBuffer用了synchronized，是线程安全的
          // jit 优化, 消除了锁
          StringBuffer stringBuffer = new StringBuffer();
          stringBuffer.append("a");
          stringBuffer.append("b");
          stringBuffer.append("c");
  
          stringBuffer.append("a");
          stringBuffer.append("b");
          stringBuffer.append("c");
  
          stringBuffer.append("a");
          stringBuffer.append("b");
          stringBuffer.append("c");
          // System.out.println(stringBuffer.toString());
      }
  
      public static void main(String[] args) throws InterruptedException {
          for (int i = 0; i < 1000000; i++) {
              new Demo4_LockElimination().test1("123");
          }
      }
  }
  ```

  - [JIT](https://www.ibm.com/developerworks/cn/java/j-lo-just-in-time/index.html)参与锁消除

### 锁升级原理

- 对象头

  > 由MarkWord、Class meta address、Array Length组成

  - MarkWord（32位）如下图：用于记录不同锁状态（实际上每个锁就占其中1行）

  ![](https://i.loli.net/2019/11/29/xWHM2YuDokQ7qv1.jpg)

- 轻量级锁：
  
- 2个线程通过CAS操作MarkWord 中无锁状态下的MarkWord（该状态下为HashCode|Age|0|01）抢锁成功则虚拟机栈栈帧中的ower会指向MarkWord，MarkWord更新为新的MarkWord（该状态下位轻量级锁状态为:指向栈中锁记录的指针|00）未抢到锁的线程会自选造成CPU资源占用，当自选次数到达一定的时候，会发生锁升级。若是此时又一个进程参与抢锁，那么不管次数到达没有，直接发生锁升级。
  
- 重量级锁：
  - 由轻量级锁升级而来
  - ObjectMonitor（对象监视器，是用C实现的所有对象基本上都有一个对象监视器）
    - owner(线程状态为running；记录抢占的线程，wait/notify中只有onwer的线程才可以调用wait，使owner设置为null，其他线程重新抢锁)
    - EntryList（线程状态为Blocking；抢锁失败的线程挂起，记录在这个队列中）
    - waitSet（线程状态为waiting；线程被wait之后被放入waitSet中，当线程调用notify之后重新抢锁，抢不到的话重新加入EntryList，）
  - 对象头若升级为重量级锁，则MarkWord更新为（Monitor address|10）
- 偏向锁
  - 锁定之后通过CAS修改MarkWord。
  - 若出现争抢，则锁升级为轻量级锁。
  - 提高在单个线程的时候的运行效率。

- 锁升级过程

  ![](https://i.loli.net/2019/12/20/ZWfrGX2BPoDshJ7.jpg)



### 总结

- 引用美团、网易云课堂的资料对锁的原理进行解析。

- 讲解了锁的一些主要分类与基本原理。

- 讲解了为了优化性能时锁的消除、锁的粗化、锁的升级。

### Java锁相关参考资料

- [美团的"锁"参考资料](https://tech.meituan.com/2018/11/15/java-lock.html)
- 网易云课堂微专业高级课程