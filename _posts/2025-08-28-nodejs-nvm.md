---
title: Nodejs和NVM
date: 2025-08-28 08:46 +0800
categories:
  - IT
  - Tutorial
tags:
  - NVM
  - Nodejs
comments: true
---
## 1 安装
可以不用直接安装nodejs，先安装nvm，用nvm安装nodejs方便版本控制

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```
可看[Install & Update Script](https://github.com/nvm-sh/nvm?tab=readme-ov-file#install--update-script)链接中的教程，安装方式可能会更新，之后的版本可能不是这样安装。

![](assets/posts/20250828/2025-08-28-nvm1756342197.png)
上面的脚本会自动添加到家目录的.bashrc里，检查一下，没有就手动添加。


## 2 概念
npm install 一个包后，会相应地修改package.json文件。


## 3 使用

### 3.1 NVM

```bash
nvm ls-remote     # 列出可下载的nodejs版本
nvm install 8.0.0  #安装node版本
nvm use 8.0.0      # 使用该版本,
nvm list      #列出使用的nodejs版本
nvm alias default v8.0.0   #设置默认版本

```

nvm for windows的use是会以系统级切换nodejs的

也就是说，在一个shell进行了nvm use 切换，会影响到其他shell正在使用的shell，但是已执行的nodejs进程不受影响

### 3.2 npm
```
npm view 包名 versions   #查看包的可用版本
npm view less@3.8.1 dependencies  #查看包的依赖
```

### 3.3 yarn
yarn是项目的包管理器，不依赖node_modules

## 4 依赖文件的版本符号

波浪符号（~）：他会更新到当前minor version（也就是中间的那位数字）中最新的版本。放到我们的例子中就是：body-parser:~1.15.2，这个库会去匹配更新到1.15.x的最新版本，如果出了一个新的版本为1.16.0，则不会自动升级。波浪符号是曾经npm安装时候的默认符号，现在已经变为了插入符号。

插入符号（^）：这个符号就显得非常的灵活了，他将会把当前库的版本更新到当前major version（也就是第一位数字）中最新的版本。放到我们的例子中就是：bluebird:^3.3.4，这个库会去匹配3.x.x中最新的版本，但是他不会自动更新到4.0.0。

## 5 常见问题

### 5.1 nodejs后的版本号的LTS后面的名称是什么？
那是版本代号，好像LTS版本都有