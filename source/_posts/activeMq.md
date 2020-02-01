---
title: activeMq
date: 2020-01-10 13:28:31
tags:
- mq
categories:
- java
- 消息中间件
---

# ActiveMq

## 入门

### 下载

[官网](https://activemq.apache.org/)

环境：CentOS7、Java8

```shell
#进入/tmp
cd tmp
#使用wget下载
wget http://mirror.bit.edu.cn/apache//activemq/5.15.11/apache-activemq-5.15.11-bin.tar.gz
#解压
tar -zxvf apache-activemq-5.15.11-bin.tar.gz -C /var
#修改文件夹名字
mv /var/apache-activemq-5.15.11/ /var/activemq/
```

### 启动

```shell
#进入安装文件夹
cd /var/activemq
#启动activemq
./bin/activemq start

#若启动成功则显示如下
[root@localhost activemq]# ./bin/activemq start
INFO: Loading '/var/activemq//bin/env'
INFO: Using java '/bin/java'
INFO: Starting - inspect logfiles specified in logging.properties and log4j.properties to get details
INFO: pidfile created : '/var/activemq//data/activemq.pid' (pid '19541')
```

### ActiveMQ服务

创建ActiveMQ

```shell
vi /usr/lib/systemd/system/activemq.service
```

填入以下内容

```java
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

修改`/var/activemq/bin/env`配置

```shell
#进入配置
vim /var/activemq/bin/env

#在最后几行会有类似下面的文档：


# Configure a user with non root privileges, if no user is specified do not change user
# (the entire activemq installation should be owned by this user)
ACTIVEMQ_USER=""

# location of the pidfile
# ACTIVEMQ_PIDFILE="$ACTIVEMQ_DATA/activemq.pid"

# Location of the java installation
# Specify the location of your java installation using JAVA_HOME, or specify the
# path to the "java" binary using JAVACMD
# (set JAVACMD to "auto" for automatic detection)
#JAVA_HOME=""
JAVACMD="auto"
```

将`JAVA_HOME=""`取消注释并填入Java地址