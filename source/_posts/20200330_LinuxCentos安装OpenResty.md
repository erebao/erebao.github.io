---
title: Linux Centos 安装OpenResty
date: 2020-03-30 15:49:57
categories:
  - centos7
tags:
  - centos7
  - OpenResty
  - nginx
  - 运维
---

解决新装centos7没有wget命令,yum不能使用和yum源配置
<!--more-->

# 源码包编译安装

## 下载

从下载页Download下载最新的 OpenResty®源码包，并且像下面的示例一样将其解压:
```
wget https://openresty.org/download/openresty-1.13.6.2.tar.gz
```

## 安装前的准备

必须将这些库 perl 5.6.1+, libpcre, libssl安装在您的电脑之中。 对于 Linux来说, 您需要确认使用 ldconfig 命令，让其在您的系统环境路径中能找到它们。

推荐您使用yum安装以下的开发库:

```
yum install pcre-devel openssl-devel gcc curl -y
```

## 安装

```
tar -zxvf openresty-1.13.6.2.tar.gz
cd openresty-1.13.6.2
./configure
make
make install
```

## 测试性能

安装压力测试工具ab

```
yum -y install httpd-tools
```

## 压力测试

-c:每次并发数为10个
-n:共发送50000个请求
```
ab -c10 -n50000 http://localhost:8080/
```
测试报详解
```
Server Software: Apache #服务器软件
Server Hostname:        localhost      #域名
Server Port:            80              #请求端口号
Document Path:          /              #文件路径
Document Length:        40888 bytes    #页面字节数
Concurrency Level:      10              #请求的并发数
Time taken for tests:  27.300 seconds  #总访问时间
Complete requests:      1000            #请求成功数量
Failed requests:        0              #请求失败数量
Write errors:          0Total transferred:      41054242 bytes  #请求总数据大小（包括header头信息）
HTML transferred:      40888000 bytes  #html页面实际总字节数
Requests per second:    36.63 [#/sec] (mean)  #每秒多少请求，这个是非常重要的参数数值，服务器的吞吐量
Time per request:      272.998 [ms] (mean)    #用户平均请求等待时间
Time per request:      27.300 [ms] (mean, across all concurrent requests)
                                                # 服务器平均处理时间，也就是服务器吞吐量的倒数
Transfer rate:          1468.58 [Kbytes/sec] received  #每秒获取的数据长度
Connection Times (ms)
              min  mean[+/-sd] median  maxConnect:      43  47  2.4    47      53Processing:  189  224  40.7    215    895Waiting:      102  128  38.6    118    794Total:        233  270  41.3    263    945Percentage of the requests served within a certain time (ms)
  50%    263    #50%用户请求在263ms内返回
  66%    271    #66%用户请求在271ms内返回
  75%    279    #75%用户请求在279ms内返回
  80%    285    #80%用户请求在285ms内返回
  90%    303    #90%用户请求在303ms内返回
  95%    320    #95%用户请求在320ms内返回
  98%    341    #98%用户请求在341ms内返回
  99%    373    #99%用户请求在373ms内返回
100%    945 (longest request)
```

## 添加nginx到服务加入开机启动

vi /lib/systemd/system/nginx.service
```
[Unit]
Description=nginx
After=network.target
[Service]
Type=forking
ExecStart=/usr/local/openresty/nginx/sbin/nginx
ExecReload=/usr/local/openresty/nginx/sbin/nginx -s reload
ExecStop=/usr/local/openresty/nginx/sbin/nginx -s quit
PrivateTmp=true
[Install]
WantedBy=multi-user.target
```

## 启动并加入开机启动

```
systemctl start nginx.service
systemctl enable nginx.service
```

## 将nginx加入到环境变量

```
vi /etc/profile
# nginx
export NGINX_HOME=/usr/local/openresty/nginx
export PATH=$PATH:$NGINX_HOME/sbin
```

## 刷新:

```
source /etc/profile
```

## 启动nginx
> cd /usr/local/openresty/nginx/sbin/nginx
> ./nginx

## nginx查看已安装模块
> ./nginx -V

## ERROR

make时候如果遇到报错

> src/os/unix/ngx_user.c:26:7: error: ‘struct crypt_data’ has no member named ‘current_salt’

则修改文件
> ./package/openresty-1.13.6.2/build/nginx-1.13.6/src/os/unix/ngx_user.c
> ./package/openresty-1.13.6.2/bundle/nginx-1.13.6/src/os/unix/ngx_user.c

注销掉
> /* cd.current_salt[0] = ~salt[0] */
