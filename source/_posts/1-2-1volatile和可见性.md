---
title: 线程可见性、内存模型与volatile
tags:
  - 高并发编程
categories:
  - technology
  - java
date: 2019-11-29 03:22:46
---



> JAVA线程之间的通信对于程序员完全透明，所以内存可见性问题很容易困扰JAVA程序员，本博客参考《JAVA并发编程的艺术》、《网易高级开发工程师网课》等内容对线程可见性和内存模型进行一些总结。

## Java内存模型基础

- 并发编程中存在2个问题

  - 线程间通信

    - > 指线程间以何种机制来交换信息

    - 通信机制与并发模型

      - 内存共享模型
        - 程序之间共享程序之间的公共状态，通过读- 写内存中的共状态进行隐式通信。
      - 消息传递模型
        - 线程之间没有公共状态，线程之间必须通过发送消息来显式进行线程间通信。

  - 线程同步

    - > 程序中用于控制不同线程间操作发生的相对顺序的机制。

    - 同步机制与并发模型
      - 内存共享模型
        - 同步是显示进行的。程序员必须显示指定某个方法或某端代码需要在线程之间互斥执行。
      - 消息传递模型
        - 同步是隐式进行的。因为消息发送必须在消息接收之前。

  - JAVA的并发模型采用的是共享内存模型

- JAVA内存模型抽象结构与JMM

  ![](https://i.loli.net/2019/12/02/fd7PtuCRopE8snB.jpg)

  - 共享变量

    - 所有实例域、静态域和数组元素都存储在堆内存内 ，堆内存在线程之间共享。
    - 存在内存可见性问题。

  - 局部变量、方法定义参数、异常处理器此参数

    - 不会在线程之间共享。

## JMM

  - > 官方定义请查看：jsr133第5章。（我保证你看完之后就不想看了）这里通俗的解释为：**定义了一系列线程间操作可见性问题的规范。**（主要是围绕happens-before原则打开，想知道详细设计原理请查看官方文档或者《JAVA并发编程的艺术》第三章，看完弄懂估计要一上午，蛮重要的，但是没啥时间详细总结，等之后有空给挖一挖）。

  - Java线程之间的通信由Java内存模型控制，JMM决定共享变量的写入何时堆另一个线程可见。

## 重排序与JMM

![](https://i.loli.net/2019/12/02/xAkN1QmwWV8fa94.jpg)

- > 为了提高性能，编译器和处理器常常会对指令做重排序。

- 分类：

  - 编译器优化的重排序。（JAVA中[JIT(Just In Time compiler)](https://www.ibm.com/developerworks/cn/java/j-lo-just-in-time/index.html)参与编译器优化重排序<.class文件翻译成机器码指令的时编译的>）
    - 编译器级别。
    - 编译器在不改变单线程语义的前提下，可以重新安排语句的执行顺序。
  - 指令级别的重排序
    - 处理器级别。
    - 现代处理器采用了指令级并线技术（ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
  - 内存系统重排序
    - 处理器级别
    - 由于处理器使用缓存的读/写缓冲区，这使得加载和存储操作看上去可能是在乱码执行。

- JMM属于语言级别的内存模型，它确保在不同的编译器和不同的处理器平台上，通过禁止特定类型的编译器重排序和处理器重排序，为程序员提供一致的内存可见性保证。

- 数据依赖性（有些地方被称为冲突）与重排序

  - > 如果两个操作访问同一个变量（共享变量），且这2个操作中有一个为**写操作**，此时2个操作之间就存在数据依赖性。

  - 对于存在数据依赖性的操作，重排序这两个操作的执行顺序，程序的执行结果就会改变。

  - 编译器和处理器在重排序时，会遵守数据依赖性，编译器和处理器不会改变存在数据依赖关系的两个操作的执行顺序。

- as-if-serial语义与重排序（重排序原则）

  - > 不管怎么重排序（程序和处理器为了提高并行度），（单线程）程序的执行结果不能被改变。
  
- happens-before（可见性保证原则）

  - > JSR-133使用happens-before的概念来阐述操作之间的内存可见性。在JMM中，如果一个操作**执行的结果**需要对另一个操作可见，那么这2个操作之间必须要存在happens-before关系。

  - 两个操作之间具有happens-before关系，并不意味着前一个操作必须要在后一个操作之前执行！happens-before仅仅要求前一个操作（执行结果）对后一个操作可见。

## volatile

- > JAVA内存模型规定：
  >
  > 对volatile变量v的写入与其他所有线程后续对v的读同步。

- volatile特性

  - 可见性
    - 对一个volatile变量的读，总能看到（任意进程）对这个volatile变量最后的写入。
  - 原子性（操作系统保证）
    - 对于任意单个volatile变量的读/写具有原子性，但类似于`volatile++`这种复合操作不具备原子性。

- volatile与happens-before

- volatile读-写内存语义（与锁的释放-获取具有相同的内存效果）
  - 写：当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存。
  - 读：当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效，线程接下来将直接从主内存中读取共享变量。
  
- 重排序规则

  - | 能否重排   |           | 第二个操作 |            |
    | ---------- | --------- | :--------: | :--------: |
    | 第一个操作 | 普通读/写 | volatile读 | volatile写 |
    | 普通读/写  |           |            |     NO     |
    | volatile读 | NO        |     NO     |     NO     |
    | volatile写 |           |     NO     |     NO     |

- volatile的实现
  
  - 通过设置内存屏障
  
- 代码实例与分析

  ```java
  //代码1
  public class Demo1Visibility {
      int i = 0;
      boolean isRunning = true;
  
      public static void main(String args[]) throws InterruptedException {
          Demo1Visibility demo = new Demo1Visibility();
  
          new Thread(new Runnable() {
              @Override
              public void run() {
                  System.out.println("here i am...");
                  while(demo.isRunning){
                      synchronized (this){
                          demo.i++;
                      }
                  }
                  System.out.println(demo.i);
              }
          }).start();
  
          Thread.sleep(3000L);
          demo.isRunning = false;
          System.out.println("shutdown...");
      }
  }
  //代码1javap -v
  {
      int i;
      descriptor: I
          flags:
  
      boolean isRunning;
      descriptor: Z
          flags:
      		···
  }
  //代码2
  public class Demo1Visibility {
      int i = 0;
      volatile boolean isRunning = true;
  
      public static void main(String args[]) throws InterruptedException {
          Demo1Visibility demo = new Demo1Visibility();
  
          new Thread(new Runnable() {
              @Override
              public void run() {
                  System.out.println("here i am...");
                  while(demo.isRunning){
                      synchronized (this){
                          demo.i++;
                      }
                  }
                  System.out.println(demo.i);
              }
          }).start();
  
          Thread.sleep(3000L);
          demo.isRunning = false;
          System.out.println("shutdown...");
      }
  }
  
  //代码 2javap -v
  {
      int i;
      descriptor: I
          flags:
  
      volatile boolean isRunning;
      descriptor: Z
          flags: ACC_VOLATILE
              ···
  }
  //结论：在代码1和代码2中不同的仅仅是关键字isRunning的申明中包含不包含关键字volatile，然而在反编译之后的代码中可以看到javap字节码中变量z的标志位flags一个为空，另一个为ACC_VOLATILE。
  ```

  - volatile字节码解析：
    - 前面提到过volatile字ACC_VOLATILE在字节码上的含义为阻碍缓存。

## 锁与可见性

> 很多时候我们对锁的理解往往停留在保证原子性上，却忽略了锁同时也保证了可见性。

- 可以简单的理解为A线程B线程同时操作一个变量c，A获取锁对c操作，操作完成之后A释放锁，B获取锁对c操作，操作完成之后B释放锁。在这个过程中，A操作happens-before B操作即A修改c变量的操作对B线程可见。
- 锁获取与volatile读有相同的内存语义。
- 详细可以参考《JAVA并发编程的艺术--3.5锁的内存语义》。

