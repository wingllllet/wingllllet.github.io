---
title: "部署ai-Goat靶场"
date: 2026-08-15T19:34:28+08:00
image: https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20260816163839279.jpg
---

## 前言

​	好久没有在电脑上做事情了，这次部署ai-goat让我重温大学期间做事的感觉了。就是明明给出的方法非常简单，可能就几行命令，但是由于该项目比较老或者自己电脑缺少某些环境导致无法一键解决，需要自己不断解决出现的问题，仔细一想跟打游戏一样，要达成好结局要不断的过关。

## ai-goat介绍

​	**官方介绍：**AI Goat 使用 Vicuna LLM，它源自 Meta 的 LLaMA，并结合了 ChatGPT 的响应数据。安装 AI Goat 时，LLM 二进制文件会从 HuggingFace 下载到您的本地计算机。这个大约 8GB 的二进制文件是所有挑战的核心 AI 引擎。LLM 二进制文件本质上接收一个输入“提示”，并输出一个输出“响应”。提示由三个元素组成，它们连接成一个字符串。这三个元素分别是：1. 指令；2. 问题；3. 响应。指令元素包含 LLM 的规则描述，用于告诉 LLM 它的运行方式。问题元素是大多数系统允许用户输入的地方。例如，在聊天引擎中输入的评论就放在问题元素中。最后，响应部分规定了 LLM 对问题的响应。

​	预构建的 Docker 镜像 ai-base 包含了运行 LLM 和挑战所需的所有库。该容器会在安装过程中与 LLM 二进制文件一起下载。每个挑战都对应一个 docker-compose 文件，该文件会附加 LLM 二进制文件、特定的挑战文件，并暴露完成每个挑战所需的 TCP 端口。有关入门说明，请参阅安装和设置部分。

​	我们准备了一个可选的 CTFd 容器，其中包含每个挑战的描述、提示、类别和 flag 提交方式。该容器镜像托管在我们的 Docker Hub 上，与 ai-base 镜像一起命名为 ai-ctfd。您可以从 ai-goat.py 启动 ai-ctfd 容器，并使用浏览器访问它。

​	**项目地址：**https://github.com/dhammon/ai-goat

## 安装

​	这位作者给出的方式是输入这些命令

```shell
git clone https://github.com/dhammon/ai-goat
cd ai-goat
pip3 install -r requirements.txt
chmod +x ai-goat.py
./ai-goat.py --install
```

​	看着很简单吧，可以每一行命令都有其要解决的问题。

- `git clone https://github.com/dhammon/ai-goat`

​	在输入命令时可能会遇到无法访问`github`的情况。这种情况可以在网页上登录`github`然后直接下载该项目的压缩包

- `pip3 install -r requirements.txt`

  我是想在`kali-linux`中部署该项目，但是 Kali Linux 的 **PEP 668 保护机制**，禁止直接向系统 Python 安装 pip 包。所以需要用虚拟环境解决：

  ```shell
  cd ai-goat
  
  # 1. 创建虚拟环境
  python3 -m venv venv
  
  # 2. 激活虚拟环境
  source venv/bin/activate
  
  # 3. 安装依赖
  pip install -r requirements.txt
  ```

  不过在以后要运行`ai-goat.py`前需要激活虚拟环境

  ```shell
  source venv/bin/activate
  ```

- `./ai-goat.py --install`

​	这个命令执行后，会先下载某个文件，然后在进行docker镜像部署。但是在执行命令时大概率访问不到这个网站。所以仍需要自己去下载。这就需要自己去审代码，在`app/config.py`文件中会给出下载地址。

```python
class Config:
    MODEL_PATH = "app/models/ggml-old-vic13b-q4_0.bin"
    MODEL_URL = "https://huggingface.co/vicuna/ggml-vicuna-13b-1.1/resolve/main/ggml-old-vic13b-q4_0.bin"
    MODEL_MD5 = "dfe5d1e30242d63a72d06f7bee5edf80"
    AI_IMAGES = [
        "rootcauz/ai-base:0.0.1",
        "rootcauz/ai-ctfd:0.0.2"
    ]
```

​	访问该链接即可下载文件，之后需要把这个文件放到指定的位置`MODEL_PATH = "app/models/ggml-old-vic13b-q4_0.bin"`，之后

执行安装命令

```
./ai-goat.py --install
```

​	就不会去下载而是直接执行`docker`拉取镜像步骤，你以为这样就完了？这远没有结束。

- 使用docker命令却无法拉镜像

​	**Debian/Kali** 近年来**优先推荐 Podman**，并且仓库中有一个 `podman-docker` 包，安装后会在 `/usr/bin/docker` 放一个软链接或脚本，所以实际调用的是 `podman`命令，`docker`命令就会被`podman`代替。但是即使使用`podman`命令也可能无法拉取镜像，大概率是无法直连`docker Hub`，所以需要配置`podman`的镜像加速，只需要输入命令：

```shell
sudo tee /etc/containers/registries.conf <<-'EOF'
unqualified-search-registries = ["docker.io"]

[[registry]]
prefix = "docker.io"
location = "docker.io"

[[registry.mirror]]
location = "docker.xuanyuan.me"

[[registry.mirror]]
location = "docker.1ms.run"

[[registry.mirror]]
location = "docker.m.daocloud.io"
EOF
```

- 使用podman在创建容器

​	`podman`命令和`docker`基本一样，只是把`docker`换成了`podman`

```shell
# 将所有镜像都创建容器
podman compose up -d
# 指定创建容器
podman compose up [ID]
# 查看所有容器信息
podman ps -a
# 打印容器所有信息
podman logs [ID]
```

## 呈现

​	当真正部署成功后，就可以访问相应端口。这个是CTFD页面

![image-20260816160802204](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20260816160809378.png)

​	这个就是第二个挑战的挑战页面，不过我目前还不知道怎么互动。

​	![image-20260816160837180](https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20260816160837293.png)

## 结语

​	`ai-goat`项目就部署到此了，我看了下只有两个训练靶场，并且我还不知道怎么互动，网上也没找到相应`wp`，所以就到这里了。不过看起来我好像什么也没做到，但还是有不少收获的。毕竟学习就是这样不断发现问题，再寻找解决问题的方法，最终解决问题达成目标。结合最近刚看完《学习观》，我会把问题和AI联系起来，在出现问题时先询问`AI`，从这里先获取方法。kali在无法访问网站或下载文件的时候先想到它是配置镜像源的问题，也了解现在kali基本使用`podman`命令来部署镜像和容器。后续我也可能写一篇《学习观》的笔记，这本书专业性比较高需要多看几遍。

