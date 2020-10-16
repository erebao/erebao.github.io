---
title: Spring Secuiry OAuth2授权码模式
date: 2020-03-02 15:49:57
categories:
  - Spring Secuiry OAuth2
tags:
  - Spring Secuiry OAuth2
  - 授权码模式
---

授权码模式（authorization code）是功能最完整、流程最严密的授权模式，code保证了token的安全性，即使code被拦截，由于没有app_secret，也是无法通过code获得token的。
<!--more-->

## 四种模式

- 授权码模式
	- 授权码模式（authorization code）是功能最完整、流程最严密的授权模式，code保证了token的安全性，即使code被拦截，由于没有app_secret，也是无法通过code获得token的。

- 简化模式
	- 和授权码模式类似，只不过少了获取code的步骤，是直接获取令牌token的，适用于公开的浏览器单页应用，令牌直接从授权服务器返回，不支持刷新令牌，且没有code安全保证，令牌容易因为被拦截窃听而泄露。

- 密码模式
	- 使用用户名/密码作为授权方式从授权服务器上获取令牌，一般不支持刷新令牌。

- 客户端凭证模式
	- 一般用于资源服务器是应用的一个后端模块，客户端向认证服务器验证身份来获取令牌。

## 创建认证服务器

创建一个名为**hello-spring-security-oauth2-server**的项目，POM如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.funtl</groupId>
        <artifactId>hello-spring-secuiry-auth2</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>

    <artifactId>hello-spring-security-oauth2-server</artifactId>
    <url>http://www.erebao.com</url>

    <licenses>
        <license>
            <name>Apache 2.0</name>
            <url>https://www.apache.org/licenses/LICENSE-2.0.txt</url>
        </license>
    </licenses>

    <developers>
        <developer>
            <id>erebao</id>
            <name>erebao</name>
            <email>erebao@qq.com</email>
        </developer>
    </developers>

    <dependencies>
        <!-- Spring Boot -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>

        <!-- Spring Security -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <mainClass>com.funtl.spring.security.oauth2.server.OAuth2ServerApplication</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

## 基于内存存储令牌

**服务器安全配置**

- 创建一个类继承**WebSecurityConfigurerAdapter**并添加相关注解：
	- @Configuration
	- @EnableWebSecurity
	- @EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true) 全局方法拦截
	
```java
package com.funtl.spring.security.oauth2.server.configure;

import ......

@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Bean
    public BCryptPasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication().withUser("user").password(passwordEncoder().encode("123456")).roles("USER").and().withUser("admin").password(passwordEncoder().encode("admin888")).roles("ADMIN");
    }
}
```

**配置认证服务器**

- 创建一个类继承**AuthorizationServerConfigurerAdapter**并添加相关注解：
	- @Configuration
	- @EnableAuthorizationServer

```java
package com.funtl.spring.security.oauth2.server.configure;

import ...

@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfiguration extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private BCryptPasswordEncoder passwordEncoder;

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("client")
                .secret(passwordEncoder.encode("secret"))
                .authorizedGrantTypes("authorization_code")
                .scopes("app")
                .redirectUris("https://www.baidu.com");
    }
}
```

**application.yml**

```yaml
spring:
  application:
    name: oauth2-server

server:
  port: 8080
```

#### 测试

**第一步请求服务器获取授权码**

- 浏览器输入：http://localhost:8080/oauth/authorize?client_id=client&response_type=code

- 登录：
	- 账号admin
	- 密码admin888
	
在URL中获得授权码code

**第二步请求服务器获取token**

- 通过curl或postman请求
```
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'grant_type=authorization_code&code=获取到的授权码' "http://client:secret@localhost:8080/oauth/token"
```

## 基于JDBC存储令牌

**初始化oAuth2相关表**

使用官方提供的表脚本初始化oAuth2相关表，地址如下：
> https://github.com/spring-projects/spring-security-oauth/blob/master/spring-security-oauth2/src/test/resources/schema.sql

由于我们使用的Mysql数据库，默认创建表语句中主键为VERCHAT(256)，这超过了最大的主键长度，请手动修改为128，并用BLOB替换语句中的LONGVARBINARY类型，修改后脚本如下：

```sql
create table oauth_client_details (
  client_id VARCHAR(128) PRIMARY KEY,
  resource_ids VARCHAR(256),
  client_secret VARCHAR(256),
  scope VARCHAR(256),
  authorized_grant_types VARCHAR(256),
  web_server_redirect_uri VARCHAR(256),
  authorities VARCHAR(256),
  access_token_validity INTEGER,
  refresh_token_validity INTEGER,
  additional_information VARCHAR(4096),
  autoapprove VARCHAR(256)
);
create table oauth_client_token (
  token_id VARCHAR(256),
  token BLOB,
  authentication_id VARCHAR(128) PRIMARY KEY,
  user_name VARCHAR(256),
  client_id VARCHAR(256)
);
create table oauth_access_token (
  token_id VARCHAR(256),
  token BLOB,
  authentication_id VARCHAR(128) PRIMARY KEY,
  user_name VARCHAR(256),
  client_id VARCHAR(256),
  authentication BLOB,
  refresh_token VARCHAR(256)
);
create table oauth_refresh_token (
  token_id VARCHAR(256),
  token BLOB,
  authentication BLOB
);
create table oauth_code (
  code VARCHAR(256), authentication BLOB
);
create table oauth_approvals (
	userId VARCHAR(256),
	clientId VARCHAR(256),
	scope VARCHAR(256),
	status VARCHAR(10),
	expiresAt TIMESTAMP,
	lastModifiedAt TIMESTAMP
);
create table ClientDetails (
  appId VARCHAR(128) PRIMARY KEY,
  resourceIds VARCHAR(256),
  appSecret VARCHAR(256),
  scope VARCHAR(256),
  grantTypes VARCHAR(256),
  redirectUrl VARCHAR(256),
  authorities VARCHAR(256),
  access_token_validity INTEGER,
  refresh_token_validity INTEGER,
  additionalInformation VARCHAR(4096),
  autoApproveScopes VARCHAR(256)
);
```

**给oauth_client_details表中加入数据**

| client_id | client_secret | scope | authorized_grant_types | web_server_redirect_uri | ...... |
| client | **加密后的secret** | app | authorization_code | http://www.baidu.com | ...... |

**写个测试类得到加密后的secret**
```java
package com.funtl.spring.security.oauth2.server;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class OAuth2ServerApplicationTests {
    @Test
    public void testPasswordEncoder(){
        System.out.println(new BCryptPasswordEncoder().encode("secret"));
    }
}
```

**pom.xml增加**
```xml
<!-- CP -->
<dependency>
	<groupId>com.zaxxer</groupId>
	<artifactId>HikariCP</artifactId>
	<version>${hikaricp.version}</version>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
	<exclusions>
		<!-- 排除tomcat-jdbc 以使用HikariCP -->
		<exclusion>
			<groupId>org.apache.tomcat</groupId>
			<artifactId>tomcat-jdbc</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>${mysql.version}</version>
</dependency>
```
**application.yml修改后如下**
```xml
spring:
  application:
    name: oauth2-server
  datasource:
    type: com.zaxxer.hikari.HikariDataSource
    driver-class-name: com.mysql.cj.jdbc.Driver
    jdbc-url: jdbc:mysql://127.0.0.1:3306/oauth2?useUnicode=true&characterEncoding=UTF-8&useSSL=false
    username: username
    password: password
    hikari:
      minimum-idle: 5
      idle-timeout: 600000
      maximum-pool-size: 10
      auto-commit: true
      pool-name: MyHikariCP
      max-lifetime: 1800000
      connection-timeout: 30000
      connection-test-query: SELECT 1
	  
server:
  port: 8080
```

**AuthorizationServerConfiguration修改后如下**
```java
package com.funtl.spring.security.oauth2.server.configure;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;
import org.springframework.security.oauth2.provider.ClientDetailsService;
import org.springframework.security.oauth2.provider.client.JdbcClientDetailsService;
import org.springframework.security.oauth2.provider.token.TokenStore;
import org.springframework.security.oauth2.provider.token.store.JdbcTokenStore;

import javax.sql.DataSource;

@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfiguration extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private BCryptPasswordEncoder passwordEncoder;

    @Bean
    @Primary
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource dataSource(){
        //配置数据源
        return DataSourceBuilder.create().build();
    }

    @Bean
    public TokenStore tokenStore(){
        //Token放入数据库
        return new JdbcTokenStore(dataSource());
    }

    @Bean
    public ClientDetailsService jdbcClientDetailsService(){
        //从数据库读取客户端配置
        return new JdbcClientDetailsService(dataSource());
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
        endpoints.tokenStore(tokenStore());
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        /*clients.inMemory()
                .withClient("client")
                .secret(passwordEncoder.encode("secret"))
                .authorizedGrantTypes("authorization_code")
                .scopes("app")
                .redirectUris("https://www.baidu.com");*/
        clients.withClientDetails(jdbcClientDetailsService());
    }
}
```

## 基于RBAC的自定义认证

**表结构**
```sql
CREATE TABLE `tb_permission` (
	`id` bigint(20) not null auto_increment,
	`parent_id` bigint(20) default null comment '父权限',
	`name` varchar(64) not null comment '权限名称',
	`enname` varchar(64) not null comment '权限英文名称',
	`url` varchar(255) not null comment '授权路径',
	`description` varchar(200) default null comment '备注',
	`created` datetime not null,
	`updated` datetime not null,
	primary key (`id`)
) engine=InnoDB auto_increment=37 default charset=utf8 comment='权限表'
CREATE TABLE `tb_role` (
	`id` bigint(20) not null auto_increment,
	`parent_id` bigint(20) default null comment '父角色',
	`name` varchar(64) not null comment '角色名称',
	`enname` varchar(64) not null comment '角色英文名称',
	`description` varchar(200) default null comment '备注',
	`created` datetime not null,
	`updated` datetime not null,
	primary key (`id`)
) engine=InnoDB auto_increment=37 default charset=utf8 comment='角色表'
CREATE TABLE `tb_role_permission` (
	`id` bigint(20) not null auto_increment,
	`role_id` bigint(20) default null comment '角色ID',
	`permission_id` bigint(20) default null comment '权限ID',
	primary key (`id`)
) engine=InnoDB auto_increment=37 default charset=utf8 comment='角色权限表'
CREATE TABLE `tb_user` (
	`id` bigint(20) not null auto_increment,
	`username` varchar(50) not null comment '用户名',
	`password` varchar(64) not null comment '密码，加密存储',
	`phone` varchar(20) default null comment '注册手机号',
	`email` varchar(20) default null comment '注册邮箱',
	created datetime not null,
	updated datetime not null,
	primary key (`id`),
	UNIQUE KEY `username` (`username`) using btree,
	unique key `phone` (`phone`) using btree,
	unique key `email` (`email`) using btree
) engine=InnoDB auto_increment=37 default charset=utf8 comment='用户表'
CREATE TABLE `tb_user_role` (
	`id` bigint(20) not null auto_increment,
	`user_id` BIGINT(20) not null comment '用户ID',
	`role_id` bigint(20) not null comment '角色ID',
	primary key (`id`)
) engine=InnoDB auto_increment=37 default charset=utf8 comment='用户角色表'
```

**向表中增加数据**
> tb_user添加用户（密码使用测试类加密）
> tb_role添加角色
> tb_permission添加权限
> tb_user_role和tb_role_permission添加对应关系

**pom.xml增加**
```xml
<dependency>
	<groupId>tk.mybatis</groupId>
	<artifactId>mapper-spring-boot-starter</artifactId>
</dependency>
```
> 版本号为2.1.5，在dependencies中定义

**application.yml增加**

```yaml
mybatis:
  type-aliases-package: com.funtl.spring.security.oath2.server.
  mapper-locations: classpath:mapper/*.xml
```

**WebSecurityConfiguration修改后如下**

```java
package com.funtl.spring.security.oauth2.server.configure;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true, jsr250Enabled = true)
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Bean
    public BCryptPasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Bean
    @Override
    protected UserDetailsService userDetailsService() {
        return new UserDetailsServiceImpl();
    }

    //表示不被拦截
    @Override
    public void configure(WebSecurity web) throws Exception {
        web.ignoring().antMatchers("/oauth/check_token");
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService());
    }
}
```

**增加UserDetailsServiceImpl类**

```java
package com.funtl.spring.security.oauth2.server.configure;

import com.funtl.spring.security.oauth2.server.domain.TbPermission;
import com.funtl.spring.security.oauth2.server.domain.TbUser;
import com.funtl.spring.security.oauth2.server.service.TbPermissionService;
import com.funtl.spring.security.oauth2.server.service.TbUserService;
import org.assertj.core.util.Lists;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private TbUserService tbUserService;

    @Autowired
    private TbPermissionService tbPermissionService;

    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        TbUser tbUser = tbUserService.getByUsername(s);
        List<GrantedAuthority> grantedAuthoritys = Lists.newArrayList();
        if(tbUser != null){
            List<TbPermission> tbPermissions = tbPermissionService.selectByUserId(tbUser.getId());
            tbPermissions.forEach(tbPermission -> {
                GrantedAuthority grantedAuthority = new SimpleGrantedAuthority(tbPermission.getEnname());
                grantedAuthoritys.add(grantedAuthority);
            });
            return new User(tbUser.getUsername(), tbUser.getPassword(), grantedAuthoritys);
        }
        return null;
    }

}
```

**Application**
增加注解
> @MapperScan(basePackages = "com.funtl.spring.security.oauth2.server.mapper")

**mybatis查询数据库**

```java
package com.funtl.spring.security.oauth2.server.service.impl;

import com.funtl.spring.security.oauth2.server.domain.TbUser;
import org.springframework.stereotype.Service;
import javax.annotation.Resource;
import com.funtl.spring.security.oauth2.server.mapper.TbUserMapper;
import com.funtl.spring.security.oauth2.server.service.TbUserService;
import tk.mybatis.mapper.entity.Example;

@Service
public class TbUserServiceImpl implements TbUserService{

    @Resource
    private TbUserMapper tbUserMapper;

    @Override
    public TbUser getByUsername(String username) {
        Example example = new Example(TbUser.class);
        example.createCriteria().andEqualTo("username", username);
        return tbUserMapper.selectOneByExample(example);
    }
}

```

```java
package com.funtl.spring.security.oauth2.server.service.impl;

import com.funtl.spring.security.oauth2.server.domain.TbPermission;
import org.springframework.stereotype.Service;
import javax.annotation.Resource;
import com.funtl.spring.security.oauth2.server.mapper.TbPermissionMapper;
import com.funtl.spring.security.oauth2.server.service.TbPermissionService;

import java.util.List;

@Service
public class TbPermissionServiceImpl implements TbPermissionService{

    @Resource
    private TbPermissionMapper tbPermissionMapper;


    @Override
    public List<TbPermission> selectByUserId(Long id) {
        return tbPermissionMapper.selectByUserId(id);
    }
}
```

TbPermissionMapper.xml
```xml
<select id="selectByUserId" resultMap="BaseResultMap">
	select p.*
	from  tb_user as u
	left join tb_user_role as ur on u.id = ur.user_id
	left join tb_role as r on r.id = ur.role_id
	left join tb_role_permission as rp on r.id = rp.role_id
	left join tb_permission as p on p.id = rp.permission_id
	where u.id = #{id}
</select>
```

> 此处省略其他mybatis查询数据功能代码
