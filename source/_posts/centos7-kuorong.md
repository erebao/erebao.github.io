---
title: centos7分区扩容
date: 2020-03-13 14:12:57
categories:
  - centos7
tags:
  - centos7
  - 分区
---

centos7分区扩容
<!--more-->

## 分区： 
>fdisk /dev/sda

```
p		查看已分区数量（我看到有两个 /dev/sda1 /dev/sda2）
n		新增加一个分区
p		分区类型我们选择为主分区 
		分区号输入3（因为1,2已经用过了,sda1是分区1,sda2是分区2,sda3分区3） 
		默认（起始扇区） 
		默认（结束扇区） 
t		修改分区类型 
		选分区3 
8e		修改为LVM（8e就是LVM）
w		写分区表 
q		完成，退出fdisk命令
```

## 重启虚拟机
> reboot

## 格式化分区3命令:
> mkfs.ext3 /dev/sda3

## 添加新LVM到已有的LVM组，实现扩容

```
lvm			进入lvm管理
lvm>pvcreate /dev/sda3		这是初始化刚才的分区3
lvm>vgextend centos /dev/sda3		将初始化过的分区加入到虚拟卷组centos (卷和卷组的命令可以通过 vgdisplay )
lvm>vgdisplay -v		或者vgdisplay查看free PE /Site
lvm>lvextend -l+6143 /dev/mapper/centos-root		扩展已有卷的容量（6143 是通过vgdisplay查看free PE /Site的大小）
lvm>pvdisplay		查看卷容量，这时你会看到一个很大的卷了
lvm>quit		退出
```

## 上面只是卷扩容了，下面是文件系统的真正扩容，输入以下命令：

CentOS7下面由于使用的是XFS命令:

> sudo xfs_growfs /dev/mapper/centos-root

是df -h查看到根目录的挂载点

到这里就算完成了,df -h 一下就能看到我们新增的sda3

