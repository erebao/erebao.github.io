---
title: docker-compose.yml方式安装GitLab中文版
date: 2020-02-29 12:12:57
categories:
  - docker
tags:
  - docker
  - docker-compose
  - GitLab
---

GitLab 是一个用于仓库管理系统的开源项目，使用Git作为代码管理工具，并在此基础上搭建起来的web服务。安装方法是参考GitLab在GitHub上的Wiki页面
<!--more-->

## GitLab 安装

使用docker-compose.yml来安装和运行GitLab中文版
```yaml
version: '3'
services:
  web:
    image: 'twang2218/gitlab-ce-zh:11.0.5'
    restart: always
    hostname: '192.168.1.129'
    environment:
      TZ: 'Asia/Shanghai'
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://192.168.1.129:8080'
        gitlab_rails['gitlab_shell_ssh_port'] = 2222
        unicorn['port'] = 8888
        nginx['listen_port'] = 8080
    ports:
      - '8080:8080'
      - '8443:443'
      - '2222:22'
    volumes:
      - /usr/local/docker/gitlab/config:/etc/gitlab
      - /usr/local/docker/gitlab/data:/var/opt/gitlab
      - /usr/local/docker/gitlab/logs:/var/log/gitlab
```
