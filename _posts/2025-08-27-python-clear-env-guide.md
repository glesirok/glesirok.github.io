---
title: Python干净开发环境指南
date: 2025-08-27 16:27 +0800
categories:
  - IT
  - Python
tags:
  - Python
comments: true
---
# pyenv



目的：管理python版本

[依赖](https://github.com/pyenv/pyenv/wiki#suggested-build-environment)：
```bash
# 以debain为例 其他系统可以去依赖链接寻找
sudo apt update; sudo apt install make build-essential libssl-dev zlib1g-dev \
libbz2-dev libreadline-dev libsqlite3-dev curl git \
libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
```

安装：
```bash
curl -fsSL https://pyenv.run | bash
```

![](assets/posts/20250827/2025-08-27-python-cl1756285793.png)
根据提示将输出的一些shell配置添加到对应文件。

完成后重启shell，不想重启可以：
```
source .bashrc
source .profile
```

检查是否安装正确：
```
pyenv doctor
```

![](assets/posts/20250827/2025-08-27-python-cl1756285983.png)
如果出现红字，会有提示教你怎么解决，底部还有解决问题的链接

## 安装python：
```bash
pyenv install -l  # 会给出所有可用版本的列表
pyenv install 3.10.4   # 安装
```

## 在 Python 版本之间切换

- `pyenv shell <version>` -- 仅针对当前 shell 会话选择使用的python版本
- `pyenv local <version>` -- 每次在当前目录使用的python版本
- `pyenv global <version>` -- 全局使用的python版本
# pipenv

目的：包管理器，且可以实现类似虚拟环境的功能，基于pip，pip更换后，是需要重新install pipenv的，一个pip对应一个pipenv
安装：
```bash
pip install pipenv
```
常用命令：
```

```
