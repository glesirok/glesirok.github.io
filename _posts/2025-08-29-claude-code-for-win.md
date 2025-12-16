---
title: Claude Code使用教程 for win
date: 2025-08-29 08:35 +0800
categories:
  - IT
  - Tutorial
tags:
  - Claude
comments: true
---

## 1 安装

要求nodejs 版本18+
以及git bash
``` bash
npm install -g @anthropic-ai/claude-code
```

查看版本
```bash
npm list -g @anthropic-ai/claude-code
```



列出所有可安装版本

```bash
npm view @anthropic-ai/claude-code versions
```

查看最新版本

```bash
npm view @anthropic-ai/claude-code version
```


## 2 配置

### 2.1 配置代理
需要配置一下环境变量:  
1.打开cmd/powershell查看自己的用户目录在哪:  
cmd可以这样查看:

`echo %USERPROFILE%`

然后在里面找到`.claude`文件夹,  
在`.claude`里面创建`settings.json`  
**端口按照自己的代理端口更改**

``` json
{
  "env": {
    "HTTP_PROXY": "http://127.0.0.1:7890",  
    "HTTPS_PROXY": "http://127.0.0.1:7890"
  }
}
```

### 2.2 配置API接口（可选）
在`.claude`里面创建`settings.json`

``` json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "sk-",  
    "ANTHROPIC_BASE_URL": "http://XXX.XXX"
  }
}
```


## 3 使用

在powershell里运行(无法再git bash里运行这个命令，这会使claude 找不到git-bash)：
```
claude
```
输出：
![](assets/posts/20250820/2025-08-1755652603.png)
按上下键选择文本风格，
回车确认，输出：
![](assets/posts/20250820/2025-08-20-Claude-co1755652988.png)
有两个选择：
1. 使用claude订阅账号
2. API付费账号
我们这里选择2. API付费账号
会弹出窗口，登录账号后填授权码就行

## 4 查看使用量

npm先安装ccusage：

```css
npm install -g ccusage
```

然后直接ccusage就可以查看了：

``` bash
ccusage
```