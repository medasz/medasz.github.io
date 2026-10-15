---
title: 通用URLDownloadToFileA&ShellExecute ShellCode
date: 2026-10-15
author: medasz
toc: "true"
tags:
categories: 安全
---
x86架构中：
1. 栈是从`高地址`向`低地址`生长的，也就是`向下生长`。
2. 字符串的读取顺序：从`低地址`向`高地址`直到遇到`\0`。
3. 采用`小端序`：**数据的低位字节存放在内存的低地址，高位字节存放在高地址**。

masm编译asm，打印汇编对应机器码命令
```
ml.exe /c /coff test.asm
#test.obj是上一个命令执行之后的产物
link.exe /dump /disasm test.obj
```

需要用到的API：

| API | 动态链接库 |
| --- | ----- |
|     |       |
