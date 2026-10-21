---
title: "[HTB]Web_Requests"
date: 2025-10-01T14:41:32+08:00
image: https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20251001145139223.png
draft: true
---

## 前言

开始学习HTB上的知识。

## 准备工作

HTB上准备了web页面的虚拟机，但是用的时候很卡。因此可以选择下载openvpn配置文件，然后在本地环境进行练习。

![image-20251001145730046](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20251001145730201.png)

点击“用户-VPN设置”。

![](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20251001145821452.png)

接下来下载合适的openvpn文件，之后按照教程，在自己的linux虚拟机的终端中输入`open academy-regular.ovpn`

## Web Request

### 超文本传输协议 （HTTP）

大多数互联网通信都是通过 HTTP 协议发出的 Web 请求。HTTP是一种用于访问万维网资源的应用级协议。HTTP 通信由客户端和服务器组成，客户端向服务器请求资源。服务器处理请求并返回请求的资源。HTTP 通信的默认端口是 port `80`，但可以根据 Web 服务器的配置将其更改为任何其他端口。当我们使用互联网访问不同的网站时，也会使用相同的请求。我们输入`Fully Qualified Domain Name`( `FQDN`) 作为`Uniform Resource Locator`( `URL`) 来访问所需的网站，例如[www.hackthebox.com](http://www.hackthebox.com/)。

### 网址

HTTP 上的资源是通过 访问的`URL`，`URL`结构如下

![URL 结构图：方案“http”，用户“admin:password”，主机“inlanefreight.com”，端口“80”，路径“/dashboard.php”，查询字符串“login=true”，片段“status”。](https://academy.hackthebox.com/storage/modules/35/url_structure.png)

其组件含义如下：

| **成分**       | **例子**             | **描述**                                                     |
| -------------- | -------------------- | ------------------------------------------------------------ |
| `Scheme`       | `http://` `https://` | 用于识别客户端正在访问的协议，以冒号和双斜杠 ( `://`)结尾    |
| `User Info`    | `admin:password@`    | 这是一个可选组件，包含用于向主机进行身份验证的凭据（以冒号分隔），并以 at 符号 ( )`:`与主机分隔。`@` |
| `Host`         | `inlanefreight.com`  | 主机表示资源位置。可以是主机名或 IP 地址                     |
| `Port`         | `:80`                | `Port`和`Host`之间用冒号 ( )分隔`:`。如果没有指定端口，`http`则方案默认为端口`80`，`https`默认为端口`443` |
| `Path`         | `/dashboard.php`     | 指向正在访问的资源，可以是文件或文件夹。如果没有指定路径，服务器将返回默认索引（例如`index.html`）。 |
| `Query String` | `?login=true`        | 查询字符串以问号 ( `?`) 开头，由一个参数（例如`login`）和一个值（例如`true`）组成。多个参数可以用与号 ( `&`) 分隔。 |
| `Fragments`    | `#status`            | 客户端的浏览器会处理片段来定位主要资源中的各个部分（例如页面上的标题或部分）。 |

### HTTP 流

![图示为用户访问 inlanefreight.com。浏览器发送 DNS 查询以查找 IP 地址，收到 152.153.81.14，然后向 Web 服务器发出 HTTP 请求，Web 服务器响应“HTTP/1.1 200 OK”。](https://academy.hackthebox.com/storage/modules/35/HTTP_Flow.png)

