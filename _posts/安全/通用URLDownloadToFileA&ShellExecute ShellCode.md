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
    USHORT e_magic;                  //0x0        可执行文件标记“MZ”
    USHORT e_cblp;                   //0x2        文件最后页的字节数
    USHORT e_cp;                     //0x4        文件页数
    USHORT e_crlc;                   //0x6        重定义元素个数
    USHORT e_cparhdr;                //0x8        头部尺寸，以段落为单位
    USHORT e_minalloc;               //0xa        所需的最小附加段
    USHORT e_maxalloc;               //0xc        所需的最大附加段
    USHORT e_ss;                     //0xe        初始化堆栈段
    USHORT e_sp;                     //0x10       初始化堆栈指针
    USHORT e_csum;                   //0x12       校验和
    USHORT e_ip;                     //0x14       入口IP值
    USHORT e_cs;                     //0x16       入口CS值
    USHORT e_lfarlc;                 //0x18       重分配表文件地址
    USHORT e_ovno;                   //0x1a       覆盖号
    USHORT e_res[4];                 //0x1c       保留字
    USHORT e_oemid;                  //0x24       OEM标识符
    USHORT e_oeminfo;                //0x26       OEM信息
    USHORT e_res2[10];               //0x28       保留字
    LONG e_lfanew;                   //0x3c       PE文件头的相对偏移地址
};
```
PE文件头在winnt.h中由IMAGE_NT_HEADERS32结构体定义
```
//0xf8 bytes (sizeof)
struct _IMAGE_NT_HEADERS
{
    ULONG Signature;                               //0x0        PE文件标识
    struct _IMAGE_FILE_HEADER FileHeader;          //0x4        映像文件头
    struct _IMAGE_OPTIONAL_HEADER OptionalHeader;  //0x18       可选映像头
};
```
`Signature`默认是`0x00004550`，ASCII码是`PE00`
```
//0x14 bytes (sizeof)
struct _IMAGE_FILE_HEADER
{
    USHORT Machine;                      //0x0      运行平台
    USHORT NumberOfSections;             //0x2      文件的区块数
    ULONG TimeDateStamp;                 //0x4      文件创建日期和时间
    ULONG PointerToSymbolTable;          //0x8      符号表指针
    ULONG NumberOfSymbols;               //0xc      符号表中的符号个数
    USHORT SizeOfOptionalHeader;         //0x10     IMAGE_OPTIONAL_HEADER32结构大小
    USHORT Characteristics;              //0x12     文件属性
};
```
`Machine`：可执行文件的目标CPU类型。

| 机器类型            | 预定义标志                      | 十六进制值 |
| --------------- | -------------------------- | ----- |
| Intel i386      | IMAGE_FILE_MACHINE_I386    | 14CH  |
| Intel 64        | IMAGE_FILE_MACHINE_IA64    | 200H  |
| IBM Power PC    | IMAGE_FILE_MACHINE_POWERPC | 1F0H  |
| Dec Alpha AXP   | IMAGE_FILE_MACHINE_ALPHA   | 184H  |
| Dec Alpha AXP64 | IMAGE_FILE_MACHINE_ALPHA64 | 284H  |
`NumberOfSections`:文件中的区段数目，`区段表`紧跟在PE文件头后面。



需要用到的API：

| API | 动态链接库 |
| --- | ----- |
|     |       |
