---
title: OSI网络模型与tcp、udp协议
tags:
  - 网络模型与tcp/udp协议
categories:
  - technology
  - java
date: 2019-12-13 19:33:26
---

### 网络模型与各层协议

![](https://i.loli.net/2019/12/13/VN8QICoMRZOxu5W.png)

- 其中物理层、数据链路层、网络层被称为低三层；会话层、表示层、应用层被称为高三层。

### TCP协议下的数据封装与解封

![](https://i.loli.net/2019/12/13/sd5rpSqJYlDvQZc.jpg)

- TCP协议

  > 面向连接、可靠有序、字节流传输服务，位于传输层

- TCP协议报文头

  ![](https://i.loli.net/2019/12/13/wWum3dfEDgOpY1z.jpg)

  - 标志位说明（值位0或1）

    - URG：紧急指针
    - ACK：确认序号
    - PSH：有DATA数据传输
    - RST：连接重置
    - SYN：建立连接
    - FIN：关闭连接

  - TCP三次握手

    ![](https://i.loli.net/2019/12/13/OVfXWwUPqSAY2jh.jpg)

  三次握手（Three-Way Handshake）即建立TCP连接，就是指建立一个TCP连接时，需要客户端和服务端总共发送3个包以确认连接的建立。在socket编程中，这一过程由客户端执行connect来触发。

  ​	第一次握手：Client将标志位SYN置为1，随机产生一个值seq=J，并将该数据包发送给Server，Client进入SYN_SENT状态，等待Server确认。

  ​	第二次握手：Server收到数据包后由标志位SYN=1知道Client请求建立连接，Server将标志位SYN和ACK都置为1，ack=J+1，随机产生一个值seq=K，并将该数据包发送给Client以确认连接请求，Server进入SYN_RCVD状态。

  ​	第三次握手：Client收到确认后，检查ack是否为J+1，ACK是否为1，如果正确则将标志位ACK置为1，ack=K+1，并将该数据包发送给Server，Server检查ack是否为K+1，ACK是否为1，如果正确则连接建立成功，Client和Server进入ESTABLISHED状态，完成三次握手，随后Client与Server之间可以开始传输数据了。

  - TCP四次挥手

    ![](https://i.loli.net/2019/12/13/LbgsDy21qahej8z.jpg)

    ​	所谓四次挥手（Four-Way Wavehand）即终止TCP连接，就是指断开一个TCP连接时，需要客户端和服务端总共发送4个包以确认连接的断开。在socket编程中，这一过程由客户端或服务端任一方执行close来触发。

    ​	第一次挥手：Client发送一个FIN，用来关闭Client到Server的数据传送，Client进入FIN_WAIT_1状态。

    ​	第二次挥手：Server收到FIN后，发送一个ACK给Client，确认序号为收到序号+1（与SYN相同，一个FIN占用一个序号），Server进入CLOSE_WAIT状态。

    ​	第三次挥手：Server发送一个FIN，用来关闭Server到Client的数据传送，Server进入LAST_ACK状态。

    ​	第四次挥手：Client收到FIN后，Client进入TIME_WAIT状态，接着发送一个ACK给Server，确认序号为收到序号+1，Server进入CLOSED状态，完成四次挥手。

### UDP协议

  -  UDP协议

  > 用户数据报协议UDP使Internet传输层协议。提供无连接、不可靠、数据包尽力传输服务。

  - UDP报文头

    ![](https://i.loli.net/2019/12/13/BkqgtFZCj79nuoM.jpg)

### UDP与TCP协议对比

|      TCP       |    UDP     |
| :------------: | :--------: |
|    面向连接    |   无连接   |
| 提供可靠性保证 |   不可靠   |
|       慢       |     快     |
|   资源占用躲   | 资源占用少 |

### Socket编程与传输层协议

> Socket（套接字）是Internet中应用最广泛的网络应用编程接口。

- 实现接口（操作系统实现）
  - 数据包类型套接字SOCK_DGRAM（面向UDP接口）
  - 流式套接字SOCK_STREAM（面向UDP接口）
  - 原始套接字SOCK_RAW（喵娘网络层协议接口IP、ICMP等）

- 主要socketAPI调用过程

  - 创建套接字
  - 端点绑定
  - 发送数据
  - 接收数据
  - 释放套接字

### 总结

  - 网络层的分层及各层原理
  - TCP协议与可靠传输原理简介
  - UDP协议报文头以及UDP与TCP协议对比
  - 建立在UDP/TCP协议上的SOCKET编程步骤