---
title: centos7安装docker和docker-compose
date: 2020-03-07 15:49:57
categories:
  - centos7
  - docker
tags:
  - centos7
  - docker
  - docker-compose
---

centos7安装docker和docker-compose
<!--more-->

## 安装 docker

####安装之前，先清除之前安装的旧版本 docker，如果有的话。
```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

####使用 repository 安装 docker ce
安装基础依赖包
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

不建议 官方给出的源，国内比较慢，可以用阿里源替代
```
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

建议 阿里源
```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

如果想安装指定版本的docker-ce，可以先用命令查看版本号
```
sudo yum list docker-ce --showduplicates | sort -r
```
直接运行会默认安装最新版
```
sudo yum install docker-ce  
```
安装指定版本，例如：yum install docker-ce-18.06.3.ce
```
sudo yum install docker-ce-<VERSION STRING>
```

## 安装 docker-compose
安装的是稳定版本1.23.2的 docker-compose，可以到github上找最新版 https://github.com/docker/compose/releases
```
curl -L https://github.com/docker/compose/releases/download/1.23.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```
gihub上下载docker-compose太慢了，下载不动，只能换成国内镜像（daocloud.io）来下载。

```
curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

## 卸载 docker-ce

```
sudo yum remove docker-ce
sudo rm -rf /var/lib/docker
```

## 卸载 docker-compse

```
sudo rm /usr/local/bin/docker-compose
```

## 启动docker
```
sudo systemctl start docker
```

## 重启docker
```
sudo systemctl restart docker
```

## 设置开机自启
```
systemctl enable docker
```

## 清理所有为 none 的镜像

```
docker rmi $(docker images -q -f dangling=true)
```
