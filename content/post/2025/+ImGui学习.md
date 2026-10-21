---
title: "ImGui学习"
date: 2025-07-20T17:07:47+08:00
draft: true
---

## 前言

怎么说呢。想学着做游戏外挂了，不过不是那些联网游戏，毕竟没那个水平QAQ，准备做一些单机小游戏的外挂，虽然有CE这样的神器，但是每次查找数据修改数据还是过于繁琐了，所以想做个一键式的外挂插件。~~当然还有别的原因~~

本笔记通过学习

## 初识Imgui

- 下载方式后面在讲，现在先看看Imgui是个什么样子。
- 现在展示的是Imgui-doking版，里面是有许多样式模板的，要想运行它的样式模版需要进入`examples`文件夹，然后双击`imgui_examples.sln`文件。

![wechat_2025-07-21_224143_507](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250721224338717.png)

- 可以看到`examples`文件夹里面有许多样式模板，但是你是无法运行的，需要双击`imgui_examlpes.sln`后才能运行，不然可能整了半天总是报错都不知道问题在哪，血的教训。
- 双击之后，如果你有visual studio 就会自动打开它（没有的话不知道会怎样），然后就会资源管理器里面就会出现Imgui的各种样式模版。

![](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250721224830737.png)

- 这里演示`example_win32_directx11`，后续也会用这个模板进行学习。接下来运行它的`main.cpp`文件。

![wechat_2025-07-21_225314_507](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250721225327223.png)

- 这个就是`example_win32_directx11`的样式模板的样子了。

## 环境部署

### 下载imgui源码

- **下载地址：**https://github.com/ocornut/imgui

- 选择doking和master版下载。

![image-20250726201406482](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250726201413587.png)

### 下载directx sdk

- **下载地址：**[Download DirectX Software Development Kit from Official Microsoft Download Center](https://www.microsoft.com/en-us/download/details.aspx?id=6812)
- 之所以安装`directx sdk`，主要是为了调用它的一些库。

- 安装过程中可能出现下面的问题

![image-20250726201810935](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250726201811006.png)

- 这是因为电脑中安装了一个`Microsoft Visual C++ 2010 Redistributable`，且版本比这个高，所以这个安装不了。

- 那么只要删除`Microsoft Visual C++ 2010 Redistributable`，再重新安装`Directx SDK`即可。

![image-20250726202021346](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250726202021402.png)

- 出现这个界面就表示安装成功了。

![image-20250726202119680](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250726202119739.png)

## Imgui初始化

- 打开`visual studio`，创建一个空项目，并起个名字，之后就以`imgui_demos`来表示这个项目了。

![image-20250726202541498](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250726202541561.png)

- 接着新建`mian.cpp`和`Auto.h`文件，先不着急往里面写东西。

![image-20250726202747185](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250726202747241.png)

- 在`imgui_demos`文件夹中新建一个`Imgui`文件夹，再打开`imgui-doking`文件夹，将根目录下的以及`backends`文件夹中的这几个文件复制进`Imgui`中。



![image-20250727095159547](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250727095159613.png)

![image-20250726203448928](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250726203448983.png)

![image-20250726203522511](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250726203522559.png)

- 接着在`visual studio`中新建筛选器，并将这些文件拖进去。
- 之后在`Auto.h`文件中把刚拖过去的头文件全部引用，并加上`d3d10.h`。

```c++
### Auto.h

#include "Imgui/imconfig.h"
#include "Imgui/imgui.h"
#include "Imgui/imgui_impl_dx11.h"
#include "Imgui/imgui_impl_win32.h"
#include "Imgui/imgui_internal.h"
#include "Imgui/imstb_rectpack.h"
#include "Imgui/imstb_textedit.h"
#include "Imgui/imstb_truetype.h"
#include <d3d11.h>
#pragma comment(lib, "d3d11.lib")
```

- 在`imgui-docking\examples\example_win32_directx11`中把`main.cpp`中的代码复制到自己的项目`main.cpp`中，接着可以运行一下代码，能出现界面表示代码引用成功。

![wechat_2025-07-21_225314_507](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250721225327223.png)

## Imgui代码简要解析

### Imgui窗口

```c++
    // Show the window
	::ShowWindow(hwnd, SW_SHOWDEFAULT);        // Hide the window initially to avoid flickering
    ::UpdateWindow(hwnd);
```

- 这块可以设置imgui窗口展现模式。通过之前的demo演示可以发现菜单栏一直在一个窗口里面，那么如果想要只展示菜单栏，就需要把背景窗口隐藏起来。那么此代码就可以修改为

```c++
    // Show the window
	::ShowWindow(hwnd, SW_HIDE);        // Hide the window initially to avoid flickering
    ::UpdateWindow(hwnd);
```

- 但是这种方式只是把背景窗口隐藏起来，它依然会限制菜单栏的移动位置，这时就需要开启浮窗跳出模式。

```c++
	io.ConfigViewportsNoAutoMerge = true;    //浮窗跳出，依赖窗口
```

- 如果复制的是imgui-doking的代码，这句代码就在里面只不过被注释了。

### Imgui主题风格

```c++
    // Setup Dear ImGui style   更新imgui风格
    //ImGui::StyleColorsDark();
    //ImGui::StyleColorsLight();
	ImGui::StyleColorsClassic();
```

- 这里是更新Imgui的主题风格，在imgui中内置了三种主题风格。有时作者会在主文件中只展示了其中几种。比如

```c++
    // Setup Dear ImGui style   更新imgui风格
    //ImGui::StyleColorsDark();
    ImGui::StyleColorsLight();
```

- 如果想找作者其他内置的功能，可以右键该函数然后选择`转到声明`，因为一般作者都会把功能相同的函数放在一块。这样就能找到其他内置的函数。

![image-20250802125851384](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250802125858486.png)

![image-20250802125919326](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250802125919412.png)

- 当然处理使用作者内置的主题风格，也可以自己设计。如果不知道作者是怎么设计的，可以右键函数选择`转到定义`。

![image-20250802130134015](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250802130134104.png)

- 之后就可以根据自己的想法重新设计一个主题风格函数。

### 创建窗口

#### Begin()函数

```c++
 ImGui::Begin("Hello, winglet!");       // IMGUI创建开始  
```

- `Begin()`函数就是创建Imgui的开始，我们先看看`Begin()`函数能传那些参数。

```c++
bool ImGui::Begin(const char* name, bool* p_open, ImGuiWindowFlags flags)
```

- `name`参数就是标题栏展示的名称。

![image-20250802131105523](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250802131105588.png)

- `p_open`参数是个`bool`类型，它的功能就是是否出现关闭按钮。

![image-20250802131220760](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250802131220844.png)

- `flag`标志位是选择Imgui窗口的功能。那么具体有哪些功能呢？右键类型`ImGuiWindowFlags`选择`转到定义`。

![image-20250802133639267](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250802133639840.png)

```c++
typedef: 为已存在的类型重新定义一个名称。
	例： typedef int a;
		a dd = 1;
	这样a就可以代表int类型
enum: 枚举类型。如：00000001, 00000010, 00000100。 
```

- 这句要看注释指定的是`ImGuiWindowFlags_`，所以`ctrl+F`搜索这个词。

![image-20250802134155976](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/20250802134156080.png)

- 接着就可以根据注释给出的功能解释选择你想要的功能。方式如下

```c++
      int show_flags = ImGuiWindowFlags_NoResize;

      if (open)
      {
          ImGui::Begin("Hello Winglet", &open, show_flags);   //窗口绘制开始)
          ImGui::End();       //窗口绘制结束
      }
      else break;
```

- 如果想选择的功能不止一个，需要使用`|`符号。

```c++
	  int show_flags = ImGuiWindowFlags_NoResize | ImGuiWindowFlags_NoMove;
```

#### 窗口属性





## 附录

### ImGuiWindowFlags

```c++
ImGuiWindowFlags_None = 0, // 无标志
ImGuiWindowFlags_NoTitleBar = 1 << 0, // 禁用标题栏
ImGuiWindowFlags_NoResize = 1 << 1, // 禁用用户通过右下角手柄调整窗口大小
ImGuiWindowFlags_NoMove = 1 << 2, // 禁用用户移动窗口
ImGuiWindowFlags_NoScrollbar = 1 << 3, // 禁用滚动条（窗口仍可通过鼠标或程序控制滚动）
ImGuiWindowFlags_NoScrollWithMouse = 1 << 4, // 禁用用户通过鼠标滚轮垂直滚动。在子窗口中，除非同时设置了NoScrollbar，否则鼠标滚轮事件会传递给父窗口
ImGuiWindowFlags_NoCollapse = 1 << 5, // 禁用用户通过双击窗口折叠。也称为窗口菜单按钮（如在停靠节点中）
ImGuiWindowFlags_AlwaysAutoResize = 1 << 6, // 每帧自动调整窗口大小以适应内容
ImGuiWindowFlags_NoBackground = 1 << 7, // 禁用绘制背景色（WindowBg等）和外边框。类似于使用SetNextWindowBgAlpha(0.0f)
ImGuiWindowFlags_NoSavedSettings = 1 << 8, // 从不加载/保存.ini文件中的设置
ImGuiWindowFlags_NoMouseInputs = 1 << 9, // 禁用捕获鼠标，悬停测试直接穿透
ImGuiWindowFlags_MenuBar = 1 << 10, // 包含菜单栏
ImGuiWindowFlags_HorizontalScrollbar = 1 << 11, // 允许显示水平滚动条（默认关闭）。可在调用Begin()前使用SetNextWindowContentSize(ImVec2(width,0.0f))指定宽度。参见imgui_demo中"水平滚动"部分的代码
ImGuiWindowFlags_NoFocusOnAppearing = 1 << 12, // 禁止窗口从隐藏变为可见状态时获取焦点
ImGuiWindowFlags_NoBringToFrontOnFocus = 1 << 13, // 禁止获取焦点时将窗口置于前台（如点击或编程方式给予焦点）
ImGuiWindowFlags_AlwaysVerticalScrollbar= 1 << 14, // 始终显示垂直滚动条（即使ContentSize.y < Size.y）
ImGuiWindowFlags_AlwaysHorizontalScrollbar=1<< 15, // 始终显示水平滚动条（即使ContentSize.x < Size.x）
ImGuiWindowFlags_NoNavInputs = 1 << 16, // 禁用窗口内的键盘/游戏手柄导航
ImGuiWindowFlags_NoNavFocus = 1 << 17, // 禁止键盘/游戏手柄导航聚焦到此窗口（如被CTRL+TAB跳过）
ImGuiWindowFlags_UnsavedDocument = 1 << 18, // 在标题旁显示圆点。在选项卡/停靠上下文中使用时，点击X不会立即关闭选项卡（等待用户停止提交），否则按下X会立即关闭
ImGuiWindowFlags_NoDocking = 1 << 19, // 禁用此窗口的停靠功能
ImGuiWindowFlags_NoNav = ImGuiWindowFlags_NoNavInputs | ImGuiWindowFlags_NoNavFocus, // 组合标志：禁用所有导航输入和聚焦
ImGuiWindowFlags_NoDecoration = ImGuiWindowFlags_NoTitleBar | ImGuiWindowFlags_NoResize | ImGuiWindowFlags_NoScrollbar | ImGuiWindowFlags_NoCollapse, // 组合标志：禁用所有装饰元素
ImGuiWindowFlags_NoInputs = ImGuiWindowFlags_NoMouseInputs | ImGuiWindowFlags_NoNavInputs | ImGuiWindowFlags_NoNavFocus, // 组合标志：禁用所有输入

// [内部使用]
ImGuiWindowFlags_DockNodeHost = 1 << 23, // 内部使用！供Begin()/NewFrame()使用
ImGuiWindowFlags_ChildWindow = 1 << 24, // 内部使用！供BeginChild()使用
ImGuiWindowFlags_Tooltip = 1 << 25, // 内部使用！供BeginTooltip()使用
ImGuiWindowFlags_Popup = 1 << 26, // 内部使用！供BeginPopup()使用
ImGuiWindowFlags_Modal = 1 << 27, // 内部使用！供BeginPopupModal()使用
ImGuiWindowFlags_ChildMenu = 1 << 28, // 内部使用！供BeginMenu()使用

// 已废弃的名称
#ifndef IMGUI_DISABLE_OBSOLETE_FUNCTIONS
ImGuiWindowFlags_NavFlattened = 1 << 29, // 已在1.90.9废弃：改用BeginChild()中的ImGuiChildFlags_NavFlattened
ImGuiWindowFlags_AlwaysUseWindowPadding = 1 << 30, // 已在1.90.0废弃：改用BeginChild()中的ImGuiChildFlags_AlwaysUseWindowPadding
#endif
```

