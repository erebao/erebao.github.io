---
title: Spring Cloud Config分布式配置中心
date: 2020-03-02 13:11:57
categories:
  - Spring Cloud
tags:
  - Spring Cloud
  - Spring Cloud Config
  - 微服务
---

在分布式系统中，由于服务数量巨多，为了方便服务配置文件统一管理，实时更新，所以需要分布式配置中心组件。在Spring Cloud中，有分布式配置中心组件Spring Cloud Config，它支持配置服务放在配置服务的内存中（即本地），也支持放在远程git仓库中。在Spring Cloud Config组件中，分两个角色，一是Config Server，二是Config Client
<!--more-->

## 创建服务消费者

创建一个工程名为**hello-spring-cloud-config**的项目，**pom.xml**配置文件如下：
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

    <artifactId>hello-spring-cloud-config</artifactId>
    <packaging>jar</packaging>

    <name>hello-spring-cloud-config</name>
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
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
        <!-- Spring Cloud End -->
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.funtl.hello.spring.cloud.config.ConfigApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

#### Application

增加注解：
> @EnableConfigServer

#### application.yml

```yaml
spring:
  application:
    name: hello-spring-cloud-config
  cloud:
    config:
      label: master
      server:
        git:
          uri: https://github.com/erebao/spring-cloud-config.git
          search-paths: respo
          username: username
          password: password

server:
  port: 8888

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

> 修改端口需要创建bootstrap.properties
```
server.port=
```

#### 测试

浏览器输入：http://localhost:8888/config-client/dev/master

#### 附HTTP请求地址和资源文件映射
```
 http://ip:port/{application}/{profile}[/{label}]
 http://ip:port/{application}-{profile}.yml
 http://ip:port/{label}/{application}-{profile}.yml
 http://ip:port/{application}-{profile}.properties
 http://ip:port/{label}/{application}-{profile}.properties
```

### 客户端pom中增加：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

#### 客户端项目中application.yml增加Config相关配置，并设置端口号为：8888

```yaml
spring:
  cloud:
    config:
      uri: http://localhost:8888
      name: web-admin-feign # 配置文件名
      label: master
      profile: dev
```
> spring.cloud.config.uri 配置服务中心的网址
> spring.cloud.config.name 配置文件名称的前缀
> spring.cloud.config.label 配置仓库的分支
> spring.cloud.config.profile 配置文件的环境标识

#### 开启Spring Boot Profile

我们在做项目开发的时候，生产环境和测试环境的一些配置可能会不一样，有时候一些功能也可能不一样，所以我们可能会在上线的时候手工修改这些配置信息，但是Spring中为我们提供了Profile这个功能。我们只需要在启动的时候添加一个虚拟机参数，激活自己环境所要用的Profile就可以了。
操作起来很简单，只需要为不同的环境编写专门的配置文件，如：**application-dev.yml**、**application-prod.yml**启动项目时只需增加一个命令参数 --spring.profiles.active=环境配置 即可，命令如下：
```
java -jar hello-spring-cloud-web-admin-feign-1.0.0-SNAPSHOT.jar --spring.profiles.active=prod
```
