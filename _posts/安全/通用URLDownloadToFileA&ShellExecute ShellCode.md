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

![](/images/867232_UQX698BZWQ7UZ8Y.jpg)
x86中获取kernel32基址：
![](/images/kernel32基址的获得方法.png)
PE文件格式：
![](/images/pe.png)
DOS MZ Header的结构体定义如下：_IMAGE_DOS_HEADER
```
//0x40 bytes (sizeof)
struct _IMAGE_DOS_HEADER
{
    USHORT e_magic;                  //0x0
    USHORT e_cblp;                   //0x2
    USHORT e_cp;                     //0x4
    USHORT e_crlc;                   //0x6
    USHORT e_cparhdr;                //0x8
    USHORT e_minalloc;               //0xa
    USHORT e_maxalloc;               //0xc
    USHORT e_ss;                     //0xe
    USHORT e_sp;                     //0x10
    USHORT e_csum;                   //0x12
    USHORT e_ip;                     //0x14
    USHORT e_cs;                     //0x16
    USHORT e_lfarlc;                 //0x18
    USHORT e_ovno;                   //0x1a
    USHORT e_res[4];                 //0x1c
    USHORT e_oemid;                  //0x24
    USHORT e_oeminfo;                //0x26
    USHORT e_res2[10];               //0x28
    LONG e_lfanew;                   //0x3c
};
```
需要用到的API：

| API | 动态链接库 |
| --- | ----- |
|     |       |
