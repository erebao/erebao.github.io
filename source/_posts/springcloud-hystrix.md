---
title: Spring Cloud Hystrix熔断器防止服务雪崩
date: 2020-03-01 13:23:57
categories:
  - Spring Cloud
tags:
  - Spring Cloud
  - 微服务
  - Hystrix
  - 熔断器
---

在微服务架构中，根据业务来拆分成一个个的服务，服务于服务之间可以通过RPC相互调用，在Spring Cloud中可以用RestTemplate + Ribbon和Feign来调用，为了保证其高可用，单个服务通常会集群部署。由于网络原因或者自身的原因，服务并不能保证100%可用，如果单个服务出现问题，调用这个服务就会出现线程阻塞，此时若有大量的请求涌入，Servlet容器的线程资源就会被消耗完毕，导致服务瘫痪。服务与服务之间的依赖性，故障会传播，会对整个微服务系统造成灾难性的严重后果，为了解决这个问题，业界提出了熔断器模型。Netflix开源了Hystrix组件，实现了熔断器模式，Spring Cloud对这一组件进行了整合。
<!--more-->

## Ribbon中使用熔断器

在**pom.xml**中增加依赖：
```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

#### Application

增加注解：
> @EnableHystrix

#### Service增加@HystrixCommand注解

```java
package com.funtl.hello.spring.cloud.web.admin.ribbon.service;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

@Service
public class AdminService {

    @Autowired
    private RestTemplate restTemplate;

    @HystrixCommand(fallbackMethod="hiError")
    public String sayHi(String message){
        return restTemplate.getForObject("http://hello-spring-cloud-service-admin/hi?message=" + message, String.class);
    }

    public String hiError(String message){
        return String.format("Hi your message is : %s but request error", message);
    }

}
```

#### 测试

此时我们关闭服务提供者，再次请求http://localhost:8764/hi?message=HelloRibbon浏览器会显示：
> Hi your message is : HelloRibbon but request error



## Feign中使用熔断器

Feign是自带熔断器的，但默认是关闭的，需要在配置文件**application.yml**中配置打开它：
```yaml
feign:
  hystrix:
    enabled: true
```

#### 在Service中增加fallback指定类

```java
package com.funtl.hello.spring.cloud.web.admin.feign.service;

import com.funtl.hello.spring.cloud.web.admin.feign.service.hystrix.AdminServiceHystrix;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;

@FeignClient(value = "hello-spring-cloud-service-admin", fallback = AdminServiceHystrix.class)
public interface AdminService {

    @RequestMapping(value = "hi", method = RequestMethod.GET)
    public String sayHi(@RequestParam(value = "message") String message);

}
```

#### 创建熔断器类并实现对应的Feign接口

```java
package com.funtl.hello.spring.cloud.web.admin.feign.service.hystrix;

import com.funtl.hello.spring.cloud.web.admin.feign.service.AdminService;
import org.springframework.stereotype.Component;

@Component
public class AdminServiceHystrix implements AdminService {

    @Override
    public String sayHi(String message) {
        return String.format("Hi your message is : %s but request bad", message);
    }
}
```
