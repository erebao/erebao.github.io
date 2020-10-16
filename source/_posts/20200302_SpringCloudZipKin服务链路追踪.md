---
title: Spring Cloud ZipKin服务链路追踪
date: 2020-03-02 14:11:57
categories:
  - Spring Cloud
tags:
  - Spring Cloud
  - zipkin
  - 微服务
---

ZipKin是一个开放源代码的分布式跟踪系统，由Twitter公司开源，它致力于收集服务的定时数据，以解决微服务架构中的延迟问题，包括数据的收集、存储、查找和展现。
<!--more-->

## 创建服务消费者

创建一个工程名为**hello-spring-cloud-zipkin**的项目，**pom.xml**配置文件如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.funtl</groupId>
        <artifactId>hello-spring-cloud-dependencies</artifactId>
        <version>1.0.0-SNAPSHOT</version>
        <relativePath>../hello-spring-cloud-dependencies/pom.xml</relativePath>
    </parent>

    <artifactId>hello-spring-cloud-zipkin</artifactId>
    <packaging>jar</packaging>

    <name>hello-spring-cloud-zipkin</name>
    <url>http://www.erebao.com</url>
    <inceptionYear>2020-Now</inceptionYear>

    <dependencies>
        <!-- Spring Boot Begin -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- Spring Boot End -->

        <!-- Spring Cloud Begin -->
		<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin</artifactId>
        </dependency>
        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-server</artifactId>
        </dependency>
        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-autoconfigure-ui</artifactId>
        </dependency>
        <!-- Spring Cloud End -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.funtl.hello.spring.cloud.zipkin.ZipKinApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```
> 注意版本号为 **2.10.1** 这里没有写版本号是因为已经托管到 **dependencies** 项目中

#### Application

增加注解：
> @EnableZipkinServer

#### application.yml

```yaml
spring:
  application:
    name: hello-spring-cloud-zipkin
  boot:
    admin:
      client:
        url: http://localhost:8084

server:
  port: 9411

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/

management:
  metrics:
    web:
      server:
        auto-time-requests: false
```

#### 追踪服务

在所有需要被追踪的项目（就当前而言，除了 **dependencies** 项目外都需要被追踪）中增加 **spring-cloud-starter-zipkin** 依赖
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```
**application.yml** 中增加：
```yaml
spring:
  zipkin:
    base-url: http://localhost:9411
```

#### 测试

浏览器输入：http://localhost:9411/zipkin

#### 术语解释
- Span：基本工作单元，例如，在一个新建的Span中发送一个RPC等同于发送一个回应请求给RPC，Span通过一个64位ID唯一标识，Trace以另一个64位ID表示。
- Trace：一系列Spans组成的一个树状结构，例如，如果你在运行一个分布式大数据工程，你可能需要一个Trace
- Annotation：用来即使记录一个事件的存在，一些核心Annotation用来定义一个请求的开始和结束
	- cs：Client Sent，客户端发起一个请求，这个Annotation描述了这个Span的开始
	- sr：Server Received，服务端获得请求并准备开始处理它，**如果将其sr减去cs时间戳便可得到网络延迟**
	- ss：Server Sent，表明请求处理的完成（当请求返回客户端），**如果ss减去sr时间戳便可得到服务端需要的处理请求时间**
	- cr：Client Received表明Span的结束，客户端成功接收到服务端的回复，**如果cr减去cs时间戳便可得到客户端从服务端获取回复的所需时间**
