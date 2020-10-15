---
title: docker-compose.yml方式安装Registry(镜像管理平台)
date: 2020-02-29 13:12:57
categories:
  - docker
tags:
  - docker
  - docker-compose
  - registry
---

Docker Registry可以用来存储和管理自己的镜像
<!--more-->

## Registry 安装

使用docker-compose.yml来安装和运行Registry
```yaml
version: '3.1'
services:
  registry:
    image: registry
    restart: always
    container_name: registry
    ports:
      - 5000:5000
    volumes:
      - /usr/local/docker/registry/data:/var/lib/registry
```

测试访问：
http://ip:5000/v2/

## 客户端配置

在/etc/docker/daemon.json中增加以下内容（如果文件不存在则新建该文件）
```
{
  "registry-mirrors": [
    "https://o4omo0yw.mirror.aliyuncs.com"
  ],
  "insecure-registries": [
    "127.0.0.1.5000"
  ]
}
```

重启Docker
>sudo systemctl daemon-reload

>sudo systemctl restart docker

检查daemon是否生效
>docker info

#### 客户端测试镜像上传

拉取一个镜像
> docker pull nginx

标记本地镜像并指向目标仓库(ip:port/image_name:tag，该格式为标记版本号)
> docker tag nginx 127.0.0.1:5000/nginx

提交镜像到仓库
> docker push 127.0.0.1:5000/nginx

查看全部镜像
> http://127.0.0.1:5000/v2/_catalog

查看指定镜像
> http://127.0.0.1:5000/v2/tomcat/tags/list

## Registry WebUI 安装

私服安装成功后就可以使用docker命令行工具队registry做各种操作了，然而不太方便的是不能直观的查看registry中的资源情况，这里介绍两个Docker Registry WebUI工具
- docker-registry-frontend
- docker-registry-web
两个工具差不多，我们以docker-registry-frontend讲解

#### docker-registry-frontend 安装

```yaml
version: '3.1'
services:
  frontend:
    image: konradkleine/docker-registry-frontend:v2
    ports:
      - 8080:80
    volumes:
      - ./certs/frontend.crt:/etc/apache2/server.crt:ro
      - ./certs/frontend.key:/etc/apache2/server.key:ro
    environment:
      - ENV_DOCKER_REGISTRY_HOST=192.168.75.131
      - ENV_DOCKER_REGISTRY_PORT=5000
```

运行成功后浏览器访问：
> http://192.168.75.131:8080
