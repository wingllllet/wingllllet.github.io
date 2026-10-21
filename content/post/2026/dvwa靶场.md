---
title: "Dvwa靶场"
date: 2026-10-09T14:35:53+08:00
draft: true
---

## 前言

​	重新学习

## 介绍

### docker

Docker 是一个开源的**容器化平台**，用于把应用及其依赖打包成标准化的“容器”，从而实现“一次构建，到处运行”。它常用于开发、测试、部署、CI/CD 和微服务。

### DVWA

是一个专为 Web 安全教学设计的漏洞靶场，‌**官方项目主页为 [https://dvwa.co.uk/](http://www.baidu.com/link?url=5TAKyOIcpxavkrbED372bcKs8FhzWfQYb-7rxjIr-di)，源码托管于 GitHub**‌。它基于 PHP/MySQL 开发，故意集成了多种常见 Web 漏洞，供安全人员在本地隔离环境中练习渗透测试技能与工具使用 。‌‌‌

## docker部署dvwa靶场

- **拉取dvwa镜像文件**

```shell
docker pull vulnerables/web-dvwa
```

- **启动容器**

```shell
docker run -d -p 80:80 --name vulnerables/web-dvwa --restart always citizenstig/dvwa
```

```
-d：后台运行
-p 宿主机端口:容器端口：将宿主机的端口映射到容器的 80 端口
--name dvwa：容器命名为 dvwa，方便后续管理
--restart always：Docker 服务启动或宿主机重启时，容器自动运行
```

- **初始化数据库并登录**

登录网站后点击`Create / Reset Database`，接下来就会初始化数据库并跳转到登录界面。用户名和密码是`admin/password`
