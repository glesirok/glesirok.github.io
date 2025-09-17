---
title: windows startup program in background
date: 2025-09-17 08:49 +0800
categories:
  - IT
  - Powershell
tags:
  - win
comments: true
---
## win后台启动命令
原因：直接用cmd或者Powershell启动程序，会在关闭父cmd或者Powershell时，把启动的程序也一起关闭

使用以下命令可以实现类似linux `nohup &`的效果：
``` Powershell
powershell -Command "Start-Process '程序名.exe' -RedirectStandardOutput 'output.txt' -RedirectStandardError 'error.txt' -WindowStyle Hidden"   # 该命令，不支持追加输出，会把之前的输出与错误输出文件给覆盖掉

powershell -Command "Start-Process 'cmd' -ArgumentList '/c', '程序名.exe  >> output.txt 2>> error.txt' -WindowStyle Hidden"    #  Powershell 通过 cmd 间接启动目标程序，使用 >> 追加写入
```
