---
title: docker-compose.yml方式安装Fast DFS分布式文件系统
date: 2020-03-02 17:11:57
categories:
  - docker-compose
tags:
  - Fast DFS
  - 分布式文件系统
---

FastDFS是一个开源的轻量级分布式文件系统，它对文件进行管理，功能包括：文件存储、文件同步、文件访问（文件上传、文件下载）等，解决了大容量存储和负载均衡的问题。特别适合以文件为载体的在线服务，如相册网站、视频网站等等。
FastDFS为互联网量身定制，充分考虑了冗余备份、负载均衡、线性扩容等机制，并注重高可用、高性能等指标，使用FastDFS很容易搭建一套高性能的文件服务器集群提供文件上传、下载等服务。
<!--more-->

## Fast DFS安装

使用docker-compose.yml来安装和运行Fast DFS

创建 **fastdfs** 目录， 在 fastdfs/ 目录下创建 **docker-compose.yml**：

```yaml
version: '3.1'
services:
  fastdfs:
    build: fastdfs
    restart: always
    container_name: fastdfs
    volumes:
      - /usr/local/docker/fastdfs/storage:/fastdfs/storage
    network_mode: host
```

在 fastdfs 目录下在创建一个 fastdfs 目录，在 **fastdfa/fastdfs/** 目录下创建 **Dockerfile**：
```
FROM ubuntu:xenial
MAINTAINER topsale@vip.qq.com

# 更新数据源
WORKDIR /etc/apt
RUN echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse' > sources.list
RUN echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse' >> sources.list
RUN echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse' >> sources.list
RUN echo 'deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse' >> sources.list
RUN apt-get update

# 安装依赖
RUN apt-get install make gcc libpcre3-dev zlib1g-dev --assume-yes

# 复制工具包
ADD fastdfs-5.11.tar.gz /usr/local/src
ADD fastdfs-nginx-module_v1.16.tar.gz /usr/local/src
ADD libfastcommon.tar.gz /usr/local/src
ADD nginx-1.13.6.tar.gz /usr/local/src

# 安装 libfastcommon
WORKDIR /usr/local/src/libfastcommon
RUN ./make.sh && ./make.sh install

# 安装 FastDFS
WORKDIR /usr/local/src/fastdfs-5.11
RUN ./make.sh && ./make.sh install

# 配置 FastDFS 跟踪器
ADD tracker.conf /etc/fdfs
RUN mkdir -p /fastdfs/tracker

# 配置 FastDFS 存储
ADD storage.conf /etc/fdfs
RUN mkdir -p /fastdfs/storage

# 配置 FastDFS 客户端
ADD client.conf /etc/fdfs

# 配置 fastdfs-nginx-module
ADD config /usr/local/src/fastdfs-nginx-module/src

# FastDFS 与 Nginx 集成
WORKDIR /usr/local/src/nginx-1.13.6
RUN ./configure --add-module=/usr/local/src/fastdfs-nginx-module/src
RUN make && make install
ADD mod_fastdfs.conf /etc/fdfs

WORKDIR /usr/local/src/fastdfs-5.11/conf
RUN cp http.conf mime.types /etc/fdfs/

# 配置 Nginx
ADD nginx.conf /usr/local/nginx/conf

COPY entrypoint.sh /usr/local/bin/
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]

WORKDIR /
EXPOSE 8888
CMD ["/bin/bash"]
```
entrypoint.sh：
```
#!/bin/sh
/etc/init.d/fdfs_trackerd start
/etc/init.d/fdfs_storaged start
/usr/local/nginx/sbin/nginx -g 'daemon off;'
```
> 注：Shell 创建后是无法直接使用的，需要赋予执行的权限，使用 chmod +x entrypoint.sh 命令

将下载下来的**fastdfs相关软件包**复制到 **fastdfs/fastdfs/** 目录下

- fastdfs相关软件包
	- client.conf
	- config
	- entrypoint.sh
	- fastdfs-5.11.tar.gz
	- fastdfs-nginx-module_v1.16.tar.gz
	- libfastcommon.tar.gz
	- mod_fastdfs.conf
	- nginx.conf
	- nginx-1.13.6.tar.gz
	- storage.conf
	- tracker.conf

配置**client.conf**修改**tracker_server**为本机ip
配置**mod_fastdfs.conf**修改**tracker_server**为本机ip
配置**storage.conf**修改**tracker_server**为本机ip

> 启动 docker-compose up -d

## 测试

进入容器 **docker exec -it fastdfs /bin/bash**

测试文件上传 **/usr/bin/fdfs_upload_file /etc/fdfs/client.conf /usr/local/src/fastdfs-5.11/INSTALL**
> 服务器反馈地址 group1/M00/00/00/wKhLi1oHVM2verahfjin2k455k3

测试Nginx访问（此处配置的nginx.conf端口号为8888）
> http://192.168.75.145:8888/group1/M00/00/00/wKhLi1oHVM2verahfjin2k455k3

