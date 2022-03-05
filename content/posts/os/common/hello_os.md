---
title: "HelloOs"
date: 2022-03-05T09:58:26+08:00
draft: false
tags: ["os","note","汇编"]
categories: ["os"]
slug: /os/common/1
---

> Hello OS 是基于硬件，一个最小的操作系统内核

Hello OS 的引导流程如下：

![](https://img.zhengyua.cn/img/20220305093741.png)


> BIOS 固件是固化在 PC 机主板上的 ROM 芯片中的，可掉电保存，它负责**检测和初始化 CPU、内存及主板平台**。
> PC 机上电后的第一条指令就是 BIOS 固件中的，然后加载引导设备（大概率是硬盘）中的第一个扇区数据，到 0x7c00 地址开始的内存空间，再接着跳转到 0x7c00 处执行指令。


## 引导汇编部分

```armasm
MBT_HDR_FLAGS EQU 0x00010003
MBT_HDR_MAGIC EQU 0x1BADB002 ;多引导协议头魔数
MBT_HDR2_MAGIC EQU 0xe85250d6 ;第二版多引导协议头魔数
global _start ;导出_start符号
extern main ;导入外部的main函数符号
[section .start.text] ;定义.start.text代码节
[bits 32] ;汇编成32位代码
_start:
jmp _entry
ALIGN 8
mbt_hdr:
dd MBT_HDR_MAGIC
dd MBT_HDR_FLAGS
dd -(MBT_HDR_MAGIC+MBT_HDR_FLAGS)
dd mbt_hdr
dd _start
dd 0
dd 0
dd _entry
;以上是GRUB所需要的头
ALIGN 8
mbt2_hdr:
DD MBT_HDR2_MAGIC
DD 0
DD mbt2_hdr_end - mbt2_hdr
DD -(MBT_HDR2_MAGIC + 0 + (mbt2_hdr_end - mbt2_hdr))
DW 2, 0
DD 24
DD mbt2_hdr
DD _start
DD 0
DD 0
DW 3, 0
DD 12
DD _entry
DD 0
DW 0, 0
DD 8
mbt2_hdr_end:
;以上是GRUB2所需要的头
;包含两个头是为了同时兼容GRUB、GRUB2
```

上述汇编代码是**用汇编定义的 GRUB 的多引导协议头，即一定格式的数据**，通过实现该协议标准让 GRUB 能识别 Hello OS。

```armasm
ALIGN 8
_entry:
;关中断
cli
;关不可屏蔽中断
in al, 0x70
or al, 0x80
out 0x70,al
;重新加载GDT
lgdt [GDT_PTR]
jmp dword 0x8 :_32bits_mode
_32bits_mode:
```

上述汇编代码作用于**关掉中断，设定 CPU 的工作模式**。


```armasm
;下面初始化C语言可能会用到的寄存器
mov ax, 0x10
mov ds, ax
mov ss, ax
mov es, ax
mov fs, ax
mov gs, ax
xor eax,eax
xor ebx,ebx
xor ecx,ecx
xor edx,edx
xor edi,edi
xor esi,esi
xor ebp,ebp
xor esp,esp
;初始化栈，C语言需要栈才能工作
mov esp,0x9000
;调用C语言函数main
call main
;让CPU停止执行指令
halt_step:
halt
jmp halt_step
```

该部分汇编代码是**初始化 CPU 的寄存器和 C 语言的运行环境**。


```armasm
GDT_START:
knull_dsc: dq 0
kcode_dsc: dq 0x00cf9e000000ffff
kdata_dsc: dq 0x00cf92000000ffff
k16cd_dsc: dq 0x00009e000000ffff
k16da_dsc: dq 0x000092000000ffff
GDT_END:
GDT_PTR:
GDTLEN dw GDT_END-GDT_START-1
GDTBASE dd GDT_START
```

该部分汇编代码是服务 **CPU 工作模式所需要的数据**。

## Hello OS 的主函数

上述汇编代码调用的 main 函数，是由下面 C 语言代码**分别由 nasm 和 GCC 编译可链接模块，由 LD 连接器链接在一起，形成可执行程序文件**：

```c
#include "vgastr.h"
void main()
{
  printf("Hello OS!");
  return;
} 
```

## 控制计算机屏幕

屏幕上显示字符，就要编程操作显卡。

**显卡都会支持 VESA 的标准**，该标准下有两种工作模式（字符和图形），且为支持该标准还会**提供 VGABIOS 的固件程序**。

下图即显卡的字符模式工作细节，可以看到主要的特点是在于**会每两个字节（ASCII码和字符颜色值）对应一个字符**。

![](https://img.zhengyua.cn/img/20220305095902.png)

上述主函数中`printf`控制输出代码如下：

```c
void _strwrite(char* string)
{
  char* p_strdst = (char*)(0xb8000);//指向显存的开始地址
  while (*string)
  {
    *p_strdst = *string++;
    // 这里省略了字符颜色信息
    p_strdst += 2;
  }
  return;
}

void printf(char* fmt, ...)
{
  _strwrite(fmt);
  return;
}
```

## make

make 即工具程序，会读取makefile文件中的规则自动化构建软件。

makefile文件中的规则：

- 首先有一个或者多个构建目标称为`target`
- 目标后面紧跟着用于构建该目标所需要的文件
- 目标下面是构建该目标所需要的命令及参数
- 同时检查文件的依赖关系

demo如下：

```makefile
CC = gcc #定义一个宏CC 等于gcc
CFLAGS = -c #定义一个宏 CFLAGS 等于-c
OBJS_FILE = file.o file1.o file2.o file3.o file4.o #定义一个宏
.PHONY : all everything #定义两个伪目标all、everything
all:everything #伪目标all依赖于伪目标everything
everything :$(OBJS_FILE) #伪目标everything依赖于OBJS_FILE，而OBJS_FILE是宏会被
#替换成file.o file1.o file2.o file3.o file4.o
%.o : %.c
   $(CC) $(CFLAGS) -o $@ $<
```

- `#`后面为注释
- `:=`或`=`可定义宏，引用宏时要用`$(宏名)`，宏最终会在宏出现的地方替换成相应的字符串
- `PHONY` 表示定义伪目标（不代表一个真正的文件名），在执行 make 时可以指定这个目标来执行其所在规则定义的命令
- 伪目标可以依赖于另一个伪目标或者文件
- `%.o : %.c`，其中%表示通配符，表示所有以“.o”结尾的文件依赖于所有以“.c”结尾的文件

> 这里补充下上面 GCC 编译的过程，我们可以手动控制该流程来得到相应的中间文件：
> ![](https://img.zhengyua.cn/img/20220305102135.png)
> 
> `gcc HelloWorld.c -E -o  HelloWorld.i`预处理：加入头文件，替换宏
> 
> `gcc HelloWorld.c -S -c -o HelloWorld.s`编译：包含预处理，将 C 程序转换成汇编程序
> 
> `gcc HelloWorld.c -c -o HelloWorld.o`汇编：包含预处理和编译，将汇编程序转换成可链接的二进制程序
> 
> `gcc HelloWorld.c -o HelloWorld`链接：包含以上所有操作，将可链接的二进制程序和其它别的库链接在一起，形成可执行的程序文件


## 编译

![](https://img.zhengyua.cn/img/20220305100940.png)

## 安装 

**当`Hello OS.bin`生成之后，需要让 GRUB 能够发现才能在启动时被加载，该过程称之为安装。**

我们通过 GRUB 启动时的规整，即加载一个`grub.cfg`的文本文件，根据其中的内容执行相应的操作，其中一部分内容就是启动项。在启动时**可选择不同的启动项后根据选择的启动项信息来加载 OS 文件到内存**。

下面即 Hello OS 的启动项：

```PYTHON
menuentry 'HelloOS' {
     insmod part_msdos #GRUB加载分区模块识别分区
     insmod ext2 #GRUB加载ext文件系统模块识别ext文件系统
     set root='hd0,msdos4' #注意boot目录挂载的分区，
     multiboot2 /boot/HelloOS.bin #GRUB以multiboot2协议加载HelloOS.bin
     boot #GRUB启动HelloOS.bin
}
```

> 上面Boot目录挂载分区可通过`df /boot/`命令查询，并且需要由特殊 GRUB 的命名方式。
> eg:`“hd0,msdos4”`，代表sda4就是硬盘的第四个分区（硬件分区选择 MBR），hd0 表示第一块硬盘，结合起来就是第一块硬盘的第四个分区。

把上面启动项的代码追加到 Linux 机器上的`/boot/grub/grub.cfg`文件末尾，并且把上面生成的`Hello OS.bin`文件复制到`/boot/`目录下。

> 需要注意的是，Ubuntu或者其他操作系统进入 GRUB 引导界面可能也需要进行设置，比如 Ubuntu 默认就是关闭和隐藏的，需要修改`/etc/default/grub`配置才能够看到。

最后重启计算机，你就可以看到 Hello OS 的启动选项了。

![](https://img.zhengyua.cn/img/20220305114135.png)

![](https://img.zhengyua.cn/img/20220305114059.png)

## 参考

https://time.geekbang.org/column/intro/100078401?tab=catalog