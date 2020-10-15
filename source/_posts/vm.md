---
title: VMware 虚拟机设置网络连接 
date: 2020-03-07 14:11:57
categories:
  - VMware
tags:
  - VMware
---

VMware 虚拟机设置网络连接
<!--more-->

## VMware 虚拟机设置网络连接

1. 打开虚拟机上右键设置 ——> 网络适配器，选择NAT模式并选择启动时连接

2. 编辑 ——> 虚拟网络编辑器
	2.1 选择右下角更改设置，使用管理员进行修改
	2.2 选择VMnet0，选择对应的桥接模式（请注意自己选择的是无线网络还是本地连接）
	
3. 进入系统 通过 vim /etc/sysconfig/network-scripts/ifcfg-ens33 命令进入ifcfg-ens33文件设置ipv4的设置信息
	3.1 注意 IPADDR 的范围需要在虚拟网络编辑器中DNCP分配的ip地址范围之内
	3.2 注意 GATEWAY 需要跟虚拟网络编辑器中的网管地址保持一致
```
ONBOOT=yes
IPADDR=192.168.64.128
NETMASK=255.255.255.0
GATEWAY=192.168.64.2
DNS1=8.8.8.8
DNS2=114.114.114.114
```

4. 设置 vim /etc/resolv.conf 在配置文件中添加
```
nameserver 8.8.8.8
nameserver 114.114.114.114
```

5. 修改完成之后 通过 service network restart 重启网络连接

6. 此时再进行ping的时候就可以成功了

## 如何使VMware ip与本机ip处于同一网段

1. 将vmware的网络适配器改成“桥接模式” 

2. vi /etc/sysconfig/network-scripts/ifcfg-ens33
```
NAME=eth0 文件名 
DEVICE=eth0 设别名 
IPADDR=192.168.1.188 想要设定的ip地址 
NETMASK=255.255.255.0 子网掩码 
NETWORK=192.168.1.0 所属网络，和ip相同网络， 一般最后一位为0 
GATEWAY=192.168.1.1 网关 
BROADCAST=192.168.1.255 广播地址， 和ip相同网络， 一般最后一位为255 
ONBOOT=yes 是否在启动时激活， yes, or no 
USERCTL=no 非root用户是否可以控制该设备，yes or no 
BOOTPROTO=static 网络分配方式， 静态 
HWADDR=00:0C:29:49:40:79 MAC地址，在查看虚拟机适配器属性中有看到
```

