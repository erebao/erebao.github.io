---
title: 在Centos7中安装hexo基础环境
date: 2020-10-16 10:52:57
categories:
  - hexo
tags:
  - hexo
  - centos7
  - node
  - git
---

#### 第一节：Git的安装
** 第一节：Git的安装 **
```
yum -y update
```

** 2、执行快速安装命令 **
```
yum install git -y
```

** 3、设置基础账号与邮箱 **
```
git config --global user.name "你的账号"
git config --global user.email "你的邮箱"
```

#### 第二节：nodejs的安装
** 1、安装基础编译环境 **
```
yum install gcc-c++
```

** 2、下载源码包 **
```
wget https://npm.taobao.org/mirrors/node/v11.0.0/node-v11.0.0.tar.gz
```

** 3、解压源码包，并进入解压后的文件夹中 **
```
tar -zxf node-v11.0.0.tar.gz
cd node-v11.0.0
```

** 4、开始进行编译，本次编译时间大约30分钟左右（可以去忙些别的了。。。） **
```
./configure && make
```

** 5、开始进行安装 **
```
make install
node -v
npm -v
```
备注：如果node -v没有版本信息，可进行一下设置环境变量再进行测试
```
vim /etc/profile
在该文件底部增加下面两行
export NODE_HOME=node目录
export PATH=$NODE_HOME/bin:$PATH
```
执行命令生效环境变量后再次重新尝试获取版本信息。
```
source /etc/profile
node -v
npm -v
```

#### 第三节：hexo的安装
```
npm install -g hexo-cli
hexo -v
```

注：初始化后生成了一系列文件详情请参考https://hexo.io/zh-cn/docs/setup

注：命令参考https://hexo.io/zh-cn/docs/commands
