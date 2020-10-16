---
title: docker-compose.yml方式安装Nexus
date: 2020-02-29 14:12:57
categories:
  - docker
tags:
  - docker
  - docker-compose
  - Nexus
---

Nexus可以做Maven私服
<!--more-->

## Nexus 安装

使用docker-compose.yml来安装和运行Nexus
```yaml
version: '3.1'
services:
  nexus:
    restart: always
    image: sonatype/nexus3:3.13.0
    container_name: nexus
    ports:
      - 8081:8081
    volumes:
      - /usr/local/docker/nexus/data:/nexus-data
```

> 启动如果出现权限问题可以使用：chmod 777 /usr/local/docker/nexus/data
> 默认账号密码 admin  admin123

## Nexus 使用

####配置认证信息

在Maven setting.xml中添加Nexus认证信息（servers节点下）：
```xml
<server>
	<id>nexus-releases</id>
	<username>admin</username>
	<password>admin123</password>
</server>
<server>
	<id>nexus-snapshots</id>
	<username>admin</username>
	<password>admin123</password>
</server>
```
- nexus-releases用于发布Release的版本
- nexus-snapshots用于发布Snapshot版本（快照版）
- 在项目**pom.xml**中设置版本号添加**SNAPSHOT**标识的都会发布为SNAPSHOT版本，没有SNAPSHOT标识的都会发布为RELEASE版本

####配置自动化部署

```xml
<distributionManagement>
	<repository>
		<id>nexus-releases</id>
		<name>Nexus Release Repository</name>
		<url>http://127.0.0.1:8081/repository/maven-releases</url>
	</repository>
	<snapshotRepository>
		<id>nexus-snapshots</id>
		<name>Nexus Snapshot Repository</name>
		<url>http://127.0.0.1:8081/repository/maven-snapshots</url>
	</snapshotRepository>
</distributionManagement>
```
- ID名称必须要与**setting.xml**中Servers配置的ID名称保持一致
- 项目版本号中有**SNAPSHOT**标识的，会发布到Nexus Snapshots Repository，否则发布到Nexus Releases Repository

####部署到仓库

> mvn deploy

####上传第三方Jar包

Nexus3.0不支持页面上传，可使用maven命令：
```
mvn deploy:deploy-file
  -DgroupId=com.google.code.kaptcha
  -DartifactId=kaptcha
  -Dversioin=2.3
  -Dpackaging=jar
  -Dfile=D:\kaptcha-2.2.3.jar
  -Durl=http://127.0.0.1:8081/repository/maven-releases/
  -DrepositoryId=nexus-releases
```
- **合并一行**后执行
- **-DrepositoryId=nexus-releases**对应的是**setting.xml**中Servers配置的ID名称

####配置代理仓库

```xml
<repositories>
	<repository>
		<id>nexus</id>
		<name>Nexus Repository</name>
		<url>http://127.0.0.1:8081/repository/maven-public</url>
		<snapshots>
			<enabled>true</enabled>
		</snapshots>
		<releases>
			<enabled>true</enabled>
		</releases>
	</repository>
</repositories>
```

```xml
<pluginRepositories>
	<pluginRepository>
		<id>nexus</id>
		<name>Nexus Plugin Repository</name>
		<url>http://127.0.0.1:8081/repository/maven-public</url>
		<snapshots>
			<enabled>true</enabled>
		</snapshots>
		<releases>
			<enabled>true</enabled>
		</releases>
	</pluginRepository>
</pluginRepositories>
```

####配置IntelliJ IDEA使用最新快照

> File -> Settings... -> Build, Execution, Deployment -> Build Tools -> Maven

> Always update snapshots 打上勾即可

## 巨坑! CI/CD持续集成环境下Nexus提示 Not authorized 问题

> 开启匿名访问即可

> Server administration and configuration ——> Security ——> Anonymous Access

> 打勾 Allow anonymous users to access the server
