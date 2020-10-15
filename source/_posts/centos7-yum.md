
---
title: 解决新装centos7没有wget命令,yum不能使用和yum源配置
date: 2020-03-18 15:49:57
categories:
  - centos7
tags:
  - centos7
  - yum
  - wget
---

解决新装centos7没有wget命令,yum不能使用和yum源配置
<!--more-->

## 解决新装centos7没有wget命令,yum不能使用

1.注意，没有vim命令，vi可用
```
//删除原来的源或者备份
$ rm -rf /etc/yum/.repos.d/*
```

下载淘宝yum镜像文件，地址：
> http://mirrors.aliyun.com/repo/Centos-7.repo

更改源
```
$ vi /etc/yum.repos.d/centos-7.repo
```

拷贝下载文件中的内容至centos-7.repo文件，按:wq保存退出
```
//清除缓存
$ yum clean all

//yum缓存
$ yum makecache
```

2.从外部下载rpm包
http://www.cnblogs.com/simonbaker/p/5131019.html

## yum源配置（阿里云源）
1) 安装wget

```
yum install -y wget
```

2) 备份/etc/yum.repos.d/CentOS-Base.repo文件

```
cd /etc/yum.repos.d/
mv CentOS-Base.repo CentOS-Base.repo.back
```

3) 下载阿里云的Centos-6.repo文件

> wget -O CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-6.repo

4) 重新加载yum

```
yum clean all
yum makecache
```
