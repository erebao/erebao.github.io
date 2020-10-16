---
title: Spring Cloud Hystrix熔断器仪表盘监控
date: 2020-03-01 22:11:57
categories:
  - Spring Cloud
tags:
  - Spring Cloud
  - 微服务
  - Hystrix仪表盘
---

在Ribbon和Feign项目增加Hystrix仪表盘功能，两个项目的改造方式相同
<!--more-->

## 在pom.xml中增加依赖

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```

#### Application

增加注解：
> @EnableHystrixDashboard注解

#### 创建hystrix.stream的Servlet配置

Spring Boot 2.x版本开启Hystrix Dashboard与Spring Boot 1.x的方式略有不同，需要增加一个HystrixMetricsStreamServlet的配置，代码如下：

```java
package com.funtl.hello.spring.cloud.web.admin.feign.config;

import com.netflix.hystrix.contrib.metrics.eventstream.HystrixMetricsStreamServlet;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class HystrixDashboardConfiguration {

    @Bean
    public ServletRegistrationBean getServlet(){
        HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
        ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);
        registrationBean.setLoadOnStartup(1);
        registrationBean.addUrlMappings("/hystrix.stream");
        registrationBean.setName("HystrixMetricsStreamServlet");
        return registrationBean;
    }
}

```

#### 测试访问

在浏览器上访问：http://localhost:8764/hystrix
