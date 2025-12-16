---
title: 在无管理员账户的情况下如何使用NVM for win
date: 2025-12-15 09:09 +0800
categories:
  - IT
tags:
  - Nodejs
  - NVM
comments: true
---
## 1 NVM无法切换Node的原因

NVM for windows（下面简称NVM）实现Node的切换的关键是使用 `mklink /D` 命令创建软连接，而这个操作是需要管理员权限的

解决方法：
1. 目前nvm作者启动了一个新的项目能覆盖解决此问题，但是！ **We don't have the time for this**     ×
2. 源码编译：将`mklink /D`命令改成一个不需要权限的命令    √

## 2 原理

`mklink /D` 有一个相似的命令 `mklink /J`，可以使用`mklink /J`替换`mklink /D` 

具体区别：
  /D - 目录符号链接 (Directory Symbolic Link)

  mklink /D LinkName TargetPath

  - 需要管理员权限（或开启开发者模式）
  - 支持相对路径和绝对路径
  - 可以指向网络路径（UNC 路径）
  - 删除链接时，目标目录不受影响
  - 存储的是目标路径字符串

  /J - 目录联接 (Junction)

  mklink /J LinkName TargetPath

  - 不需要管理员权限
  - 只支持绝对路径（相对路径会自动转换为绝对路径）
  - 不能指向网络路径，只能指向本地目录
  - 删除链接时，目标目录不受影响
  - 是 NTFS 的原生功能，兼容性更好

  简单来说，`/D`是`/J`的超集，如果我们用不到网络路径和相对路径的链接功能，那么没必要使用`/D`, 使用`/J`命令就好了


### 2.1 参考文献：
  https://github.com/coreybutler/nvm-windows/pull/1139
  https://github.com/coreybutler/nvm-windows/pull/1117

## 3 操作步骤

### 3.1 修改源码
  1. Clone源码
  ```bash
git clone git@github.com:coreybutler/nvm-windows.git
  ```

2. 创建自己的分支
```bash
git checkout -b noadmin 1.2.2
```

后面的tag可以自己选，使用`git tag -l`查看tag

3. 修改源码

a. 全局搜索`func elevatedRun(name string, arg ...string) (bool, error)`
注释里面的调用提权cmd的片段
``` go
func elevatedRun(name string, arg ...string) (bool, error) {

    ok, err := run("cmd", nil, append([]string{"/C", name}, arg...)...)
    // if err != nil {
    //  exe, _ := os.Executable()
    //  cmd := filepath.Join(filepath.Dir(exe), "elevate.cmd")
    //  ok, err = run(cmd, &env.root, append([]string{"cmd", "/C", name}, arg...)...)
    // }

    return ok, err

}
```

这里的原理是：
- 先尝试普通权限执行
- 如果失败，调用 elevate.cmd 提权再执行
- elevate.cmd调用 elevate.vbs 进行实际的提权

这两个脚本都在`nvm-windows/assets`路径下

  b. 全局搜索`mklink`
将`/D`修改成`/J`
  ```go
ok, err = elevatedRun("mklink", "/D", filepath.Clean(env.symlink), filepath.Join(env.root, "v"+version))
if err != nil {
    if strings.Contains(err.Error(), "not have sufficient privilege") || strings.Contains(strings.ToLower(err.Error()), "access is denied") {
        ok, err = elevatedRun("mklink", "/D", filepath.Clean(env.symlink), filepath.Join(env.root, "v"+version))
        ...
    } else if strings.Contains(err.Error(), "file already exists") {
       ...
}
  ```



### 3.2 手动编译

#### 3.2.1 原本项目编译分析

目前[nvm-windows](https://github.com/coreybutler/nvm-windows)（[883c0de](https://github.com/coreybutler/nvm-windows/commit/883c0dea378ba991c7dc38ea5a39e6d5fc56453f)）的master分支上的README上提到手动编译是通过build.bat，但是根本没有这个文件，推测为该文档过时，没有更新

根据当前项目的文件分析推测为使用**Deno构建**

**前提**：需要安装Deno、golang编译器

通过项目根目录下的build.js   Deno 脚本，在本地生成 Windows 安装程序（Installer）。

Deno是什么：Deno 是一个 JavaScript/TypeScript 运行时，由 Node.js 的原作者 Ryan Dahl 创建。

build.js会使用Inno Setup 编译器（无需安装，项目自带：`assets/buildtools/iscc.exe`），通过iscc.exe编译nvm.iss 生成安装程序

``` text
  nvm.iss (Inno Setup 脚本，定义安装程序行为)
      ↓
  build.js (Deno 脚本，替换版本号，调用 iscc.exe)
      ↓
  iscc.exe (Inno Setup 编译器，生成安装程序)
      ↓
  nvm-setup.exe (最终的安装程序)
```


而如果我们不需要安装程序，只要可执行文件的话，就直接用go build 出可执行文件就行。

根据代码分析，必需文件（运行时依赖）如下，安装程序只是执行了一些设置环境变量、注册表、创建 symlink、处理已存在的 Node.js 安装等操作
  
  | 文件          | 用途                         |
  |-------------|----------------------------|
  | nvm.exe     | 主程序                        |
  | elevate.cmd | 提权脚本，用于 nvm use 创建 symlink |
  | elevate.vbs | 提权脚本，被 elevate.cmd 调用      |


#### 3.2.2 手动编译步骤

而我们的目的是不使用需要管理员权限的`mklink /D`命令，所以不需要elevate.cmd、elevate.vbs 这两个文件，最终只需要nvm.exe可执行程序就行
  
编译命令：

```bash
cd nvm-windows/src   # 建议进入目录后编译，以免一些依赖无法识别
go build -o ../bin/nvm.exe -ldflags "-X main.NvmVersion=1.2.2-noadmin" nvm.go
```

  `ldflags -X`解释：
  
    代码中有var NvmVersion = ""
      -X main.NvmVersion=1.2.2 在编译时把这个值设置为 "1.2.2"。
      也就是说这个参数不用按照我这个值设置，你可以自定义

  这个变量在多处使用：

  | 场景          | 代码位置                                |
  |-------------|-------------------------------------|
  | nvm version | 第 253 行，输出版本号                       |
  | nvm help    | 第 1723 行，显示 "Running version 1.2.2" |
  | nvm debug   | 第 1590 行，显示版本信息                     |
  | 检查更新        | 第 1710 行，比较版本号                      |

#### 3.2.3 手动安装

参考官方文档
https://github.com/coreybutler/nvm-windows/wiki#manual-installation


1. 创建安装目录，放入已编译的nvm.exe
2. 设置环境变量 NVM_HOME 和 NVM_SYMLINK：
    NVM_HOME：nvm.exe的安装目录
    NVM_SYMLINK：安装目录\nodejs，
3. 添加 %NVM_HOME%;%NVM_SYMLINK% 到 PATH
4. 创建 settings.txt（安装目录）示例（examples/settings.txt）：
    root: 这是文件解压的安装目录
    path: 安装目录\nodejs
  