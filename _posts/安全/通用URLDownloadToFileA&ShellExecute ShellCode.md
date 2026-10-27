---
title: 通用URLDownloadToFileA&ShellExecute ShellCode
date: 2026-10-15
author: medasz
toc: "true"
tags:
categories: 安全
---
[PE格式资源网站](https://shell-storm.org/formats/pe.html)
![](/images/ascii.png)
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
### PE格式
<iframe src="/images/PEFormat.pdf" width="100%" height="800" style="border: 0;"></iframe>

### TEB
![](/images/867232_UQX698BZWQ7UZ8Y.jpg)
x86中获取kernel32基址：

![](/images/kernel32基址的获得方法.png)
PE文件格式：
![](/images/pe.png)通过dll基址动态寻找函数地址
![](/images/通过基址动态寻找函数地址.png)
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
`SizeOfOptionalHeader`:紧跟在本结构体之后的`IMAGE_OPTIONAL_HEADER32`结构体的大小，即`sizeof(IMAGE_OPTIONAL_HEADER32)`,应该是`00E0h`
`Characteristics`:文件属性标志字段，对于`普通的可执行`文件此字段值应为`010Fh`,对于`dll文件`，此字段值应为`210Eh`。
### IMAGE_OPTIONAL_HEADER32结构体
可选映像头
```
//0xe0 bytes (sizeof)
struct _IMAGE_OPTIONAL_HEADER
{
    USHORT Magic;                                       //0x0       标志符
    UCHAR MajorLinkerVersion;                           //0x2       链接器主版本号
    UCHAR MinorLinkerVersion;                           //0x3       连接器次版本号
    ULONG SizeOfCode;                                   //0x4       所有含代码区块的总大小
    ULONG SizeOfInitializedData;                        //0x8       所有初始化数据区块的总大小
    ULONG SizeOfUninitializedData;                      //0xc       所有未初始化数据区块的总大小
    ULONG AddressOfEntryPoint;                          //0x10      程序执行入口RVA
    ULONG BaseOfCode;                                   //0x14      代码区块起始RVA
    ULONG BaseOfData;                                   //0x18      数据区块起始RVA
    ULONG ImageBase;                                    //0x1c      映像基址
    ULONG SectionAlignment;                             //0x20      内存中区块对齐值
    ULONG FileAlignment;                                //0x24      文件中区块对齐值
    USHORT MajorOperatingSystemVersion;                 //0x28      操作系统主版本号
    USHORT MinorOperatingSystemVersion;                 //0x2a      操作系统次版本号
    USHORT MajorImageVersion;                           //0x2c      用户自定义主版本号
    USHORT MinorImageVersion;                           //0x2e      用户自定义次版本号
    USHORT MajorSubsystemVersion;                       //0x30      所需要子系统主版本号
    USHORT MinorSubsystemVersion;                       //0x32      所需要子系统次版本号
    ULONG Win32VersionValue;                            //0x34      保留，设为0
    ULONG SizeOfImage;                                  //0x38      内存中的映像总大小
    ULONG SizeOfHeaders;                                //0x3c      区块表及其之前结构的总大小
    ULONG CheckSum;                                     //0x40      校验和
    USHORT Subsystem;                                   //0x44      文件子系统
    USHORT DllCharacteristics;                          //0x46      DLL特性标志
    ULONG SizeOfStackReserve;                           //0x48      初始化堆栈大小
    ULONG SizeOfStackCommit;                            //0x4c      初始化实际提交堆栈大小
    ULONG SizeOfHeapReserve;                            //0x50      初始化保留堆大小
    ULONG SizeOfHeapCommit;                             //0x54      实际提交堆大小
    ULONG LoaderFlags;                                  //0x58      调试相关，默认为0
    ULONG NumberOfRvaAndSizes;                          //0x5c      数据目录表的项数
    struct _IMAGE_DATA_DIRECTORY DataDirectory[16];     //0x60      数据目录表结构
};
```
#### 相对虚拟地址（RVA）
PE文件中的某个数据在装入内存后，相对于内存中文件头的偏移量。
`Magic`:一个标记字，`ROM映像`，则值为`0107h`;`普通可执行文件映像`值为`010Bh`。
`SizeOfCode`:带有IMAGE_SCN_CNT_CODE属性的区块的总大小，并且是经过字节对齐后的值。新增代码区块，这个值需要进行相应的更改。

注意：
```
VC6 的 `__asm` 同时支持两种十六进制写法：

push 0xF0F45516     ; C/C++ 风格
push 0F0F45516h     ; MASM 风格


它们表示同一个 32 位数。

区别在于 MASM 的 `h` 后缀写法有一个词法规则：如果第一位是 `A` 到 `F`，必须在前面补一个 `0`，否则汇编器会把它当作标识符或标签。

例如：


push 6D6C7275h      ; 正确，第一位是数字
push 213BCB58h      ; 正确，第一位是数字

push F0F45516h      ; 错误，被识别成标签 F0F45516h
push 0F0F45516h     ; 正确，前面补 0


而 `0x` 前缀已经明确说明后面是十六进制数，所以不需要额外补零：


push 0xF0F45516     ; 正确
push 0xFF32336C     ; 正确
```



需要用到的API：

| API | 动态链接库 |
| --- | ----- |
|     |       |
