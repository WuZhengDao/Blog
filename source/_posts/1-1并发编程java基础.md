---
title: 并发编程java基础
date: 2019-11-27 17:57:16
tags:
- 高并发编程
categories:
- technology
- java
---

## java程序运行堆栈分析（15min）

- .class文件

  - .java文件通过编译器编译成.class文件。
  - .class文件为以0xcafebabe开头的二进制文件流

- java运行时数据区（.class文件运行的区域）

- ![](https://i.loli.net/2019/11/27/vrbq8XmgoYAPc14.jpg)

  - 线程独占&线程共享

    - 线程独占：每个线程都会有它独占的空间，随线程生命周期而创建和销毁（包括虚拟机栈，本地方法栈，程序计数器）
    - 线程共享：所有线程能访问这块内存数据，随虚拟机或者GC而创建销毁(包括Java堆，方法区)

  - 程序计数器：

    - **线程私有**，每条线程都有独立的程序计数器，各条线程之间互不影响
    - 较小的内存空间，保存当前线程所执行的字节码开始地址（字节码解释器通过改变计数器的值来选取下一条所需执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。）
    - 若线程执行的是**JAVA方法**==>>保存正在执行**虚拟机字节码指令地址**
    - 若线程执行的是**Native方法**==>>**计数器为空**
    - 唯一一个在JAVA虚拟机中**没有**规定**OutOfMemoryError**情况的内存区域

  - 虚拟机栈

    - **线程私有**

    - 描述**java方法执行的内存模型**：每个方法在执行的同时都会创建一个**栈帧（Stack Frame）**，栈帧中存储着**局部变量表**、**操作数栈**、**动态链接**、**方法出口**等信息。**每一个方法从调用直至执行完成的过程，会对应一个栈帧在虚拟机栈中入栈到出栈的过程。**

    - 被调用方法**先入后出**。

    - **局部变量表**中存放了编译期可知的各种：

      - **基本数据类型**(boolen、byte、char、short、int、 float、 long、double）
      - **对象引用**（reference类型，它不等于对象本身，可能是一个指向对象起始地址的指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）
      - **returnAddress类型**（指向了一条字节码指令的地址）

      其中64位长度的long和double类型的数据会占用2个局部变量空间（Slot），其余数据类型只占用1个。**局部变量表所需的内存空间在编译期间完成分配**，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。

    - 2种异常：

      - **StackOverflowError**：线程请求的栈深度大于虚拟机所允许的深度，将会抛出此异常。
      - **OutOfMemoryError**：当可动态扩展的虚拟机栈在扩展时无法申请到足够的内存，就会抛出该异常。

  - 本地方法栈：

    - **线程私有**
    - 服务于虚拟机使用到的Native方法
    - 在Sun HotSpot虚拟机中直接把本地方法栈和虚拟机栈合二为一
    - 2种异常
      - **StackOverflowError**
      - **OutOfMemoryError**

  - Java堆

    - **线程共享**

  - 对于大多数应用而言，**Java堆（Heap）**是Java虚拟机所管理的内存中最大的一块

    - 此内存区域**唯一的目的**是**存放对象实例**

  - 在Heap 中分配一定的内存来保存对象实例，实际上只是保存**对象实例的属性值**，**属性的类型**和**对象本身的类型标记**等，**并不保存对象的方法（方法是指令，保存在Stack中）**,在Heap 中分配一定的内存保存对象实例和对象的序列化比较类似。对象实例在Heap 中分配好以后，需要**在Stack中保存一个4字节的Heap 内存地址**，用来定位该对象实例在Heap 中的位置，便于找到该对象实例。

  - Java堆是垃圾收集器管理的主要区域，因此也被称为**“GC堆（Garbage Collected Heap）”**。

  - 方法区

    - 线程共享
    - **Object Class Data(类定义数据)**是存储在方法区的
      - 此外，**常量**、**静态变量**、**JIT编译后的代码**也存储在方法区。

  - 从内存回收的角度内存空间可以划分为：（java 1.8之后的元空间就不多加讲解了）

    - **新生代（Young）**： 新生成的对象优先存放在新生代中，新生代对象朝生夕死，存活率很低。在新生代中，常规应用进行一次垃圾收集一般可以回收70% ~ 95% 的空间，回收效率很高。新生代又可细分为**Eden空间**、**From Survivor空间**、**To Survivor空间**，默认比例为8:1:1。它们的具体作用将在下一篇文章讲解GC时介绍。
    - **老年代（Tenured/Old）**：在新生代中经历了多次（具体看虚拟机配置的阀值）GC后仍然存活下来的对象会进入老年代中。老年代中的对象生命周期较长，存活率比较高，在老年代中进行GC的频率相对而言较低，而且回收的速度也比较慢。
    - **永久代（Perm）**：永久代存储类信息、常量、静态变量、即时编译器编译后的代码等数据，对这一区域而言，Java虚拟机规范指出可以不进行垃圾收集，一般而言不会进行垃圾回收。
    - 新生代和老年代组成了Java堆的全部内存区域，永久代不属于堆空间

线程状态（阅读需要3分钟）

![](https://i.loli.net/2019/11/27/yCgoP8NfFLGt1Q6.png)

- 线程状态：
  - New：尚未启动的线程的线程状态
  - Runnable：可运行线程的线程状态，等待CPU调度
  - Blocked：线程阻塞等待监视器锁定的线程状态，处于synchronized同步代码块或方法中被阻塞。
  - Waiting：等待线程的线程状态。
  - Timed Waiting：具有指定等待时间的等待线程的线程状态。
    - Thread.sleep、Object.wait、Thread.join、LockSupport.parkNanos、LockSupport.parkUntil。
  - Terminated：终止线程状态。

## 线程终止（5min）

- 不正确的线程终止
  - Stop：终止线程并秦楚监控器锁的信息，但是可能导致线程安全问题，JDK不建议使用。
  - Destroy ：JDK未实现。
- 正确的线程终止
- interrupt

> 如果目标现场在调用Object class的wait()，wait(long)，或者wait(long,int)方法、join()、join(long,int)或者sleep(long,int)方法被阻塞，那么Interrupt生效，该线程的中断状态将被清除，抛出InterruptException异常。 
>
> 如果目标线程是被I/O或者NIO中的Channel所阻塞，同样，I/O操作会被中断或者返回特殊异常值。达到终止线程的目的。
>
> 如果以上条件都不满足，则会设置此线程的中断状态。

- 标志位

> 代码逻辑中，增加一个判断，用来控制线程执行的中止。

- interrupt和stop导致的原因：
  - stop为强行中止，故会导致数据不一致。



## CPU缓存和内存屏障(5min)

- CPU性能优化手段-缓存

> 解决CPU运算速度与内存读写速度不匹配的矛盾，提高程序运行的性能。

- 按照数据读取顺序和与CPU结合的紧密程度，CPU缓存可以分为一级缓存，二级缓存，部分高端CPU还具有三级缓存，每一级缓存中所储存的全部数据都是下一级缓存的一部分，这三种缓存的技术难度和制造成本是相对递减的，所以其容量也是相对递增的。

  - L1（CPU一级缓存）：

    - 致力于更快的速度
    - 直接与总线相连，位于CPU旁边，传输速度接近CPU处理速度。
    - 一般缓存容量为32-4096KB。
    - 每个CPU核心一个
    - 采用[哈弗结构](https://blog.csdn.net/u014470361/article/details/79774331)
    - 技术难度和制造成本最高。
    - 命中率已经很高
    - 分别有数据缓存和指令缓存

  - L2：

    - 提高命中率
    - 冯.诺伊曼模型
    - 每个CPU核心一个
    - 根据28定律，80%的的数据在都可以直接在一级缓存中找到，而剩下的20%的80%可以再二级缓存中找到，因此就是说，96%的数据都可以在一二级缓存中找到。

  - L3：

    - 进一步提高命中率
    - 所有核心共享一个3级缓存
    - 直接与内存相连
    - 冯.诺伊曼模型

  - 更多级缓存

    - Intel 的CPU已经有增加L4缓存的版本了，不过这里的L4主要用于解决核显和CPU之间交换数据，称为eDRAM。原先的核显没有显存，只能共享内存空间，限制了核显性能和效率。现在在CPU和GPU之间增加了128MB的eDRAM，让GPU与CPU做数据交换。

      另外服务器cpu也有四级缓存

    - ![](https://i.loli.net/2019/11/27/v5cOEtCqU3Mo4mI.png)

- 缓存存储

> cpu缓存和内存之间交换的数据,是长度固定的块,称为缓存航.通常是2^n,一般16~256字节不等.
>
> 大量缓存是通过硬件哈希表来实现的.这些哈希表有固定长度的哈系桶.
>
> 每个桶能够存储一个缓存行大小的数据.然后几个桶组成一路.也就是哈希值定位到的位置.
>
> 一般哈希都比较简单,单纯的使用4位来表示,因此也就是说有16路.每路多个桶.

- 缓存一致性协议（缓存同步协议）

  **MESI协议**，它规定每条缓存有个状态位，同时定义了下面四个状态：

  - **修改态(Modified)**-此cache行已被修改过(脏行),内容已不同于主存，为此cache专有；
  - **专有态(Exclusive)**-此cache行内容同于主存，但不出现于其它cache中；
  - **共享态(Shared)**-此cache行内容同于主存，但也出现于其它cache中；
  - **无效态(Invalid)**-此cache行内容无效(空行)。

  多处理器，单个CPU对缓存中数据进行了改动，需要通知给其它CPU。也就是意味着，CPU处理要控制自己的读写操作，还要监听其他CPU发出的通知，从而保证最终一致。

- CPU性能优化手段-指令重排：

> 当CPU写缓存时发现缓存区块正被其他CPU占用，为了提高CPU处理性能，可能将后面的读缓存命令优先执行。

- 需要遵守**as-if-serial**语义

- as-if-serial语义的意思指：不管怎么重排序(编译器和处理器为了提高并行度)，(单线程）程序的执行结果不能被改变。编译器，runtime和处理器都必须遵守as-if-serial语义。也就是说：编译器和处理器不会对存在数据依赖关系的操作做重排序。

- 两个问题：

  1、CPU高速缓存下有一个问题：

缓存中数据与主内存的数据并不是实时同步的，各CPU(或CPU核心)间缓存的数据也不是实时同步。

**在同一个时间点，各CPU所看到同一内存地址的数据的值可能是不一致的。**

​	2、CPU执行指令重排序优化下有一个问题：

虽然遵守了**as-if-serial**语义，单仅在单CPU自己执行的情况下能保证结果正确。多核多线程中，指令逻辑无法分辨因果关联，可能出现**乱序执行**，导致程序运行结果错误。

- 内存屏障

> 处理器提供了两个内存屏障指令(Memory Barrier)用于解决上述两个问题：

​	写内存屏障(Store Memory Barrier):在指令后插入Store Barrier,能让写入缓存中的最新数据更新写入主内存，让其他线程可见。强制写入主内存，这种显示调用，CPU就不会因为性能考虑而去对指令重排。

​	读内存屏障(Load Memory Barrier):在指令前插入Load Barrier,可以让高速缓存中的数据失效，强制从主内存加载数据。强制读取主内存内容，让CPU缓存与主内存保持一致，避免了缓存导致的一致性问题。

## 线程通信(30min)

> 想要实现多个线程之间的协同，如：线程执行先后顺序、获取某个线程执行的结果等等。

- 线程之间相互通信分为：

  - 文件共享

  - 网络共享

  - 共享变量

  - JDK提供的线程协调api

    - suspend/resume、wait/notify、park/unpark

  - suspend和resume（容易写出死锁的进程->已经被弃用）

    - 用法

    ```java
    //正确调用
    //Thread consumerThread;
    //挂起
    Thread.currentThread().suspend();
    //唤醒
    cousumerThread.resume();
    ```

    - 同步代码块中会发生死锁（suspend挂起之后并不会释放锁）

    ```java
    //死锁
    //挂起标志代码
    synchronized(this){
        Thread.currentThread().suspend();
    }
    //唤醒标志代码
    synchronized (this) {
        consumerThread.resume();
    }
    //会发生死锁，因为suspend不释放锁
    ```

    - suspend和resume调用顺序不正确会发生死锁。

    ```java
    //死锁
    //Thread consumerThread;
    //挂起,模拟处理业务代码用时5秒
    try{
        Thread.sleep(5000L);
    }catch(){
    
    }
    Thread.currentThread().suspend();
    //唤醒
    Thread.sleep(3000L);//模拟挂起线程没执行完之后使它进入等待
    cousumerThread.resume();
    //resume()必须在suspend()之后调用，本代演示了多线程中先执行了resume()后执行suspend(),
    //导致线程发生死锁
    ```

- wait/notify机制

  - wait会队同步代码块进行解锁，只能由同一对象锁的持有和线程调用，即只能写在同步代码块中。

  ```java
  public void waitNotifyTest() throws Exception {
      // 启动线程
      new Thread(() -> {
          synchronized (this) {
              while (baozidian == null) { // 如果没包子，则进入等待
                  try {
                      System.out.println("1、进入等待");
                      this.wait();
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          }
          System.out.println("2、买到包子，回家");
      }).start();
      // 3秒之后，生产一个包子
      Thread.sleep(3000L);
      baozidian = new Object();
      synchronized (this) {
          this.notifyAll();
          System.out.println("3、通知消费者");
      }
  }
  ```

  - wait和notify调用顺序不能颠倒，不然也会发生死锁

  ```java
  public void waitNotifyDeadLockTest() throws Exception {
      // 启动线程
      new Thread(() -> {
          if (baozidian == null) { // 如果没包子，则进入等待
              try {
                  Thread.sleep(5000L);
              } catch (InterruptedException e1) {
                  e1.printStackTrace();
              }
              synchronized (this) {
                  try {
                      System.out.println("1、进入等待");
                      this.wait();
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              }
          }
          System.out.println("2、买到包子，回家");
      }).start();
      // 3秒之后，生产一个包子
      Thread.sleep(3000L);
      baozidian = new Object();
      synchronized (this) {
          this.notifyAll();
          System.out.println("3、通知消费者");
      }
  }
  ```

- park/unpark机制

  - park/unpark机制为等待许可机制，可以多次调用unpark后再调用park，并且不要求调用顺序

  ```java
  public void parkUnparkTest() throws Exception {
      // 启动线程
      Thread consumerThread = new Thread(() -> {
          while (baozidian == null) { // 如果没包子，则进入等待
              System.out.println("1、进入等待");
              LockSupport.park();
          }
          System.out.println("2、买到包子，回家");
      });
      consumerThread.start();
      // 3秒之后，生产一个包子
      Thread.sleep(3000L);
      baozidian = new Object();
      LockSupport.unpark(consumerThread);
      System.out.println("3、通知消费者");
  }
  //park()函数为java.util.concurrent.locks.LockSupport包下的一个函数
  //unpark(parms)中参数必须为指定的线程对象
  ```

  - park不会释放锁，所以同步代码块中使用park/unpark会发生死锁

  ```java
  public void parkUnparkDeadLockTest() throws Exception {
      // 启动线程
      Thread consumerThread = new Thread(() -> {
          if (baozidian == null) { // 如果没包子，则进入等待
              System.out.println("1、进入等待");
              // 当前线程拿到锁，然后挂起
              synchronized (this) {
                  LockSupport.park();
              }
          }
          System.out.println("2、买到包子，回家");
      });
      consumerThread.start();
      // 3秒之后，生产一个包子
      Thread.sleep(3000L);
      baozidian = new Object();
      // 争取到锁以后，再恢复consumerThread
      synchronized (this) {
          LockSupport.unpark(consumerThread);
      }
      System.out.println("3、通知消费者");
  }
  //不能在同步代码块中使用
  ```

- 这些由**Java Api**提供的线程通信机制是之后多线程并发类的底层代码。

- 伪唤醒：是只线程并非应为notify、notifyall、unpark等api调用而唤醒，而是由更底层的原因导致唤醒，所以之前代码中应该在**循环中检查等待条件**。

## 线程封闭(10min)

> 多线程访问共享可变数据时，涉及到线程间数据同步的问题。并不是所有时候都要用到共享数据，所以就需要使用线程封闭。
>
> 数据都被封闭在各自的先之中，就不需要同步，这种通过将数据封闭在线程之中而避免使用同步的的技术称为**线程封闭**。

- 实现线程封闭的方法

  - ThreadLocal

    - java之中的一种特殊变量
    - 是一个线程级别的变量
  
```java
  //创建
  ThreadLocal<T> value = new ThreadLocal<>();
  //设值
  value.set(xxx);
  //取值
  value.get();
  //原理
  //TODO
```

```java
  /** 线程封闭示例 */
  public class Demo7 {
      /** threadLocal变量，每个线程都有一个副本，互不干扰 */
      public static ThreadLocal<String> value = new ThreadLocal<>();
  
      /**
  	 * threadlocal测试
  	 * 
  	 * @throws Exception
  	 */
      public void threadLocalTest() throws Exception {
  
          // threadlocal线程封闭示例
          value.set("这是主线程设置的123"); // 主线程设置值
          String v = value.get();
          System.out.println("线程1执行之前，主线程取到的值：" + v);
  
          new Thread(new Runnable() {
              @Override
              public void run() {
                  String v = value.get();
                  System.out.println("线程1取到的值：" + v);
                  // 设置 threadLocal
                  value.set("这是线程1设置的456");
  
                  v = value.get();
                  System.out.println("重新设置之后，线程1取到的值：" + v);
                  System.out.println("线程1执行结束");
              }
          }).start();
  
          Thread.sleep(5000L); // 等待所有线程执行结束
  
          v = value.get();
          System.out.println("线程1执行之后，主线程取到的值：" + v);
  
      }
  
      public static void main(String[] args) throws Exception {
          new Demo7().threadLocalTest();
      }
  }
```

- 栈封闭

> 局部变量，位于线程栈帧中，其他线程无法使用改变量




## 线程池(20min)

> 1、线程在java中是一个对象，更是操作系统的资源，线程创建、销毁需要时间。如果创建时间+销毁时间>执行时间，则会降低系统性能。
>
> 2、java对象占用堆内存，操作系统线程占用系统内存，根据jvm贵方，一个线程默认最大栈大小为1m，这个栈空间是需要从系统内存中分配。线程越多消耗的内存越多
>
> 3、操作系统需要频繁切换线程上下文，影响性能。
>
> ->线程池的推出为了方便控制线程数量。

- 线程池原理-概念
- ![](https://i.loli.net/2019/11/27/N9OLsx4C7TJUyV5.jpg)
  - 线程池管理器
    
    > 用于创建并管理线程池，包括创建线程池、销毁线程池、添加新任务
    
  - 工作线程
  
    > 线程池中线程，在没有任务时处于等待状态，可以循环的执行任务。
  
  - 任务接口
  
    > 每个任务必须实现的接口，以供工作线程调度任务的执行，他主要规定了任务的入口，任务执行往后的收尾工作，任务的执行状态等。
  
  - 任务队列
  
    > 用于存放没人处理的任务，提供一种缓存机制。
  
- 线程池API-接口定义和实现

![](https://i.loli.net/2019/11/27/cRPFDSfH92EIJLQ.jpg)

- ExecutorService方法定义

![](https://i.loli.net/2019/11/27/CzUkcSx8u7I6OZs.jpg)