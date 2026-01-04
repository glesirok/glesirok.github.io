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

## 1. 关于testing
`go test` 会在运行测试函数之前自动调用 `flag.Parse()`，可以直接在测试中使用自定义 flag，无需手动解析。

参考文献：
[Using Go Flags in Tests](https://blog.jbowen.dev/2019/08/using-go-flags-in-tests/)
[go232806](https://golang.org/cl/232806)


## 2. 交叉编译

| 设置          | 作用             |
|---------------|------------------|
| GOOS=linux    | 目标平台是 Linux |
| CGO_ENABLED=1 | 启用 C 代码编译  |

纯 Go 代码 → Windows 上可以交叉编译到任何平台

用了 CGO  → 需要目标平台的 C 编译器


在windows下，编译linux平台的可执行程序时，有时候需要开启CGO.

这样需要一个能编译 Linux C 代码的编译器，而MinGW64无法做到，MinGW64是用于编译C代码，出来的可执行程序是windows平台的.

现在查到的资料是使用 zig

解决方案:
```bash
CGO_ENABLED=1 GOOS=linux GOARCH=amd64 CC="zig cc -target x86_64-linux-gnu" CXX="zig c++ -t arget x86_64-linux-gnu" go build -o xxx  ./path/to/main_go_dir
```



ps: 有可能你会搜到musl.cc，但好像windows平台用不了.
musl 是一个轻量级的 C 标准库，替代 glibc（GNU C Library）.
musl.cc 网站提供基于 musl 的预编译 GCC 交叉编译工具链.

### 2.1 ZIG

Zig 是一门编程语言，内置跨平台 C/C++ 编译器（基于 LLVM/Clang）


使用zig cc即可编译C/C++代码，解决c/c++的跨平台编译问题：
```bash
zig cc -target x86_64-linux-gnu    # 编译 Linux
zig cc -target aarch64-macos       # 编译 macOS ARM
zig cc -target x86_64-windows      # 编译 Windows
```

下载好ZIG后，添加环境变量.

### 2.1.2 为什么c是跨平台的语言，但在编译目标平台程序有各种问题呢？

其中，需要理解的概念就是：

C 语言：源代码跨平台（同一份 .c 文件可以在任何平台编译）

GCC：每个 GCC 可执行文件绑定一个目标平台

MinGW = Minimalist GNU for Windows  MinGW 就是把 GNU 工具链移植到 Windows，让 Windows 用户能用熟悉的 GCC 生态开发 Windows 程序。 

想生成 Linux 程序，需要另一套"目标=Linux"的 GCC.