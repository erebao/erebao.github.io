---
title: Spring Cloud创建服务提供者
date: 2020-03-01 12:42:57
categories:
  - Spring Cloud
tags:
  - Spring Cloud
  - 微服务
  - Spring Cloud服务提供者
---

当Client向Server注册时，它会提供一些元数据，例如主机和端口，URL，主页等。Eureka Server从每个Client实例接收心跳消息。如果心跳超时，则通常将该实例从注册Server中删除
<!--more-->

## 创建服务提供者

创建一个工程名为**hello-spring-cloud-service-admin**的项目，**pom.xml**配置文件如下：
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

    <artifactId>hello-spring-cloud-service-admin</artifactId>
    <packaging>jar</packaging>

    <name>hello-spring-cloud-service-admin</name>
    <url>http://www.erebao.com</url>
    <inceptionYear>2020-Now</inceptionYear>

    <dependencies>
        <!-- Spring Boot Begin -->
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
        <!-- Spring Cloud End -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.funtl.hello.spring.cloud.service.admin.ServiceAdminApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

#### Application

增加注解：
> @EnableEurekaClient

#### application.yml

```yaml
spring:
  application:
    name: hello-spring-cloud-service-admin

server:
    port: 8763

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```
