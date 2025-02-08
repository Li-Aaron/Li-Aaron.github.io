---
layout: post
title: ASL语言学习（5）- OVMF + QEMU 实战
date: 2025-02-08 19:30:00.000000000 +08:00
description: 本文介绍如何基于 Edk2 的 OvmfPkg 来调试 ASL 代码。
author: aaron-li
categories: [代码相关, BIOS]
tags: [asl]
---

续上四篇，这一篇介绍下如何基于 Edk2 的 OvmfPkg 来调试 ASL 代码。  
[ASL语言学习（1）- 基本语法，作用域]({{site.url}}/2023/04/asl-code-intro/)，  
[ASL语言学习（2）- Debug，常用操作符]({{site.url}}/2023/04/asl-code-intro2/)，  
[ASL语言学习（3）- 隐式转换，Method调用]({{site.url}}/2023/04/asl-code-intro3/)，  
[ASL语言学习（4）- OpRegion 实战]({{site.url}}/2023/04/asl-code-intro4/)  

Reference:  
[ACPI Source Language (ASL) Tutorial](https://acpica.org/sites/acpica/files/asl_tutorial_v20190625.pdf)  
[ACPI Source Language (ASL) Reference](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html)  
[Edk2 OvmfPkg](https://github.com/tianocore/edk2/tree/master/OvmfPkg)  
[Ubuntu Wiki - OVMF](https://wiki.ubuntu.com/UEFI/OVMF)  

# 构建 OvmfPkg 的 Bios Binary

首先我们构建Edk2的OvmfPkg。

如果完全不了解Edk2，可以先参考[Getting Started with EDK II](https://github.com/tianocore/tianocore.github.io/wiki/Getting-Started-with-EDK-II)。

这里简要介绍下基于WSL+GCC以及基于VS2022 Community的构建流程。

## 基于 WSL+GCC 构建

先贴一下我这个环境的信息

```text
Linux version 5.15.153.1-microsoft-standard-WSL2
Ubuntu 22.04.5 LTS
gcc version 11.4.0 (Ubuntu 11.4.0-1ubuntu1~22.04)
```

配置环境可以参考[Using EDK II with Native GCC](https://github.com/tianocore/tianocore.github.io/wiki/Using-EDK-II-with-Native-GCC#user-content-Ubuntu_2004_LTS)。

这里贴一下我安装的依赖

```bash
# note: iasl is replaced by acpica-tools
sudo apt install build-essential uuid-dev acpica-tools git nasm python-is-python3
```

在WSL中进入edk2的上一层，利用下面的命令构建（这一步可以参考[Edk2 OvmfPkg](https://github.com/tianocore/edk2/tree/master/OvmfPkg)  ）。

```bash
export WORKSPACE=$(pwd)
export PACKAGES_PATH=$WORKSPACE/edk2

cd edk2
make -C BaseTools/
. edksetup.sh

build -a X64 -t GCC5 -p OvmfPkg/OvmfPkgX64.dsc -D DEBUG_ON_SERIAL_PORT
```

构建好的 BIOS Binary 位于 `$WORKSPACE/Build/OvmfX64/DEBUG_GCC5/FV/OVMF.fd`

## 基于 VS2022 构建

先贴一下我这个环境的信息

```text
Microsoft Windows [版本 10.0.19045.5371]
cl:     用于 x64 的 Microsoft (R) C/C++ 优化编译器 19.42.34436 版
nmake:  Microsoft (R) 程序维护实用工具 14.42.34436.0 版
iasl:   ASL+ Optimizing Compiler/Disassembler version 20241212
nasm:   NASM version 2.16.01 compiled on Dec 21 2022
python: Python 3.10.13 | packaged by Anaconda, Inc.
```

VS2022使用的是community版本，必须安装的工作负载如下：

- C++ 桌面开发（Desktop development with C++）
- 使用 C++ 的 Windows 生成工具（MSVC v143）
- Windows 10 SDK 或 Windows 11 SDK
- Windows 通用 C 运行时
- C++ CMake 工具

其他可酌情选择。

在command prompt/powershell中进入edk2的上一层，利用下面的命令构建（这一步可以参考[Edk2 OvmfPkg](https://github.com/tianocore/edk2/tree/master/OvmfPkg)  ）。

```bash
set WORKSPACE=%CD%
set PACKAGES_PATH=%WORKSPACE%\edk2

cd edk2
edksetup.bat
nmake -f %BASE_TOOLS_PATH%\Makefile

build -a X64 -t VS2022 -p OvmfPkg\OvmfPkgX64.dsc -D DEBUG_ON_SERIAL_PORT
```

构建好的 BIOS Binary 位于 `%WORKSPACE%\Build\OvmfX64\DEBUG_VS2022\FV\OVMF.fd`

一些相关下载链接：

[iasl: ACPI Component Architecture Downloads](https://www.intel.com/content/www/us/en/download/774881/acpi-component-architecture-downloads-windows-binary-tools.html)  
[nasm: Netwide Assembler](https://www.nasm.us/pub/nasm/releasebuilds/?C=M;O=D)  
[Visual Studio Community](https://visualstudio.microsoft.com/zh-hans/free-developer-offers/)  

# 启动 QEMU 并安装 Ubuntu

接下来我们安装QEMU，并在QEMU中安装Ubuntu

## 安装 QEMU

QEMU可以在这里下载：[QEMU Binaries for Windows (64 bit)](https://qemu.weilnetz.de/w64/)

安装好之后可以添加安装目录到环境变量PATH中。

## QEMU 中安装 Ubuntu

**创建虚拟磁盘：**

```bash
qemu-img create -f qcow2 \path\to\ubuntu-disk.qcow2 20G
```

**启动 QEMU 安装 Ubuntu:**

这里分配 4GB 内存，使用 4 核 CPU，使用用户模式网络，指定 e1000 网卡，启用 USB 鼠标，并使用 Ubuntu 22.04.5 ISO 作为安装光盘。BIOS 则使用我们前面编译的 OVMF 的 Binary。

```bash
qemu-system-x86_64 -pflash \path\to\OVMF.fd^
 -m 4096 -smp 4^
 -cdrom \path\to\ubuntu-22.04.5-desktop-amd64.iso ^
 -drive file=\path\to\ubuntu-disk.qcow2,format=qcow2 ^
 -vga virtio ^
 -netdev user,id=net0 -device e1000,netdev=net0 ^
 -usb -device usb-tablet
```

安装 Ubuntu 即一般的在虚拟机中安装的方式，一路下一步就可以了。

**安装后，则直接可以从虚拟磁盘启动：**

下面脚本增加了 serial output，并同时保存到`serial.log`文件。

```bash
qemu-system-x86_64 -pflash \path\to\OVMF.fd^
 -m 4096 -smp 4^
 -drive file=\path\to\ubuntu-disk.qcow2,format=qcow2 ^
 -vga virtio ^
 -netdev user,id=net0 -device e1000,netdev=net0 ^
 -usb -device usb-tablet^
 -chardev stdio,id=char0,logfile=\path\to\serial.log,signal=off ^
 -serial chardev:char0
```

# 调试 ASL

接下来就是在 QEMU 的 Ubuntu 中调试 ASL 代码了。

## 直接查看 APCI Table

> 相关的代码：[TestAcpi in OvmfPkg](https://github.com/Li-Aaron/edk2/commit/7154f1924be25de910f83ed05ac9f37a9ab32eaa)

参考下面脚本

```bash
# ACPI table 都在这个路径
cd /sys/firmware/acpi/tables
# hexdump SSDT
sudo xxd SSDT
```

![1](/assets/img/posts/2025-02-08-asl-code-intro5/1.png)

## 利用 acpidbg 调试

安装 acpidbg 工具

```bash
sudo apt install linux-tools-`uname -r` linux-tools-generic
```

举个 acpidbg 调试的例子，参考 ASL [代码](https://github.com/Li-Aaron/edk2/commit/7154f1924be25de910f83ed05ac9f37a9ab32eaa#diff-f94633de94720e622d8dbe8cd6a664253fbabfc971e48c631e6af3e441b0e6a5)如下

```c
{
  Scope (\_SB)
  {
    Device (TEST)
    {
      Name (_HID, "ACPI0012")
      Name (_STR, Unicode ("Test ACPI Device"))
      Method (_STA, 0)
      {
        Return (0x0f)
      }
    }
  }
}
```

调试用的命令

```bash
sudo acpidbg

# 查找TEST的NamePath，同时可以预览其值。
- find TEST
# 显示TEST的所有信息。
- dump \_SB.TEST
# 执行\_SB.TEST._STA
- ev \_SB.TEST._STA
```

![2](/assets/img/posts/2025-02-08-asl-code-intro5/2.png)

关于 acpidbg 的更多用法，这里不再赘述，可以参考 [Debug under Linux (Ubuntu)]({{site.url}}/2023/04/asl-code-intro2/#debug-under-linux-ubuntu)。

## 实际操作 [OpRegion 实战]({{site.url}}/2023/04/asl-code-intro4/)中的内容

> 相关的代码：[TestAcpi: OpRegion](https://github.com/Li-Aaron/edk2/commit/73ec024b733e16e387942f9386f3a068e3d6655b)  

执行 SSDT 的 hexdump，可以看到 Patch 后的 OpRegion TEST 对应的Binary（图中绿色框）：

![3](/assets/img/posts/2025-02-08-asl-code-intro5/3.png)

通过 `find TEST`，找到 OperationRegion 的 memory 地址。  
执行 `ev \_SB.TEST.EXEC`，看前后 memory 的变化。  

![4](/assets/img/posts/2025-02-08-asl-code-intro5/4.png)

> 图中 memory dump 命令 `sudo hexdump -C --skip 0xBF55EE18 /dev/mem | head -n 6`
