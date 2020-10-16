---
title: docker-compose.yml方式安装zookeeper
date: 2020-02-29 14:12:57
categories:
  - docker
tags:
  - docker
  - docker-compose
  - zookeeper
---

ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。
<!--more-->

## zookeeper 安装

使用docker-compose.yml来安装和运行zookeeper
```yaml
version: '3.1'
services:
    zoo1:
        image: zookeeper:3.4.14
        restart: always
        hostname: zoo1
        ports:
            - 2181:2181
        environment:
            ZOO_MY_ID: 1
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888

    zoo2:
        image: zookeeper:3.4.14
        restart: always
        hostname: zoo2
        ports:
            - 2182:2181
        environment:
            ZOO_MY_ID: 2
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888

    zoo3:
        image: zookeeper:3.4.14
        restart: always
        hostname: zoo3
        ports:
            - 2183:2181
        environment:
            ZOO_MY_ID: 3
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
```

#### 进入容器
>> docker exec -it [NAME] /bin/bash

#### 进入bin/目录下执行zkServer.sh查看zookeeper状态
>> bin/
>> ./zkServer.sh status

#### 配置文件路径
>> /conf/zoo.cfg

#### 常用命令
启动服务
> ./zkServer.sh start

停止服务
> ./zkServer.sh stop

重启服务
> ./zkServer.sh restart

执行状态
> ./zkServer.sh status

## 客户端

连接服务端
> ./zkCli.sh -server localhost:2181

创建节点
> create /test "hello Zookeeper"

获得节点
> get /test

删除节点
> delete /test
