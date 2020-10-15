---
title: 单体应用架构下的spring security + oauth2功能实现
date: 2020-03-25 15:49:57
categories:
  - spring security
  - oauth2
tags:
  - spring security
  - oauth2
---

单体应用架构下的spring security + oauth2功能实现
<!--more-->

## 所需JAR
pom.xml
```xml
<!-- jwt -->
<dependency>
	<groupId>org.springframework.security</groupId>
	<artifactId>spring-security-jwt</artifactId>
	<version>1.0.10.RELEASE</version>
</dependency>
<!-- security -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
	<version>2.0.3.RELEASE</version>
</dependency>
<!-- oauth2 -->
<dependency>
	<groupId>org.springframework.security.oauth</groupId>
	<artifactId>spring-security-oauth2</artifactId>
	<version>2.1.0.RELEASE</version>
</dependency>
```

## 配置文件
application.properties
```
############################################################
# 自定义配置在Oauth2Configuration中认证服务器中使用
############################################################
#客户端
authentication.oauth.clientId=app
#密钥
authentication.oauth.secret=secret
#令牌有效期
authentication.oauth.tokenValiditySeconds=7200
#签名密钥
authentication.oauth.signingKey=sign
#授权范围
authentication.oauth.scopes=all
```

## OAuth2总配置类
Oauth2Configuration.java

```java
package com.reai.drive.oauth2;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.oauth2.config.annotation.configurers.ClientDetailsServiceConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configuration.AuthorizationServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableAuthorizationServer;
import org.springframework.security.oauth2.config.annotation.web.configuration.EnableResourceServer;
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerEndpointsConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configurers.AuthorizationServerSecurityConfigurer;
import org.springframework.security.oauth2.config.annotation.web.configurers.ResourceServerSecurityConfigurer;
import org.springframework.security.oauth2.provider.ClientDetailsService;
import org.springframework.security.oauth2.provider.code.AuthorizationCodeServices;
import org.springframework.security.oauth2.provider.code.InMemoryAuthorizationCodeServices;
import org.springframework.security.oauth2.provider.token.*;
import org.springframework.security.oauth2.provider.token.store.JwtAccessTokenConverter;
import org.springframework.security.oauth2.provider.token.store.JwtTokenStore;

import java.util.Arrays;

@Configuration
public class Oauth2Configuration {

    //资源服务器
    @Configuration
    @EnableResourceServer
    protected static class ResourceServerConfiguration extends ResourceServerConfigurerAdapter {
        @Value("${authentication.oauth.signingKey}")
        private String signingKey;

        //使用内存存储令牌
        @Bean
        public TokenStore tokenStore(){
            //JWT令牌存储方案
            return new JwtTokenStore(accessTokenConverter());
        }
        @Bean
        public JwtAccessTokenConverter accessTokenConverter(){
            JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
            converter.setSigningKey(signingKey);
            return converter;
        }

        @Override
        public void configure(ResourceServerSecurityConfigurer resources) throws Exception {
            resources.tokenStore(tokenStore())
                    .stateless(true);
        }
        @Override
        public void configure(HttpSecurity http) throws Exception {
            http
                .authorizeRequests()
                //不拦截swagger
                .antMatchers("/swagger*//**", "/v2/api-docs", "/webjars*//**").permitAll()
                //除了swagger的其他都拦截
                .antMatchers("/**").access("#oauth2.hasScope('all')")
                .and().csrf().disable()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
        }
    }

    //认证服务器
    @Configuration
    @EnableAuthorizationServer
    public static class AuthorizationServerConfiguration extends AuthorizationServerConfigurerAdapter {
        @Value("${authentication.oauth.clientId}")
        private String clientId;
        @Value("${authentication.oauth.secret}")
        private String secret;
        @Value("${authentication.oauth.tokenValiditySeconds}")
        private String tokenValiditySeconds;
        @Value("${authentication.oauth.signingKey}")
        private String signingKey;
        @Value("${authentication.oauth.scopes}")
        private String scopes;

        @Autowired
        private AuthenticationManager authenticationManager;
        @Autowired
        private ClientDetailsService clientDetailsService;

        //JWT令牌存储方案
        @Bean
        public TokenStore tokenStore(){
            return new JwtTokenStore(accessTokenConverter());
        }
        @Bean
        public JwtAccessTokenConverter accessTokenConverter(){
            JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
            converter.setSigningKey(signingKey);
            return converter;
        }
        //授权码的使用模式
        @Bean
        public AuthorizationCodeServices authorizationCodeServices(){
            return new InMemoryAuthorizationCodeServices();
        }
        //令牌管理模式
        @Bean
        public AuthorizationServerTokenServices tokenServices(){
            DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
            defaultTokenServices.setClientDetailsService(clientDetailsService);//客户端详情服务
            defaultTokenServices.setSupportRefreshToken(true);//支持刷新令牌
            defaultTokenServices.setTokenStore(tokenStore());//令牌存储策略

            //令牌增强
            TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
            tokenEnhancerChain.setTokenEnhancers(Arrays.asList(accessTokenConverter()));
            defaultTokenServices.setTokenEnhancer(tokenEnhancerChain);

            defaultTokenServices.setAccessTokenValiditySeconds(Integer.parseInt(tokenValiditySeconds)); //令牌有效期
            defaultTokenServices.setRefreshTokenValiditySeconds(259200); //刷新令牌默认有效期3天
            return defaultTokenServices;
        }
        //令牌访问端点和令牌服务
        @Override
        public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
            endpoints
                    .authenticationManager(authenticationManager)//密码模式
                    .authorizationCodeServices(authorizationCodeServices())//授权码模式
                    .tokenServices(tokenServices())//令牌管理模式
                    .allowedTokenEndpointRequestMethods(HttpMethod.POST);
        }
        //令牌端点安全约束
        @Override
        public void configure(AuthorizationServerSecurityConfigurer security) throws Exception {
            security
                    .tokenKeyAccess("permitAll()")// oauth/token_key 公钥公开
                    .checkTokenAccess("permitAll()")// oauth/check_token 检测token公开
                    .allowFormAuthenticationForClients();//表单认证（申请令牌）
        }
        //客户端详情服务
        @Override
        public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
            clients.inMemory()//使用内存存储
                    .withClient(clientId)//客户端名称
                    .secret(secret)//密钥
                    .authorizedGrantTypes("authorization_code", "password", "client_credentials", "implicit", "refresh_token")//允许的oauth授权类型
                    .scopes(scopes);//允许的授权范围
        }
    }

}
```

## 配置SpringSecurity相关的内容
SecurityConfiguration.java
```java
package com.reai.drive.oauth2;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.password.NoOpPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;

@Configuration
@EnableWebSecurity
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Value("${authentication.oauth.scopes}")
    private String scopes;

    //密码编码器
    @Bean
    public PasswordEncoder passwordEncoder(){
        return NoOpPasswordEncoder.getInstance();
    }

    //定义用户信息服务（查询用户信息）
    @Bean
    public UserDetailsService userDetailsService(){
        InMemoryUserDetailsManager manage = new InMemoryUserDetailsManager();
        manage.createUser(User.withUsername("admin").password("123456").authorities("ROLE_APP").build());
        return manage;
    }

    //认证管理器
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception{
        return super.authenticationManagerBean();
    }

    //安全拦截机制
    /*@Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .authorizeRequests()
                .antMatchers("/**").authenticated()//所有/**的请求必须认证通过
                .anyRequest().permitAll();//除了/**，其它的请求可以访问
    }*/

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService());
    }
}
```

## 调用

账号密码模式获取token：
```
POST
http://localhost:8080/oauth/token
参数：
client_id：app
client_secret：secret
username：admin
password：123456
grant_type：password
```

其他方式或更多请参考：
https://github.com/erebao/distributed-security
