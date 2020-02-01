---
title: ActiveMQ
date: 2020-01-31 23:26:24
tags:
- 消息中间件
categories:
- java
- 中间件
---

# ActiveMQ

> 完全支持JMS1.1和J2EE1.4规范的JMS Provider实现。（老牌的消息中间件）

## JMS规范

> Java消息服务（Java Message Service，即JMS）应用程序接口的是一个Java平台中关于面向消息中间件的API，用于在2个应用程序之间，或分布式系统中发送消息，进行异步通信。是一个与具体平台无关的API。

### JMS元素

![1580485275826](E:\Blog\source\_drafts\1580485275826.png)

JMS由如下几个模块组成：

- 管理对象（Administered objects）-连接工厂（Connection Factories）和目的地（Destination）

  > 管理对象（Administered objects）是预先配置的JMS对象，由系统管理员为使用JMS的客户端创建，主要有两个被管理的对象：
  >
  > 连接工厂（ConnectionFactory）
  >
  > 目的地（Destination）
  >
  > 这两个管理对象由JMS系统管理员通过使用Application Server管理控制台创建，存储在应用程序服务器的JNDI名字空间或JNDI注册表。

- 连接对象（Connections）

  > 连接对象封装了与JMS提供者之间的虚拟连接，如果我们有一个ConnectionFactory对象，可以使用它来创建一个连接

- 会话（Sessions）

  > Session是一个单线程上下文，用于生产和消费消息，可以创建出消息生产者和消息消费者。
  >
  > Session对象实现了Session接口，在创建完连接后，我们可以使用它创建Session。

- 消息生产者（Message Producers）

  > 消息生产者由Session创建，用于往目的地发送消息。生产者实现MessageProducer接口，我们可以为目的地、队列或话题创建生产者；

- 消息消费者（Message Consumers）

  > 消息消费者由Session创建，用于接受目的地发送的消息。消费者实现MessageConsumer接口，，我们可以为目的地、队列或话题创建消费者

- 消息监听者（Message Listeners）

  > JMS消息监听器是消息的默认事件处理者，他实现了MessageListener接口，该接口包含一个onMessage方法，在该方法中需要定义消息达到后的具体动作。通过调用setMessageListener方法我们给指定消费者定义了消息监听器

### JMS消息模型

- P2P

  > 一个生产者只给一个消费者

  ![1580485840557](E:\Blog\source\_drafts\1580485840557.png)

- 订阅发布

  > 一个生产者把消息发送个所有订阅了的消费者

  ![1580485956192](E:\Blog\source\_drafts\1580485956192.png)

### JMS消息结构

![1580486032558](E:\Blog\source\_drafts\1580486032558.png)

- 消息头

  > 定义了消息的路由和格式

  ![1580486070969](E:\Blog\source\_drafts\1580486070969.png)

- 消息属性

  > 消息的附加消息头，属性名可以自定义

- 消息体

  ![1580486313000](E:\Blog\source\_drafts\1580486313000.png)

## ActiveMQ的特性

- 支持多种编程语言
- 支持多种传输协议
- 有多种持久化方式

## ActiveMQ安装

[**官网教程**](https://activemq.apache.org/getting-started)

> 演示环境： Centos7、jdk8、activemq5.15.8下载地址： http://activemq.apache.org/activemq-5158-release.html
> 解压： ```tar -zxvf apache-activemq-5.15.8-bin.tar.gz -C /var```
> 修改目录名称 ```mv /var/apache-activemq-5.15.8/ /var/activemq/```
> 启动： ```./bin/activemq start```
> 停止：``` ./bin/activemq stop```

### 操作练习

1、创建一个systemd服务文件：``` vi /usr/lib/systemd/system/activemq.service```

2、 放入内容

```
[Unit]
Description=ActiveMQ service
After=network.target

[Service]
Type=forking
ExecStart=/var/activemq/bin/activemq start
ExecStop=/var/activemq/bin/activemq stop
User=root
Group=root
Restart=always
RestartSec=9
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=activemq

[Install]
WantedBy=multi-user.target
```

3、 找到java命令所在的目录 ``` whereis java ```

4、设置activemq配置文件/var/activemq/bin/env中的JAVA_HOME

```
# Location of the java installation
# Specify the location of your java installation using JAVA_HOME, or specify the
# path to the "java" binary using JAVACMD
# (set JAVACMD to "auto" for automatic detection)
JAVA_HOME="/usr/local/java/jdk1.8.0_181"
JAVACMD="auto"
```

5、 通过systemctl管理activemq启停

- 启动activemq服务: ```systemctl start activemq```
- 查看服务状态: ```systemctl status activemq```
- 创建软件链接：``` ln -s /usr/lib/systemd/system/activemq.service /etc/systemd/system/multi-user.target.wants/activemq.service ```
- 开机自启: ```systemctl enable activemq```
- 检测是否开启成功(enable)： ```systemctl list-unit-files |grep activemq ```

6、 防火墙配置，Web管理端口默认为8161（admin/admin），通讯端口默认为61616

- 添加并重启防火墙

```
firewall-cmd --zone=public --add-port=8161/tcp --permanent
firewall-cmd --zone=public --add-port=61616/tcp --permanent
systemctl restart firewalld.service
```

- 或者直接关闭防火墙: ```systemctl stop firewalld.service```

7、 修改web管理系统的部分配置,配置文件在```/var/activemq/conf```

- 端口修改

```
<bean id="jettyPort" class="org.apache.activemq.web.WebConsolePort" init-method="start">
  <!-- the default port number for the web console -->
  <property name="host" value="0.0.0.0"/>
  <!--此处即为管理平台的端口-->
  <property name="port" value="8161"/>
</bean>
```

- 关闭登录

```
<bean id="securityConstraint" class="org.eclipse.jetty.util.security.Constraint">
  <property name="name" value="BASIC" />
  <property name="roles" value="user,admin" />
  <!-- 改为false即可关闭登陆 -->
  <property name="authenticate" value="true" />
</bean>
```

- 其他配置: ```/var/activemq/conf/jetty-realm.properties```

```
## ---------------------------------------------------------------------------
# 在此即可维护账号密码，格式：
# 用户名:密码,角色
# Defines users that can access the web (console, demo, etc.)
# username: password [,rolename ...]
admin: admin, admin
user: 123, user
```

> 环境：win10 、Java1.8

1、下载

到官网下载最新版本，有windows版本和linux版本的。

http://activemq.apache.org/download.html

windows版本：apache-activemq-5.10-20140603.133406-78-bin.zip

2、解压和配置环境

将activemq安装包解压到你喜欢的文件夹下，然后将其bin目录添加到path环境中（也可以不加不过每次都需要进入安装文件的bin目录下执行命令，麻烦干脆直接加）

3、启动

win+r

cmd

`activemq start`启动服务

启动成功界面：

![1580487781288](E:\Blog\source\_drafts\1580487781288.png)

 在浏览器中输入：`服务器地址：8161（这里由于是本地安装，所以为：http://127.0.0.0:8161）`

![1580487903466](E:\Blog\source\_drafts\1580487903466.png)

点击Manage ActiveMQ broker输入账号密码管理（默认账号：admin，默认密码：amdin）。

![1580487890021](E:\Blog\source\_drafts\1580487890021.png)



JAVA客户端的使用

- 标准客户端使用

```
<dependency>
  <groupId>org.apache.activemq</groupId>
  <artifactId>activemq-all</artifactId>
  <version>5.15.8</version>
</dependency>
```

- Spring中使用: ```http://spring.io/guides/gs/messaging-jms/```

```
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jms</artifactId>
    <version>5.1.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-broker</artifactId>
    <version>5.15.8</version>
    <exclusions>
    <exclusion>
        <artifactId>geronimo-jms_1.1_spec</artifactId>
        <groupId>org.apache.geronimo.specs</groupId>
    </exclusion>
    </exclusions>
</dependency>
```

## ActiveMQ使用

```java
package com.study.mq.a1_example.discovery;
import org.apache.activemq.ActiveMQConnectionFactory;
import javax.jms.*;
// 组播的形式自动发现服务器: http://activemq.apache.org/multicast-transport-reference.html
// 自己电脑上启动一个activemq，在activemq.xml connector加上
// <transportConnector name="openwire" uri="tcp://0.0.0.0:61616?trace=true&amp;maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600" discoveryUri="multicast://default"/>
//上述uri在conf文件夹下而且原来就已经存在这个bean，所以只需要在最后面添加discoveryUri="multicast://default"即可
// 玩一玩就行，跨网络啥的，要配置网络.客户端不用这个，一般是服务器集群用得到
public class ConsumerAndProducerMulticastDiscovery {
    public static void main(String[] args) {
        ActiveMQConnectionFactory connectionFactory = null;
        Connection conn = null;
        Session session = null;
        MessageConsumer consumer = null;
        try {
            // 1、创建连接工厂（不需要手动指定，自动发现）
            connectionFactory = new ActiveMQConnectionFactory("discovery:(multicast://default)");
            // 2、创建连接对象
            conn = connectionFactory.createConnection();
            conn.start();
            // 3、 创建session
            session = conn.createSession(false, Session.AUTO_ACKNOWLEDGE);
            // 4、创建点对点接收的目标
            Destination destination = session.createQueue("queue1");
            // 5、创建生产者消息
            MessageProducer producer = session.createProducer(destination);
            // 设置生产者的模式，有两种可选
            producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
            // 6、创建一条消息
            String text = "Hello world!";
            TextMessage message = session.createTextMessage(text);
            // 7、发送消息
            producer.send(message);
            // 8、创建消费者消息
            consumer = session.createConsumer(destination);
            // 9、接收消息
            Message consumerMessage = consumer.receive();
            if (consumerMessage instanceof TextMessage) {
                System.out.println("收到文本消息：" + ((TextMessage) consumerMessage).getText());
            } else {
                System.out.println(consumerMessage);
            }
            consumer.close();
            session.close();
            conn.close();
        } catch (JMSException e) {
            e.printStackTrace();
        }
    }
}
```

代码的主要过程：

![1580485275826](E:\Blog\source\_drafts\1580485275826.png)

代码结果：

![1580579889075](E:\Blog\source\_drafts\1580579889075.png)

这样就基本入门了activemq。



## 参考文献

- https://howtodoinjava.com/jms/jms-java-message-service-tutorial/
- https://docs.oracle.com/javaee/6/tutorial/doc/bnceh.html
- 网易高级Java开发工程师课程