---
title: "Upload Labs靶场"
date: 2026-10-11T22:45:34+08:00
draft: true
---

## 前言

​	之前打`uploadlabs`靶场的笔记不小心删了，然后现在也是忘了不少，所以算是重新复习一下。当然这次会笔记会做的详细一些。不过只有前两道会详细一下解题步骤，后面我熟悉的就简化步骤，不熟悉的再详细步骤。

## 介绍

`upload-labs`是一个使用`php`语言编写的，专门收集渗透测试和CTF中遇到的各种上传漏洞的靶场。旨在帮助大家对上传漏洞有一个全面的了解。目前一共21关，每一关都包含着不同上传方式。

###  前置知识

- 文件上传正常流程：选择文件 -> 请求上传 -> 临时文件 -> 后端校验 -> 移动文件 -> 返回路径
- 前端校验特点：
- 后端校验特点：
- MIME 与 `Content-Type`：
- 文件解析规则：

# UploadLabs 学习总览

> 仅用于本地授权靶场学习。

### 1. 进度表

| Pass    | 考点             | 状态 | 日期       | 笔记                       | 一句话总结                                                   |
| ------- | ---------------- | ---- | ---------- | -------------------------- | ------------------------------------------------------------ |
| Pass-01 | 前端 JS 校验     | 完成 | 2026.10.16 | 禁用js或者bp抓包改文件后缀 | 前端校验不可信。**前端 JavaScript 运行在用户的浏览器里，用户完全可以控制、修改、禁用、绕过它。** |
| Pass-02 | MIME文件类型校验 | 完成 | 2026.10.16 | 抓包改MIME类型             | 前端校验只做体验，服务端校验才是安全边界。服务端也不能信任 MIME，要检查真实文件内容。 |
|         |                  |      |            |                            |                                                              |

### 2. 知识地图

- 前端校验：JS、accept、onsubmit、隐藏字段
- 后端校验：MIME、后缀黑名单、后缀白名单、内容检测、二次渲染
- 解析漏洞：Apache、Nginx、IIS、.htaccess、.user.ini
- 绕过技巧：大小写、双写、点、空格、`::$DATA`、`%00`、目录穿越
- 竞争条件：上传临时文件、删除前访问
- 图片马：文件头、EXIF、二次渲染、文件包含配合

### 3. 错题本

| 日期 | 关卡 | 问题 | 原因 | 正确思路 |
| ---- | ---- | ---- | ---- | -------- |
|      |      |      |      |          |

## Pass-1

### 1. 关卡信息

| 项目     | 内容     |
| -------- | -------- |
| 关卡     | Pass-1   |
| 难度     | 低       |
| 考点     | 前端校验 |
| 后端语言 | PHP      |

### 2. 搜集信息

#### 测试上传功能

-  上传图片

![image-20261016101512500](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20261016103755978.jpg)

上传成功

- 上传木马

```php
<?php
@eval($_GET["c"])
?>
```

![image-20261016101624920](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20261016103808025.jpg)

​	无法上传，并告诉可上传的白名单文件。不过出现弹窗提示我们这个文件检验功能可能是前端js检验。

- 查看网页源代码

![image-20261016101818986](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20261016103814300.jpg)发现该网页确实将文件检验写在了前端，不过不确定服务器端是否存在检验。

### 3. 攻击手段

1.  在火狐浏览器上安装`javascript switcher`插件。

![image-20261016103206478](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20261016103822935.jpg)

> `javascript switcher`可以控制网页js代码的执行或禁用。

我们选择禁用js代码，然后上传木马文件。

![image-20261016103405086](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20261016103902802.jpg)

成功上传，接着连接木马。

![image-20261016103618591](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20261016103909394.jpg)

2.  使用bp抓包修改文件后缀

### 4. 漏洞原理

#### 前端检验不可信

安全边界不能放在客户端。
前端校验只能防“手滑选错文件”，不能防攻击者。

攻击者可以不使用你的浏览器，直接构造 HTTP 请求上传文件。
也可以禁用 JS、改前端代码、用 Burp 改包。
所以如果服务端没有再次校验，攻击者就能上传 `.php`、`.jsp`、`.asp` 等恶意文件，进一步 getshell。

漏洞根源不是这段 JS 本身，而是：

1. 服务端信任了前端校验结果；
2. 服务端根本没有校验；
3. 或者服务端校验不严格。

### 5. 防御手段

**核心原则：**

> **前端校验只做用户体验，服务端校验才是安全边界。**

#### 1）白名单校验扩展名

```php
$allow = ['jpg', 'png', 'gif'];
$ext = strtolower(pathinfo($_FILES['file']['name'], PATHINFO_EXTENSION));
if (!in_array($ext, $allow, true)) {
    die('不允许上传');
}
```

#### 2）检查 MIME 和文件头

```php
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mime = finfo_file($finfo, $_FILES['file']['tmp_name']);
$allowMime = ['image/jpeg', 'image/png', 'image/gif'];
if (!in_array($mime, $allowMime, true)) {
    die('MIME 不允许');
}
```

#### 3）重命名文件

```php
$newName = bin2hex(random_bytes(16)) . '.' . $ext;
move_uploaded_file($_FILES['file']['tmp_name'], '/var/www/uploads/' . $newName);
```

#### 4）上传目录不可执行

```nginx
# nginx
location ^~ /uploads/ {
    location ~ \.(php|php5|phtml)$ {
        deny all;
    }
}
```

#### 5）限制权限

- Web 用户不能写可执行目录；
- 上传目录权限最小化；
- 禁止目录执行脚本。

#### 6）二次渲染

对图片进行重新压缩、生成缩略图，去掉可能夹带的恶意代码。

## Pass-2

### 1. 关卡信息

| 项目     | 内容     |
| -------- | -------- |
| 关卡     | Pass-2   |
| 难度     | 低       |
| 考点     | MIME校验 |
| 后端语言 | PHP      |

**源码：**

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        if (($_FILES['upload_file']['type'] == 'image/jpeg') || ($_FILES['upload_file']['type'] == 'image/png') || ($_FILES['upload_file']['type'] == 'image/gif')) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH . '/' . $_FILES['upload_file']['name']            
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '文件类型不正确，请重新上传！';
        }
    } else {
        $msg = UPLOAD_PATH.'文件夹不存在,请手工创建！';
    }
}
```

> 提示： 本pass在服务端对数据包的MIME进行检查！

### 2. 搜集信息

- 上传 木马文件

  ![image-20261016104642278](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20261016104642373.jpg)

  文件类型不正确提示本关是后端检验MIME类型。

### 3. 攻击手段

- **上传木马文件，并用bp抓包**

  ![image-20261016105022405](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20261016105022494.jpg)

  可以看到我们上传的`MIME类型 ：application/octet-stream`，这个类型大概不在检测白名单中。我们换成图片的`image/jpeg`

- **接着发包。**

  ![image-20261016105351899](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20261016105351994.jpg)

  上传成功。

- **连接脚本**

  ![image-20261016105510163](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20261016105510399.jpg)

### 4. 漏洞原理

#### 产生原因

核心原因有三个：

##### 1）服务端信任了客户端输入

`$_FILES['upload_file']['type']` 是请求的一部分。
请求由客户端发出，客户端可以完全控制。

浏览器正常上传时，它会根据文件扩展名或系统注册表猜一个 MIME，但攻击者不用浏览器。

##### 2）只校验 MIME，不校验扩展名和文件内容

这段代码没有检查：

- 文件扩展名是不是 `.php`；
- 文件真实内容是不是图片；
- 文件头是不是 `FF D8 FF`、`89 50 4E 47` 等；
- 是否重命名；
- 上传目录是否可执行。

所以只要 MIME 过关，什么文件都能传

##### 3）保存原始文件名

```php
$img_path = UPLOAD_PATH . '/' . $_FILES['upload_file']['name'];
```

直接使用用户文件名，导致攻击者可以上传 `shell.php`。

如果这里强制重命名为随机 `.jpg`，即使上传了 PHP 代码，也很难直接执行。

另外，文件名直接拼接还存在潜在风险：

- 路径穿越：`../../shell.php`
- 特殊字符
- 空字节截断（老版本 PHP）
- Windows 文件名特性

### 5. 防御手段

#### 1）扩展名白名单

```php
$allow_ext = ['jpg', 'jpeg', 'png', 'gif'];
$ext = strtolower(pathinfo($_FILES['upload_file']['name'], PATHINFO_EXTENSION));

if (!in_array($ext, $allow_ext, true)) {
    die('扩展名不允许');
}
```

一定要用白名单，不要用黑名单。

#### 2）检查真实 MIME

不要用 `$_FILES['upload_file']['type']`。

用 `finfo_file` 读取临时文件的真实 MIME：

```php
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mime = finfo_file($finfo, $_FILES['upload_file']['tmp_name']);
finfo_close($finfo);

$allow_mime = ['image/jpeg', 'image/png', 'image/gif'];
if (!in_array($mime, $allow_mime, true)) {
    die('真实 MIME 不允许');
}
```

#### 3）检查图片有效性

```php
$info = getimagesize($_FILES['upload_file']['tmp_name']);
if ($info === false) {
    die('不是有效图片');
}
```

这能挡住很多“改 MIME 但内容不是图片”的文件。

## Pass-3
