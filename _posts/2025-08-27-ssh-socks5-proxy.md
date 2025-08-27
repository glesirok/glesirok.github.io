---
title: SSH使用SOCKS5代理
date: 2025-08-27 15:36 +0800
categories:
  - IT
  - Tutorial
tags:
  - SSH
comments: true
---
# 1 场景
- 本机为Windows
- 当你本机网络无法直接与服务器连接，
- 本机有网络代理软件，但无法启用tun模式
- 能代理软件能启动SOCKS5代理

# 2 方法

## 2.1 下载ncat便携版

- 用途：ssh本身没有代理选项，需要通过ncat转发
- 下载链接：https://nmap.org/dist/ncat-portable-5.59BETA1.zip

解压后，将ncat.exe改名nc.exe（也可不改，后面的命令你需要将nc替换为ncat），再目录地址添加到环境变量

## 2.2 测试是否可用
```BASH
ssh -o ProxyCommand="nc --proxy-type socks4 --proxy 127.0.0.1:代理端口 %h %p" -t 服务器用户名@服务器ip -p 服务器ssh端口
```
1. 是否提示输入密码，提示后代表已经能与服务器建立ssh连接
2. 再查看代理日志，是否有你的服务器访问记录，有就代表网络经过代理，没有就上网查查ProxyCommand参数怎么调
注意：目前ncat版本5.59BETA1支持socks4，不支持socks5

## 2.3 配置ssh的config
目的：为了省去每次连接ssh时的参数——`-o ProxyCommand="nc --proxy-type socks4 --proxy 127.0.0.1:代理端口 %h %p"`

步骤：打开windows家目录的.ssh文件夹（~/.ssh），新建一个config文件，写入：
```bash
Host 服务器ip
    HostName 服务器ip
	Port 服务器端口
	ProxyCommand nc --proxy-type socks4 --proxy 127.0.0.1:代理端口 %h %p
    
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

## 2.4 最后测试

```bash
ssh -t 服务器用户名@服务器ip -p 服务器ssh端口
``` 

发现和2.2效果一样那么就完结撒花✿✿ヽ(°▽°)ノ✿