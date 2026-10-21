---
title: "Wireshark知识总结"
date: 2024-06-27T07:48:56+08:00
draft: true
---

## wireshark介绍和基本操作

### wireshark是什么

- wireshark是一个可以进行数据包的捕获和分析的软件
- 网卡在对接收到的数据包进行处理之前，会先对它们的目的地址进行检查，如果目的地址不为本机的话就会丢弃这些数据，相反就会将这些数据包交给操作系统，操作系统再将其分配给应用程序。如果启动了wireshark的话，操作系统会将经过网卡的所有数据包都复制一份提供给它，这样我们就可以再wireshark中查看到本机进出的数据包。

### 操作过程

#### 1、 选择合适的网卡

这里我用的是虚拟机的`wireshark`，所以后续选择`eth0`网卡

![image-20240627080708498](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202406270807598.png)

```
ps. 鼠标放在网卡名称上会展示该网卡的地址等信息
```

#### 2、开始捕获数据包

双击`eth0`进入抓包界面，然后在浏览器中访问`www.wireshark.org`，当我们捕获到了想要的数据包后就可以停止捕获并开始分析。

![image-20240627081549316](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202406270815439.png)

![20240627083247](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202406270833417.png)

#### 3、过滤掉无用的数据包

现在要查看访问wireshark官网所产生的数据包，选择`统计->会话`

![image-20240627084059645](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202406270840762.png)

所展示的是网卡ip与其他ip之间的交流信息。

如果想要了解该ip的域名什么，需要开启网络ip解析功能，选择`视图->名称解析->解析网络地址`

```
wireshark并不自带解析功能，它需要向dnr服务器发送请求。
（dnr服务器，即域名解析服务器。负责将域名映射成IP地址）
```

![image-20240627091239034](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202406270912078.png)

接下来右键这栏，选择`作为过滤器应用->选中-> A<->B `，就会对捕获的数据进行过滤展示

#### 4、将捕获到的数据包保存为文件

选择`文件->保存/另存为`

如果我们只想把访问`wireshark`官网的流量保存为文件，选择`文件->导出特定分组`

## 网络安全仿真环境构建

### 网络设备的仿真工具

- 一个完整的网络结构通常都是由各种网络设备和计算机共同构成的。
- 对于大多数学习者来说，购买全套的交换机、路由器以及防火墙等设备是不现实的。不过好在现在又很多工具软件都提供了网络设备的虚拟功能，其中最著名的包括GNS3和eNSP等。

### eNSP安装与使用

![image-20240703110154625](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407031102381.png)

```
左侧：可以模仿华为设备
中间：eNSP自带的拓扑协议
```

#### 创建内网环境

![image-20240703111429005](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407031114406.png)

```
交换机：s3700
终端：PC
设备连线：auto
```

### 虚拟PC工具VMware

![image-20240703111623058](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407031116150.png)

#### 安装kali linux

![image-20240703112222552](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407031122585.png)

#### 网络设置

```
桥接
nat
仅主机
```

在新建的拓扑图中添加`其他设备-cloud`，并设置如下，为的是绑定与`kali linux`的网卡

![image-20240703145722327](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407031457734.png)

接着选择一个`PC`，设置如下

![image-20240703145821457](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407031458556.png)

![image-20240703150529559](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407031505868.png)

这样这个终端的网段就与我们的`kali linux`一样，之后可以互相`ping`通，就算成功。接下来可以在它们相互`ping`的时候尝试用`wireshark`进行抓包

![image-20240703151201342](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407031512900.png)

## wireshark的部署方式

### 远程完成远程数据包捕获

- `wireshark`提供了远程数据包捕获功能，可以方便我们监控远程服务器上的流量。只需要在服务器上安装`rpcapd`即可

- 如果服务器是windows系统可以直接安装`winpcap`

![image-20240705075704543](C:\Users\winglet\AppData\Roaming\Typora\typora-user-images\image-20240705075704543.png)

- rpcapd有被动工作和主动工作两种模式，其中被动模式需要客户端连接服务器。一般服务器所在网络都部署了防火墙，且使用了NAT既相互，这种情况下客户端无法连接服务器，就需要选择主动工作，服务器主动连接客户端。常用参数如下

```
-b  指定rpcapd进程监听的IP地址。若省略该项表示rpcapd监听所有本地的IPV4地址
-p  指定rpcapd进程监听的端口，默认是2002，可省略
-l  指定一个地址列表文件，允许哪些地址可以访问rpcap进程
-n  不启用认证功能，任何主机都可以访问rpcapd进程
-d  守护进程方式运行
```

进入windows系统`C:\Program Files (x86)\WinPcap`，进入`shell`界面，输入

```
rpcapd.exe -n
```

开启服务

接着打开另一台主机的`wireshark`工具，选择`捕获选项->manage interface->远程接口`，进行添加

![image-20240707212308561](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407072123822.png)

之后捕获的流量将是这台主机的流量

```
ps. kali中的wireshark中没有远程接口选项
```

### 集线器环境

集线器（Hub）是一种基本的网络设备，用于将多个网络设备连接到一个共享的通信通道上。集线器不具备智能转发功能，它简单地将所有接收到的数据包广播到所有连接的端口，无论数据包的目的地是哪一个设备。常用于星型拓扑的网络结构中。

### 交换环境

以太网交换机是一种网络设备，用于连接多个网络设备（如计算机、打印机、服务器等），并根据数据包的MAC地址进行智能转发。交换机能够识别每个端口的数据包目的地，并仅将数据包发送到目标设备所在的端口，从而提高网络效率和性能。

### 端口镜像

端口镜像是将网络设备上指定端口接收或发送的报文复制到目的端口，可以只复制端口接收或者发送的报文，也可以同时复制接收和发送的报文。

- 如果你的主机支持镜像端口，可以将监视的流量都转发到这个端口上，这样也可以监视其他主机的流量。

网络设备图如下

![image-20240714201519827](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407142015002.png)

```
PC1:
ip地址：192.168.1.100
子网掩码：255.255.255.0
网关：192.168.1.0

PC2:
ip地址：192.168.2.100
子网掩码：255.255.255.0
网关：192.168.2.0
```

现在视`PC1`的网络环境为内网，`PC2`为外网，它们之间流量传输都要经过`AR2`路由，现在我们想要监测它们流量，就是流经`GE 0/0/1`的流量，那么可以把`Ethernet 0/0/0`作为镜像接口，将`GE 0/0/1`的流量都复制一份到`Ethernet 0/0/0`，这样就能在`PC3`中实现监控`PC1/2`之间的流量信息。

- 右键`AR2` ，选择`CLI`，输入以下命令

![image-20240714202537807](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407142025879.png)

![image-20240714211710560](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407142117612.png)

```cli
<huawei> sys				进入系统视图
[huawei] observe-port interface e0/0/0
[huawei] int GigabitEthernet 0/0/0
[huawei-GigabitEthernet0/0/0] ip address 192.168.1.1 255.255.255.0
[Huawei] int GigabitEthernet 0/0/1
[Huawei-GigabitEthernet0/0/1] ip address 192.168.2.1 255.255.255.0
[Huawei-GigabitEthernet0/0/1] mirror to observe-port ?
[Huawei-GigabitEthernet0/0/1] mirror to observe-port both
```

这样就配置好了端口镜像

- 右键`pc3`或者`e 0/0/0`，选择抓包，之后用`PC1`主机使用`pc2`使用`ping`命令，就可以在`PC3`中监测它们之间发送的`ICMP`流量报文

![image-20240714215852780](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407142158867.png)

### ARP欺骗

```
地址解析协议（英语：Address Resolution Protocol，缩写：ARP）是一个通过解析网络层地址来找寻数据链路层地址的网络传输协议，它在IPv4中极其重要。简单来说是通过网络地址来定位MAC地址

ARP欺骗，是针对以太网地址解析协议（ARP）的一种攻击技术。此种攻击可让攻击者获取局域网上的数据包甚至可篡改数据包，且可让网络上特定电脑或所有电脑无法正常连线。

ARP欺骗的运作原理是由攻击者发送假的ARP数据包到网络上，尤其是送到网关上。其目的是要让送至特定的IP地址的流量被错误送到攻击者所取代的地方。因此攻击者可将这些流量另行转送到真正的网关（被动式数据包嗅探，passive sniffing）或是篡改后再转送（中间人攻击，man-in-the-middle attack）。攻击者亦可将ARP数据包导到不存在的MAC地址以达到拒绝服务攻击的效果，例如netcut软件。
```

- 大多情况，我们无法更改网络物理线路，也不能使用交换机的端口镜像功能。ARP欺骗无需对网络做出任何的改动，只需要在自己计算机上运行欺骗工具即可。

![image-20240715171136451](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407151711604.png)

```
PC1: DHCP
ip地址: 192.168.153.137

PC2: DHCP
ip地址: 192.168.153.136

cloud1: 
```

正常情况下，`PC1`与`PC2`之间的流量传输，`cloud1`是无法监测的，想要监测的话就需要进行ARP欺骗。

- 使用`kali`自带的`ettercap`

![image-20240715172305117](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407151723187.png)

- 接下来选择ARP欺骗功能

![image-20240715172911005](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407151729089.png)

- 之后就能监听`PC1/PC2`之间的流量包

![image-20240715173018879](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202407151730960.png)

```
缺点：该行为容易被检测出来
```

### 网络分组器

- 网络分组器作用：原来的流量正常通行，同时复制一份出来供监测设备分析使用。

### 本地流量的捕获

- 简单来说就是捕获`127.0.0.1`的流量

### 对虚拟机流量捕获