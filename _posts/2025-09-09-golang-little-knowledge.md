---
title: Golang 小知识
date: 2025-09-09 09:26 +0800
categories:
  - IT
  - Thinking
tags:
  - Golang
comments: true
---

# 关于testing
`go test` 会在运行测试函数之前自动调用 `flag.Parse()`，可以直接在测试中使用自定义 flag，无需手动解析。

参考文献：
[Using Go Flags in Tests](https://blog.jbowen.dev/2019/08/using-go-flags-in-tests/)
[go232806](https://golang.org/cl/232806)