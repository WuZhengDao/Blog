---
title: 从BIO到NIO
tags:
  - 网络编程
categories:
  - technology
  - java
date: 2019-12-13 20:54:59
---


### BIO网络编程

> 阻塞（Blocking）I/O：资源不可用时，IO请求一直阻塞，直到有反馈结果。
>
> 非阻塞（Non-Blocking）I/O：资源不可用时，IO请求离开返回，返回数据标识资源不可用。
>
> 同步（synchronous）I/O：应用阻塞在发送或者接收数据的状态，知道数据成功传输或返回失败
>
> 异步（asynchronous）I/O：应用发送或接收数据后立刻放回，诗句处理是异步执行的。
>
> 阻塞和非阻塞式获取资源的方式，同步/异步是程序如何处理资源的逻辑设计，并没有太大联系。

```java
//客户端代码
public class BIOClient {
	private static Charset charset = Charset.forName("UTF-8");

	public static void main(String[] args) throws Exception {
		Socket s = new Socket("localhost", 8080);
		OutputStream out = s.getOutputStream();
		Scanner scanner = new Scanner(System.in);
		System.out.println("请输入：");
		String msg = scanner.nextLine();
		out.write(msg.getBytes(charset)); // 阻塞，写完成
		scanner.close();
		s.close();
	}
}
```

```java
//BioServer1:只能相应单线程
public class BIOServer {

    public static void main(String[] args) throws Exception {
        //创建绑定到特殊端口的服务器套接字
        ServerSocket serverSocket = new ServerSocket(8080);
        System.out.println("服务器启动成功");
        while (!serverSocket.isClosed()) {
            Socket request = serverSocket.accept();// 阻塞
            System.out.println("收到新连接 : " + request.toString());
            try {
                // 接收数据、打印
                InputStream inputStream = request.getInputStream(); // net + i/o
                BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream, "utf-8"));
                String msg;
                while ((msg = reader.readLine()) != null) { // 没有数据，阻塞
                    if (msg.length() == 0) {
                        break;
                    }
                    System.out.println(msg);
                }
                System.out.println("收到数据,来自："+ request.toString());
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    request.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        serverSocket.close();
    }
}
```

- inputStream、outputStream、accept都是阻塞的，即没有接收到信号则会一直等待。

- 该代码服务端仅能处理一个客户端的请求。

- 改进代码满足可以实现多个客户端请求。

- 引进多线程技术满足相应多个客户端请求

  ```java
  //BioServer2：引用线程池、响应多个客户端请求
  public class BIOServer1 {
      //使用线程池技术，使用Executor框架快速搭建线程池
      private static ExecutorService threadPool = Executors.newCachedThreadPool();
  
      public static void main(String[] args) throws Exception {
          ServerSocket serverSocket = new ServerSocket(8080);
          System.out.println("tomcat 服务器启动成功");
          while (!serverSocket.isClosed()) {
              Socket request = serverSocket.accept();
              System.out.println("收到新连接 : " + request.toString());
              //往线程池中添加任务，修改成submit(()->{})也一样可行
              threadPool.execute(() -> {
                  try {
                      // 接收数据、打印
                      InputStream inputStream = request.getInputStream();
                      BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream, "utf-8"));
                      String msg;
                      while ((msg = reader.readLine()) != null) { // 阻塞
                          if (msg.length() == 0) {
                              break;
                          }
                          System.out.println(msg);
                      }
                      System.out.println("收到数据,来自："+ request.toString());
                  } catch (IOException e) {
                      e.printStackTrace();
                  } finally {
                      try {
                          request.close();
                      } catch (IOException e) {
                          e.printStackTrace();
                      }
                  }
              });
          }
          serverSocket.close();
      }
  }
  ```

  - 引进多线程技术，使用线程池解决无法相应多个客户端的弊端。
  - 对一些特殊协议无法解析（http协议等）
  - 解决方案：给返回的OutputStream中添加相应的报文头，使得返回的数据可以被浏览器解析

### NIO网络编程

> JAVA1.4，提供了新的JAVAI/O操作非阻塞API

- 核心组件

  - [Buffer缓冲区](https://docs.oracle.com/javase/7/docs/api/java/nio/ByteBuffer.html)

    - > 本质上是一个可以写入数据的内存块（类似数组，源码中也是被声明为数组类型），然后可以再次读取。

    - Buffer进行数据写入与读取

      - 申请缓冲区

      - ```java
        // 构建一个byte字节缓冲区，容量是4，从堆外内存申请，速度快，但是存在oom风险，故需要jvm参数设置最大堆外内存MaxDirectMemorySize。
        ByteBuffer byteBuffer = ByteBuffer.allocateDirect(4);
        //从java堆heap中申请，速度较慢
        ByteBuffer byteBuffer = ByteBuffer.allocate(4);
        ```

      - 将数据写入缓存区

        ```java
        bytebuffer.put((byte)1);
        ```

      - 调用buffer.flip()，转换为读取模式

        ```java
        byteBuffer.flip();//作用为转换为读取模式，即position从0开始
        ```

      - 缓冲区读取数据

        ```java
        byteBuffer.get();//获取当前position的数据
        ```

      - 调用buffer.clear()或buffer.compact()清除缓冲区

        ```java
        byteBuffer.clear();//全删
        byteBuffer.compact();//删已读的
        ```

    - Buffer三个重要属性

      - capacity容量：Buffer具有一定固定的大小，即容量。
      - position位置：写入模式时代表写数据的位置。读取模式时代码读区数据的位置。（类似于数组下标）
      - limit限制：写入模式，limit等于buffer的容量。读取模式下，limit等于写入的数据量。（防止溢出、读到错误数据）

    - 实例：

    ```java
    public class BufferDemo {
        public static void main(String[] args) {
            /* 从heap
            构建一个byte字节缓冲区，容量是4
            ByteBuffer byteBuffer = ByteBuffer.allocate(4);
            */
            //从堆外内存构建一个ByteBuffer缓冲区
            ByteBuffer byteBuffer = ByteBuffer.allocateDirect(4);
            // 默认写入模式，查看三个重要的指标
            System.out.println(String.format("初始化：capacity容量：%s, position位置：%s, limit限制：%s", byteBuffer.capacity(),
                    byteBuffer.position(), byteBuffer.limit()));
            // 写入2字节的数据
            byteBuffer.put((byte) 1);
            byteBuffer.put((byte) 2);
            byteBuffer.put((byte) 3);
            // 再看数据
            System.out.println(String.format("写入3字节后，capacity容量：%s, position位置：%s, limit限制：%s", byteBuffer.capacity(),
                    byteBuffer.position(), byteBuffer.limit()));
    
            // 转换为读取模式(不调用flip方法，也是可以读取数据的，但是position记录读取的位置不对)
            System.out.println("#######开始读取");
            byteBuffer.flip();
            byte a = byteBuffer.get();
            System.out.println(a);
            byte b = byteBuffer.get();
            System.out.println(b);
            System.out.println(String.format("读取2字节数据后，capacity容量：%s, position位置：%s, limit限制：%s", byteBuffer.capacity(),
                    byteBuffer.position(), byteBuffer.limit()));
    
            // 继续写入3字节，此时读模式下，limit=3，position=2.继续写入只能覆盖写入一条数据
            // clear()方法清除整个缓冲区。compact()方法仅清除已阅读的数据。转为写入模式
            byteBuffer.compact(); // buffer : 1 , 3
            byteBuffer.put((byte) 3);
            byteBuffer.put((byte) 4);
            byteBuffer.put((byte) 5);
            System.out.println(String.format("最终的情况，capacity容量：%s, position位置：%s, limit限制：%s", byteBuffer.capacity(),
                    byteBuffer.position(), byteBuffer.limit()));
            // rewind() 重置position为0
            // mark() 标记position的位置
            // reset() 重置position为上次mark()标记的位置
    
        }
    }
    ```

  - [Channel通道](https://docs.oracle.com/javase/7/docs/api/java/nio/channels/package-summary.html)

    - Channel为NIO通道，常见分类为：

      - FileChannel, 文件操作
      - DatagramChannel, UDP 操作
      - SocketChannel, TCP 操作
      - ServerSocketChannel, TCP 操作, 使用在服务器端

    - SocketChannel

      - > Nio中用于创建TCP网络连接，类似java.net.Socket。

      - 有两种创建SocketChannel的方式：

        - 客户端主动发起和服务器的连接

        ```java
        //从客户端主动发起连接
        SocketChannel socketChannel = SocketChannel.open();
        socketChannel.configureBlocking(false);//设置为非阻塞模式
        socketChannel.connect(new InetSocketAddress("http://163.com",80));
        
        //write()方法：发送请求数据-向通道写入数据，write()在尚未写入任何内容时就可能返回了。需要在循环中调用write()
        channel.write(byteBuffer);
        
        //read()方法：可能直接返回而根本不读取任何数据，根据返回的int值判断读取了多少字节
        socketChannel.read(byteBuffer);int
        
        //关闭连接
        socketChannel.close();
        ```

        - 服务端获取的新连接

        ```java
        //创建网络端
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        //设置为非阻塞模式
        serverSocketChannel.configureBlocking(false);
        //绑定端口
        serverSocketChannel.socket().bind(new InetSocketAddress(8080));
        while(true){
            //获取新的TCP连接通道
            SocketChannel socketChannel = serverSocketChannel.accpet();
            //在Nio过程中该通道处于非阻塞模式，如果没有挂起的连接则返回null,所以必须检查返回的SocketChannel是否为null;
        	if(socketChannel!=null){
                //tcp请求 读取/响应
            }
        }
        ```

        - 实例

        ```java
        //client
        public class NIOClient {
            public static void main(String[] args) throws Exception {
                SocketChannel socketChannel = SocketChannel.open();
                //设置为非阻塞模式
                socketChannel.configureBlocking(false);
                socketChannel.connect(new InetSocketAddress("127.0.0.1", 8080));
                while (!socketChannel.finishConnect()) {
                    // 没连接上,则一直等待
                    Thread.yield();
                }
                Scanner scanner = new Scanner(System.in);
                System.out.println("请输入：");
                // 发送内容
                String msg = scanner.nextLine();
                ByteBuffer buffer = ByteBuffer.wrap(msg.getBytes());
                while (buffer.hasRemaining()) {
                    socketChannel.write(buffer);
                }
                // 读取响应
                System.out.println("收到服务端响应:");
                ByteBuffer requestBuffer = ByteBuffer.allocate(1024);
        
                while (socketChannel.isOpen() && socketChannel.read(requestBuffer) != -1) {
                    // 长连接情况下,需要手动判断数据有没有读取结束 (此处做一个简单的判断: 超过0字节就认为请求结束了)
                    if (requestBuffer.position() > 0) break;
                }
                requestBuffer.flip();
                byte[] content = new byte[requestBuffer.limit()];
                requestBuffer.get(content);
                System.out.println(new String(content));
                scanner.close();
                socketChannel.close();
            }
        }
        ```

        - NIOServer：

        ```java
      //NIOServer：使用NIO获取客户端发送的消息
        public class NIOServer {
            public static void main(String[] args) throws Exception {
                // 创建网络服务端
                ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
                serverSocketChannel.configureBlocking(false); // 设置为非阻塞模式
                serverSocketChannel.socket().bind(new InetSocketAddress(8080)); // 绑定端口
                System.out.println("启动成功");
                while (true) {
                    SocketChannel socketChannel = serverSocketChannel.accept(); // 获取新tcp连接通道
                    // tcp请求 读取/响应
                    if (socketChannel != null) {
                        System.out.println("收到新连接 : " + socketChannel.getRemoteAddress());
                        socketChannel.configureBlocking(false); // 默认是阻塞的,一定要设置为非阻塞
                        try {
                            ByteBuffer requestBuffer = ByteBuffer.allocate(1024);
                            while (socketChannel.isOpen() && socketChannel.read(requestBuffer) != -1) {
                                // 长连接情况下,需要手动判断数据有没有读取结束 (此处做一个简单的判断: 超过0字节就认为请求结束了)
                                if (requestBuffer.position() > 0) break;
                            }
                            if(requestBuffer.position() == 0) continue; // 如果没数据了, 则不继续后面的处理
                            requestBuffer.flip();
                            byte[] content = new byte[requestBuffer.limit()];
                            requestBuffer.get(content);
                            System.out.println(new String(content));
                            System.out.println("收到数据,来自："+ socketChannel.getRemoteAddress());
        
                            // 响应结果 200
                            String response = "HTTP/1.1 200 OK\r\n" +
                                    "Content-Length: 11\r\n\r\n" +
                                    "Hello World";
                            ByteBuffer buffer = ByteBuffer.wrap(response.getBytes());
                            while (buffer.hasRemaining()) {
                                socketChannel.write(buffer);// 非阻塞
                            }
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                    }
                }
                // 用到了非阻塞的API, 在设计上,和BIO可以有很大的不同.继续改进
            }
        }
        ```
        
        - 由于需要使用循环判断获取长连接数据是否完成，故在处理多线程过程中即使有多个客户端进行访问，但任然只有一个客户端访问数据，在BIO中采取了线程池方案来优化，但是在NIO中提出一个线程处理多个客户端请求的解决方案。
        - 解决方案：使用数组保存所有的连接，循环检查当没有新的连接时，就去处理现有的连接数据，处理完删除。
        
        - NIOserver1
        
        ```java
        public class NIOServer1 {
            /**
             * 已经建立连接的集合
             */
            private static ArrayList<SocketChannel> channels = new ArrayList<>();
        
            public static void main(String[] args) throws Exception {
                // 创建网络服务端
                ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
                serverSocketChannel.configureBlocking(false); // 设置为非阻塞模式
                serverSocketChannel.socket().bind(new InetSocketAddress(8080)); // 绑定端口
                System.out.println("启动成功");
                while (true) {
                    SocketChannel socketChannel = serverSocketChannel.accept(); // 获取新tcp连接通道
                        // tcp请求 读取/响应
                        if (socketChannel != null) {
                        System.out.println("收到新连接 : " + socketChannel.getRemoteAddress());
                        socketChannel.configureBlocking(false); // 默认是阻塞的,一定要设置为非阻塞
                        channels.add(socketChannel);
                    } else {
                        // 没有新连接的情况下,就去处理现有连接的数据,处理完的就删除掉
                        Iterator<SocketChannel> iterator = channels.iterator();
                        while (iterator.hasNext()) {
                            SocketChannel ch = iterator.next();
                            try {
                                ByteBuffer requestBuffer = ByteBuffer.allocate(1024);
        
                                if (ch.read(requestBuffer) == 0) {
                                    // 等于0,代表这个通道没有数据需要处理,那就待会再处理
                                    continue;
                                }
                                while (ch.isOpen() && ch.read(requestBuffer) != -1) {
                                    // 长连接情况下,需要手动判断数据有没有读取结束 (此处做一个简单的判断: 超过0字节就认为请求结束了)
                                    if (requestBuffer.position() > 0) break;
                                }
                                if(requestBuffer.position() == 0) continue; // 如果没数据了, 则不继续后面的处理
                                requestBuffer.flip();
                                byte[] content = new byte[requestBuffer.limit()];
                                requestBuffer.get(content);
                                System.out.println(new String(content));
                                System.out.println("收到数据,来自：" + ch.getRemoteAddress());
        
                                // 响应结果 200
                                String response = "HTTP/1.1 200 OK\r\n" +
                                        "Content-Length: 11\r\n\r\n" +
                                        "Hello World";
                                ByteBuffer buffer = ByteBuffer.wrap(response.getBytes());
                                while (buffer.hasRemaining()) {
                                    ch.write(buffer);
                                }
                                iterator.remove();
                            } catch (IOException e) {
                                e.printStackTrace();
                                iterator.remove();
                            }
                        }
                    }
                }
                // 用到了非阻塞的API, 再设计上,和BIO可以有很大的不同
                // 问题: 轮询通道的方式,低效,浪费CPU
            }
        }
        ```
        
        	- 尽管解决了不能够处理多个客户端的问题，但是代码由于需要使用轮询的方式等待其他channel连接，故效率低下。
        	- 解决方案：使用Selector选择器来对channel连接进行筛选。
  
  - Selector选择器
  
    > Selector是一个JavaNIO组件，可以检查一个或者多个NIO通道，并确定那些通带已经准备好进行读取或写入。实现单线程管理多个通道从而管理多个网络连接。
  
    - 一个线程使用Selector监听多个channel的不同事件，四个事件分别对用SelectorKey四个常量：
      - Connect连接（SelectionKey.OP_CONNECT）
      - Accept准备就绪（OP_ACCEPT）
      - Read读取（OP_READ）
      - Write写入（OP_WRITE）
  
    ```java
    /**
     * 结合Selector实现的非阻塞服务端(放弃对channel的轮询,借助消息通知机制)
     */
    public class NIOServerV2 {
    
        public static void main(String[] args) throws Exception {
            // 1. 创建网络服务端ServerSocketChannel
            ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
            serverSocketChannel.configureBlocking(false); // 设置为非阻塞模式
    
            // 2. 构建一个Selector选择器,并且将channel注册上去
            Selector selector = Selector.open();
            SelectionKey selectionKey = serverSocketChannel.register(selector, 0, serverSocketChannel);// 将serverSocketChannel注册到selector
            selectionKey.interestOps(SelectionKey.OP_ACCEPT); // 对serverSocketChannel上面的accept事件感兴趣(serverSocketChannel只能支持accept操作)
    
            // 3. 绑定端口
            serverSocketChannel.socket().bind(new InetSocketAddress(8080));
    
            System.out.println("启动成功");
    
            while (true) {
                // 不再轮询通道,改用下面轮询事件的方式.select方法有阻塞效果,直到有事件通知才会有返回
                selector.select();
                // 获取事件
                Set<SelectionKey> selectionKeys = selector.selectedKeys();
                // 遍历查询结果e
                Iterator<SelectionKey> iter = selectionKeys.iterator();
                while (iter.hasNext()) {
                    // 被封装的查询结果
                    SelectionKey key = iter.next();
                    iter.remove();
                    // 关注 Read 和 Accept两个事件
                    if (key.isAcceptable()) {
                        ServerSocketChannel server = (ServerSocketChannel) key.attachment();
                        // 将拿到的客户端连接通道,注册到selector上面
                        SocketChannel clientSocketChannel = server.accept(); // mainReactor 轮询accept
                        clientSocketChannel.configureBlocking(false);
                        clientSocketChannel.register(selector, SelectionKey.OP_READ, clientSocketChannel);
                        System.out.println("收到新连接 : " + clientSocketChannel.getRemoteAddress());
                    }
    
                    if (key.isReadable()) {
                        SocketChannel socketChannel = (SocketChannel) key.attachment();
                        try {
                            ByteBuffer requestBuffer = ByteBuffer.allocate(1024);
                            while (socketChannel.isOpen() && socketChannel.read(requestBuffer) != -1) {
                                // 长连接情况下,需要手动判断数据有没有读取结束 (此处做一个简单的判断: 超过0字节就认为请求结束了)
                                if (requestBuffer.position() > 0) break;
                            }
                            if(requestBuffer.position() == 0) continue; // 如果没数据了, 则不继续后面的处理
                            requestBuffer.flip();
                            byte[] content = new byte[requestBuffer.limit()];
                            requestBuffer.get(content);
                            System.out.println(new String(content));
                            System.out.println("收到数据,来自：" + socketChannel.getRemoteAddress());
                            // TODO 业务操作 数据库 接口调用等等
    
                            // 响应结果 200
                            String response = "HTTP/1.1 200 OK\r\n" +
                                    "Content-Length: 11\r\n\r\n" +
                                    "Hello World";
                            ByteBuffer buffer = ByteBuffer.wrap(response.getBytes());
                            while (buffer.hasRemaining()) {
                                socketChannel.write(buffer);
                            }
                        } catch (IOException e) {
                            // e.printStackTrace();
                            key.cancel(); // 取消事件订阅
                        }
                    }
                }
                selector.selectNow();
            }
            // 问题: 此处一个selector监听所有事件,一个线程处理所有请求事件. 会成为瓶颈! 要有多线程的运用
        }
    }
    ```
  
    ```java
    /**
     * NIO selector 多路复用reactor线程模型
     */
    public class NIOServerV3 {
        /** 处理业务操作的线程 */
        private static ExecutorService workPool = Executors.newCachedThreadPool();
    
        /**
         * 封装了selector.select()等事件轮询的代码
         */
        abstract class ReactorThread extends Thread {
    
            Selector selector;
            LinkedBlockingQueue<Runnable> taskQueue = new LinkedBlockingQueue<>();
    
            /**
             * Selector监听到有事件后,调用这个方法
             */
            public abstract void handler(SelectableChannel channel) throws Exception;
    
            private ReactorThread() throws IOException {
                selector = Selector.open();
            }
    
            volatile boolean running = false;
    
            @Override
            public void run() {
                // 轮询Selector事件
                while (running) {
                    try {
                        // 执行队列中的任务
                        Runnable task;
                        while ((task = taskQueue.poll()) != null) {
                            task.run();
                        }
                        selector.select(1000);
    
                        // 获取查询结果
                        Set<SelectionKey> selected = selector.selectedKeys();
                        // 遍历查询结果
                        Iterator<SelectionKey> iter = selected.iterator();
                        while (iter.hasNext()) {
                            // 被封装的查询结果
                            SelectionKey key = iter.next();
                            iter.remove();
                            int readyOps = key.readyOps();
                            // 关注 Read 和 Accept两个事件
                            if ((readyOps & (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != 0 || readyOps == 0) {
                                try {
                                    SelectableChannel channel = (SelectableChannel) key.attachment();
                                    channel.configureBlocking(false);
                                    handler(channel);
                                    if (!channel.isOpen()) {
                                        key.cancel(); // 如果关闭了,就取消这个KEY的订阅
                                    }
                                } catch (Exception ex) {
                                    key.cancel(); // 如果有异常,就取消这个KEY的订阅
                                }
                            }
                        }
                        selector.selectNow();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
    
            private SelectionKey register(SelectableChannel channel) throws Exception {
                // 为什么register要以任务提交的形式，让reactor线程去处理？
                // 因为线程在执行channel注册到selector的过程中，会和调用selector.select()方法的线程争用同一把锁
                // 而select()方法实在eventLoop中通过while循环调用的，争抢的可能性很高，为了让register能更快的执行，就放到同一个线程来处理
                FutureTask<SelectionKey> futureTask = new FutureTask<>(() -> channel.register(selector, 0, channel));
                taskQueue.add(futureTask);
                return futureTask.get();
            }
    
            private void doStart() {
                if (!running) {
                    running = true;
                    start();
                }
            }
        }
    
        private ServerSocketChannel serverSocketChannel;
        // 1、创建多个线程 - accept处理reactor线程 (accept线程)
        private ReactorThread[] mainReactorThreads = new ReactorThread[1];
        // 2、创建多个线程 - io处理reactor线程  (I/O线程)
        private ReactorThread[] subReactorThreads = new ReactorThread[8];
    
        /**
         * 初始化线程组
         */
        private void newGroup() throws IOException {
            // 创建IO线程,负责处理客户端连接以后socketChannel的IO读写
            for (int i = 0; i < subReactorThreads.length; i++) {
                subReactorThreads[i] = new ReactorThread() {
                    @Override
                    public void handler(SelectableChannel channel) throws IOException {
                        // work线程只负责处理IO处理，不处理accept事件
                        SocketChannel ch = (SocketChannel) channel;
                        ByteBuffer requestBuffer = ByteBuffer.allocate(1024);
                        while (ch.isOpen() && ch.read(requestBuffer) != -1) {
                            // 长连接情况下,需要手动判断数据有没有读取结束 (此处做一个简单的判断: 超过0字节就认为请求结束了)
                            if (requestBuffer.position() > 0) break;
                        }
                        if (requestBuffer.position() == 0) return; // 如果没数据了, 则不继续后面的处理
                        requestBuffer.flip();
                        byte[] content = new byte[requestBuffer.limit()];
                        requestBuffer.get(content);
                        System.out.println(new String(content));
                        System.out.println(Thread.currentThread().getName() + "收到数据,来自：" + ch.getRemoteAddress());
    
                        // TODO 业务操作 数据库、接口...
                        workPool.submit(() -> {
                        });
    
                        // 响应结果 200
                        String response = "HTTP/1.1 200 OK\r\n" +
                                "Content-Length: 11\r\n\r\n" +
                                "Hello World";
                        ByteBuffer buffer = ByteBuffer.wrap(response.getBytes());
                        while (buffer.hasRemaining()) {
                            ch.write(buffer);
                        }
                    }
                };
            }
    
            // 创建mainReactor线程, 只负责处理serverSocketChannel
            for (int i = 0; i < mainReactorThreads.length; i++) {
                mainReactorThreads[i] = new ReactorThread() {
                    AtomicInteger incr = new AtomicInteger(0);
    
                    @Override
                    public void handler(SelectableChannel channel) throws Exception {
                        // 只做请求分发，不做具体的数据读取
                        ServerSocketChannel ch = (ServerSocketChannel) channel;
                        SocketChannel socketChannel = ch.accept();
                        socketChannel.configureBlocking(false);
                        // 收到连接建立的通知之后，分发给I/O线程继续去读取数据
                        int index = incr.getAndIncrement() % subReactorThreads.length;
                        ReactorThread workEventLoop = subReactorThreads[index];
                        workEventLoop.doStart();
                        SelectionKey selectionKey = workEventLoop.register(socketChannel);
                        selectionKey.interestOps(SelectionKey.OP_READ);
                        System.out.println(Thread.currentThread().getName() + "收到新连接 : " + socketChannel.getRemoteAddress());
                    }
                };
            }
    
    
        }
    
        /**
         * 初始化channel,并且绑定一个eventLoop线程
         *
         * @throws IOException IO异常
         */
        private void initAndRegister() throws Exception {
            // 1、 创建ServerSocketChannel
            serverSocketChannel = ServerSocketChannel.open();
            serverSocketChannel.configureBlocking(false);
            // 2、 将serverSocketChannel注册到selector
            int index = new Random().nextInt(mainReactorThreads.length);
            mainReactorThreads[index].doStart();
            SelectionKey selectionKey = mainReactorThreads[index].register(serverSocketChannel);
            selectionKey.interestOps(SelectionKey.OP_ACCEPT);
        }
    
        /**
         * 绑定端口
         *
         * @throws IOException IO异常
         */
        private void bind() throws IOException {
            //  1、 正式绑定端口，对外服务
            serverSocketChannel.bind(new InetSocketAddress(8080));
            System.out.println("启动完成，端口8080");
        }
    
        public static void main(String[] args) throws Exception {
            NIOServerV3 nioServerV3 = new NIOServerV3();
            nioServerV3.newGroup(); // 1、 创建main和sub两组线程
            nioServerV3.initAndRegister(); // 2、 创建serverSocketChannel，注册到mainReactor线程上的selector上
            nioServerV3.bind(); // 3、 为serverSocketChannel绑定端口
        }
    }
    ```
  
    