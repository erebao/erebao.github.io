---
title: docker-compose.yml方式安装gitlab runner
date: 2020-03-08 14:12:57
categories:
  - docker
tags:
  - docker
  - docker-compose
  - gitlab runner
---

持续集成
<!--more-->

## 环境准备

创建工作目录 /usr/lcoal/docker/runner
创建构建目录 /usr/local/docker/runner/environment
下载 jdk-8u152-linux-x64.tar.gz 并复制到 /usr/local/docker/runner/environment

## 在/usr/lcoal/docker/runner目录下创建 docker-compose.yml

```yaml
version: '3.1'
services:
	gitlab-runner:
		build: environment
		container_name: gitlab-runner
		privileged: true
		volumes:
			- /usr/local/docker/runner/config:/etc/gitlab-runner
			- /var/run/docker.sock:/var/run/docker.sock
```

## Dockerfile

在 /usr/local/docker/runner/environment 目录下创建 Dockerfile

```
FROM gitlab/gitlab-runner:v11.0.2

# 修改软件源
RUN echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse' > /etc/apt/sources.list && \
    echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse' >> /etc/apt/sources.list && \
    echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse' >> /etc/apt/sources.list && \
    echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse' >> /etc/apt/sources.list && \
    apt-get update -y && \
    apt-get clean

# 安装 Docker
RUN apt-get -y install apt-transport-https ca-certificates curl software-properties-common && \
    curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | apt-key add - && \
    add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable" && \
    apt-get update -y && \
    apt-get install -y docker-ce
COPY daemon.json /etc/docker/daemon.json

# 安装 Docker Compose
WORKDIR /usr/local/bin
RUN wget https://raw.githubusercontent.com/topsale/resources/master/docker/docker-compose
RUN chmod +x docker-compose

# 安装 Java
RUN mkdir -p /usr/local/java
WORKDIR /usr/local/java
COPY jdk-8u152-linux-x64.tar.gz /usr/local/java
RUN tar -zxvf jdk-8u152-linux-x64.tar.gz && \
    rm -fr jdk-8u152-linux-x64.tar.gz

# 安装 Maven
RUN mkdir -p /usr/local/maven
WORKDIR /usr/local/maven
RUN wget https://raw.githubusercontent.com/topsale/resources/master/maven/apache-maven-3.5.3-bin.tar.gz
RUN tar -zxvf apache-maven-3.5.3-bin.tar.gz && \
    rm -fr apache-maven-3.5.3-bin.tar.gz

# 配置环境变量
ENV JAVA_HOME /usr/local/java/jdk1.8.0_152
ENV MAVEN_HOME /usr/local/maven/apache-maven-3.5.3
ENV PATH $PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin

# 配置gitlab-runner权限
chgrp docker /var/run/docker.sock
usermod -aG docker gitlab-runner

WORKDIR /
```

## 在enviroment目录下创建

在 /usr/local/docker/runner/environment 目录下创建 daemon.json，用于配置加速器和仓库地址

```
{
  "registry-mirrors": [
    "https://registry.docker-cn.com"
  ],
  "insecure-registries": [
    "ip:port"
  ]
}
```

> 执行 docker-compose up -d

## 注册Runner

```
docker exec -it gitlab-runner gitlab-runner register

# 输入 GitLab 地址
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://192.168.75.146:8080/

# 输入 GitLab Token
Please enter the gitlab-ci token for this runner:
1Lxq_f1NRfCfeNbE5WRh

# 输入 Runner 的说明
Please enter the gitlab-ci description for this runner:
可以为空

# 设置 Tag，可以用于指定在构建规定的 tag 时触发 ci
Please enter the gitlab-ci tags for this runner (comma separated):
可以为空

# 选择 runner 执行器，这里我们选择的是 shell
Please enter the executor: virtualbox, docker+machine, parallels, shell, ssh, docker-ssh+machine, kubernetes, docker, docker-ssh:
shell
```

## 完善项目

项目中用到的 .gitlab-ci.yml、Dockerfile、docker-compose.yml 如下：

.gitlab-ci.yml
```yaml
stages:
  - build
  - push
  - run
  - clean

build:
  stage: build
  script:
    - /usr/local/maven/apache-maven-3.5.3/bin/mvn clean package
    - cp target/reai-drive-config-1.0.0-SNAPSHOT.jar docker
    - cd docker
    - docker build -t reai-drive-config .

push:
  stage: push
  script:
    - docker push reai-drive-config

run:
  stage: run
  script:
    - cd docker
    - docker-compose down
    - docker-compose up -d

clean:
  stage: clean
  script:
    - docker rmi $(docker images -q -f dangling=true)
```

Dockerfile
```
FROM openjdk:8-jre
MAINTAINER Tanwb

ENV APP_VERSION 1.0.0-SNAPSHOT

RUN mkdir /app
COPY reai-drive-config-$APP_VERSION.jar /app/app.jar

ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/app/app.jar", "--spring.profiles.active=prod"]

EXPOSE 8888
```

docker-compose.yml
```yaml
version: '3.1'
services:
  reai-drive-config:
    restart: always
    image: registry.cn-chengdu.aliyuncs.com/tanwb/reai-drive-config
    container_name: reai-drive-config
    ports:
      - 8888:8888
```

## ERROR

要用gitlab-ci加docker来构建项目，运行job时报错：
ERROR: Preparation failed: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
原因是我用root运行docker，而gitlab的runner是用gitlab-runner这个帐户来运行的，遇到了权限问题。

> ls -slh /var/run/docker.sock

```
0 srw-rw---- 1 root root 0 Dec 7 10:16 /var/run/docker.sock
```

用su - gitlab-runner切换帐户后，也会报告相同的错

> docker info

将文件docker.sock分配到docker组

> chgrp docker /var/run/docker.sock

最后把gitlab-runner账户加入docker组就可以了

> usermod -aG docker gitlab-runner

没有docker组则创建一个docker组

> groupadd docker

重启docker进程

> systemctl restart docker

> ls -slh /var/run/docker.sock

```
0 srw-rw---- 1 root docker 0 Dec 10 10:30 /var/run/docker.sock
```

