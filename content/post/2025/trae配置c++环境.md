---
title: "Trae配置c++环境"
date: 2025-07-11T21:55:09+08:00
image: https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250713134437415.png
tags: ["C++"]
---

## 前言

​	最近想学C++做个小项目，原本想是用visual studio来写C++的，但是看到桌面有个trae还没怎么用过，而且后面我只想做些小项目，就打算用trae了，不好用再换。不过发现trae配置C++环境没有visual studio容易，所以写这篇文章来记录一下如何配置。

## 前期准备

### MSVC

- 在visual studio下载界面，下载**Visual Studio 2022 生成工具**。

![image-20250712210705038](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250712210705122.png)

- 下载后打开**Visual Studio Installer**，在**安装位置**可以修改安装路径，在**单个组件**中选择`Windows 1 SDK`和`MSVC v143 -VS 2022 C++ x64/x86 生成工具`。

![QQ20250711-222411](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250711222428017.png)

### CMake

- 官网：[Download CMake](https://cmake.org/download/)
- 选择合适的版本点击下载，之后双击安装。

![QQ20250711-222320](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250711222338002.png)

- 最后在终端输入`cmake --version`查看是否安装成功。

### Clang-Format

![QQ20250711-222508](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250711222546874.png)

![QQ20250711-222534](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250711222555371.png)

## 新建项目

- 新建一个项目文件夹`Project1`，再生成`main.cpp`和`CMakeLists.txt`文件，内容如下。

```c++
// main.cpp

#include <iostream>
using namespace std;

int main() {
    cout << "Hello, World!" << endl;
    return 0;
}
```

```cmake
# CMakeLists.txt

cmake_minimum_required(VERSION 4.0) # 该项目所需的CMake最低版本
project(Project1)                      # 项目名称

set(CMAKE_CXX_STANDARD 26)         # 使用的C++标准

add_executable(test main.cpp)      # 生成可执行文件: "test.exe"
```

- 用trae打开这个文件夹，接着CMake插件会自动弹出让你选择工具包，选择64位的。

![QQ20250712-214705](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250712222843686.png)

## 运行文件

- 后续运行文件就只需要点击左下角的按钮。

![wechat_2025-07-12_224603_680](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250712224616794.png)

![image-20250712224632621](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250712224632684.png)

## 补充

- 通常不会只运行一个文件，还有不同的源文件和头文件，所以可以对CMakeLists.txt添加以下内容。

```cmake
file(GLOB HEADERS "*.h")
file(GLOB SOURCES "*.cpp")

add_executable(my_program ${SOURCES} ${HEADERS})
```

## 参考文章

- [Windows11上配置C、C++环境](http://trrw.tech/p/windows11%E4%B8%8A%E9%85%8D%E7%BD%AEcc-%E7%8E%AF%E5%A2%83/#cmake)

## 可能遇到的问题

### 问题一

![image-20250712222952439](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250712222952509.png)



- 这个问题可能是vscode没有识别到CMake的安装路径，需要在setting.json加入以下内容。

```json
{
    "cmake.cmakePath": "D:/CMake/bin/cmake.exe",
}
```

### 问题二

- 在编译代码时可能出现中文字符乱码的情况，这个可能是因为编码不同一导致的，CMake默认使用GBK编码，所以需要在trae界面右下角设置编码格式。


![image-20250716141719732](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250716141726847.png)
