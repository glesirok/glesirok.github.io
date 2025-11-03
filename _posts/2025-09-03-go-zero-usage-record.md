---
title: Go-zero 使用记录
date: 2025-09-03 14:34 +0800
categories:
  - IT
  - Tutorial
tags:
  - Go-zero
comments: true
---

# 安装
```bash
go install github.com/zeromicro/go-zero/tools/goctl@latest  #手脚架，主要工具

goctl env check --install --verbose --force   # 安装protoc相关组件

```

## API jwt认证

通过以下声明开启：

```go
@server (  
jwt: Auth // 开启 jwt 认证  
)
```

go-zero 的 JWT 中间件会自动将 JWT claims 中的 key 存储到 context，通过context访问：

``` go
ctx.Value("userId")
```
