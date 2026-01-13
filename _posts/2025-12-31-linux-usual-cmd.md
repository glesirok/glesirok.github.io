---
title: Linux常用命令记录
date: 2025-12-31 09:06 +0800
categories:
  - Default
tags:
  - writing
comments: true
---

## 1. 进程相关的
### 1.1 kill

`pkill xxx` 可以按名称杀进程
`pkill -f /path/xxx` 匹配进程匹配完整路径，而不只是进程名


## 2. 网络相关的

### 2.1 域名解析


```bash
getent hosts example.com
```


getent  不是专门查 DNS 的工具，而是查询各种系统数据库的统一接口, 通过配置文件动态决定查询顺序和数据源.

流程：

```bash
  1. 用户执行：getent hosts example.com
     ↓
  2. getent 调用 glibc 函数：gethostbyname("example.com")
     ↓
  3. glibc 读取 /etc/nsswitch.conf
     发现 hosts: files dns
     ↓
  4. 加载 NSS 模块：
     - libnss_files.so（处理文件查询）
     - libnss_dns.so（处理 DNS 查询）
     ↓
  5. 按顺序执行查询：

     ┌─ 第一步：libnss_files.so
     │  → 打开 /etc/hosts
     │  → 逐行匹配 "example.com"
     │  → 如果找到，返回结果，结束 ✓
     │  → 如果没找到，继续下一个 ↓
     │
     └─ 第二步：libnss_dns.so
        → 读取 /etc/resolv.conf 获取 DNS 服务器
        → 构造 DNS 查询报文
        → 发送 UDP 包到 DNS 服务器（53 端口）
        → 解析响应报文
        → 返回结果 ✓
```