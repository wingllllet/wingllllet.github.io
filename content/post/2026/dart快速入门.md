---
title: "Dart快速入门"
date: 2026-07-20T08:57:52+08:00
draft: true
categories: ["安卓开发"]
---

## 创建首个dart项目

### 前言

​	各类编程语言底层逻辑相通，掌握一门后再学习其他语言都能快速上手。我会以python为对照，在本文中简单梳理记录 Dart 语言的独有特性。

### 配置dart环境

- 如果之前配置过**Flutter SDK**（一般包含完整的**Dart SDK**），则无需单独安装**Dart SDK**

- 在终端输入` dart --version`，会出现相似输出

```
Dart SDK version: 3.12.2 (stable) (Tue Jun 9 01:11:39 2026 -0700) on "windows_x64"
```

### 创建dart项目

- 在同一终端中创建`dartpedia`存储项目的目录

```
mkdir dartpedia
cd dartpedia
```

- 运行以下命令，创建基本项目cli(用于命令行接口)

```
dart create cli
```

- 执行dart程序

```
cd cli
dart run
```

```
Building package executable... (1.0s)
Built cli:cli.
Hello world: 42!
```

### 更改第一个dart代码

- 在代码编辑器中`bin/`目录是可执行代码的所在。`cli.dart`是申请入口，里面会有`main`函数，所有dart程序都是从`main`函数开始执行。

- `bin/cli.dart`内容应当如此：

  ```dart
  import 'package:cli/cli.dart' as cli;
  
  void main(List<String> arguments) {
    print('Hello world: ${cli.calculate()}!');
  }
  ```

- 将打印的文字修改为`Hello,Dart!`

  ```dart
  // import 'package:cli/cli.dart' as cli;
  
  void main(List<String> arguments) {
    print('Hello Dart!');
  }
  ```

-  运行程序，可以使用代码编辑的运行快捷键`crtl+F5`或者在终端输入

  ```
  dart run
  ```

- 输出应当如下

  ```shell
  Building package executable...
  Built cli:cli.
  Hello, Dart!
  ```

## 添加交互功能

### 命令行参数传递

- 将代码更改为

  ```dart
  const version = '0.0.1';
  
  void main(List<String> arguments) {
    if (arguments.isEmpty) {
      print('Hello Dart!');
    } else {
      print('Dartpedia CLI version: $version');
    }
  }
  ```

  - **const：**命名常量
  - **arguments.isEmpty：**检测变量`arguments`是否为`[]`，`[]`为`True`
  - **$version：**该语法称为字符串插值。通过使用一个变量名前缀来执行一个字符串`$`标志，它可让你直接将变量的值嵌入。

- **测试`version`命令：**

  ```shell
  dart bin/cli.dart version
  ```

  输出结果如下

  ![image-20260722085703561](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20260722085710736.png)

  ```
  该命令原理如下：
  ---第一个参数 dart → 启动 Dart VM 的可执行文件（不是脚本参数）。
  ---第二个参数 bin/cli.dart → 要执行的脚本文件路径（也不是脚本参数）。
  ---第三个及之后 的所有参数，例如 versionw → 全部作为脚本参数，按顺序放入 List<String> arguments
  因此，arguments 此时就是一个长度为 1 的列表：['versionw']
  ```

#### 进阶

- **添加一个`printUsage`函数：**为了使输出更加易于使用,创建一个单独的函数来显示使用信息。将此功能放在您的外部和下方`main`函数。

  ```dart
  void printUsage() { // Add this new function
    print(
      "The following commands are valid: 'help', 'version', 'search <ARTICLE-TITLE>'"
    );
  }
  ```

-  **实施`help`命令并加以改进`main`：**现在,将`help`使用 an 命令`else if`声明并清理默认行为以调用`printUsage`函数。

  修改你的`main`函数如下所示:

  ```dart
  void main(List<String> arguments) {
    if (arguments.isEmpty || arguments.first == 'help') {
      printUsage(); // Change this from 'Hello, Dart!'
    } else if (arguments.first == 'version') {
      print('Dartpedia CLI version $version');
    } else {
      printUsage(); // Catch-all for any unrecognized command.
    }
  }
  ```

- 运行该程序

  ```
  dart bin/cli.dart help
  ```

  ![image-20260722092843790](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20260722092843926.png)

  ```
  dart bin/cli.dart
  ```

  ![image-20260722093006484](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20260722093006601.png)

### 实现搜索命令

- **集成`search`命令进入`main`：**首先,修改`main`函数在`cli/bin/cli.dart`到包括一个`else if`负责处理的分支`search`命令。目前,只需打印一条占位符消息即可。

  ```dart
  void main(List<String> arguments) {
    if (arguments.isEmpty || arguments.first == 'help') {
      printUsage();
    } else if (arguments.first == 'version') {
      print('Dartpedia CLI version $version');
    } else if (arguments.first == 'search') {
      // Add this new block:
      print('Search command recognized!');
    } else {
      printUsage();
    }
  }
  ```

  测试输出结果

  ```shell
  dart bin/cli.dart search
  ```

  ![image-20260722094317128](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20260722094317246.png)

- **定义`searchWikipedia`函数:** `search`命令最终将运行您应用程序的核心逻辑调用一个函数`searchWikipedia`。目前,已经`searchWikipedia`打印                        论点随之传递`search`命令。将这一新功能放在下方`main` 。

  ```dart
  // ... (your existing main function)
  
  void searchWikipedia(List<String>? arguments) { // Add this new function and add ? to arguments type
    print('searchWikipedia received arguments: $arguments');
  }
  
  // ... (your existing printUsage() function)
  
  ```

  - `List<String>? arguments`这意味着`arguments`清单本身可以`null`
  - `List<String> arguments`则是`arguments`不能为`null`，可以为空列表`[]`



