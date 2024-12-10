---
layout: post
title: ASL语言学习（4）- OpRegion 实战
date: 2023-04-22 09:30:00.000000000 +08:00
description: 本文介绍ASL语言的Operation Region的用法。
author: aaron-li
categories: [代码相关, BIOS]
tags: [asl]  
---

续上三篇，这一篇实战一下与BIOS C代码结合的内存修改。  
[ASL语言学习（1）- 基本语法，作用域]({{site.url}}/2023/04/asl-code-intro/)，  
[ASL语言学习（2）- Debug，常用操作符]({{site.url}}/2023/04/asl-code-intro2/)  
[ASL语言学习（3）- 隐式转换，Method调用]({{site.url}}/2023/04/asl-code-intro3/)  

Reference:

[ACPI Source Language (ASL) Tutorial](https://acpica.org/sites/acpica/files/asl_tutorial_v20190625.pdf)

[ACPI Source Language (ASL) Reference](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html)

# OperationRegion 的运用方式

`OperationRegion`是ASL中最常用的关键字之一，在[第一篇文章]({{site.url}}/2023/04/asl-code-intro/#tocAnchor-1-2-3)中也讲过。

我们可以通过`OperationRegion`实现内存修改、寄存器修改等功能，这里实战一下与BIOS C代码结合的内存修改。

## 代码部分

ASL 示例如下（这是生成一个Fibonacci数列的递推）：

```c
    OperationRegion (TEST, SystemMemory, 0xA5A5A5A5, 0xA5A5A5A5)
    Field (TEST, ByteAcc, NoLock, Preserve) {
      ITER, 32,  // Iteration
      //
      // 16 bit x 32 = 1024 bits.
      //
      IBUF, 512, // Buffer
    }
    Method(EXEC, 0, Serialized) {
      Name (BUFF, Buffer(64){0}) //64 bytes = 512 bits.
      // BUFF[0] = 1
      // BUFF[1] = 1
      Store (0x1, Index(BUFF, 0))
      Store (0x1, Index(BUFF, 2))
      Store (2, Local0)
      // BUFF[i] = BUFF[i-2] + BUFF[i-1]
      // i in [2, ITER) -- Local0
      While (LLess(Local0, ITER)) {
        Multiply (Local0, 2, Local1) // 1 Word = 2 Byte
        Store (DerefOf(Index(BUFF, Subtract(Local1, 3))),Local2)        // BUFF[i-2] higher byte
        ShiftLeft (Local2, 8, Local2)
        Add (Local2, DerefOf(Index(BUFF, Subtract(Local1, 4))), Local2) // BUFF[i-2] lower byte

        Store (DerefOf(Index(BUFF, Subtract(Local1, 1))),Local3)        // BUFF[i-1] higher byte
        ShiftLeft (Local3, 8, Local3)
        Add (Local3, DerefOf(Index(BUFF, Subtract(Local1, 2))), Local3) // BUFF[i-1] lower byte

        Add (Local2, Local3, Local4)

        Store (And(Local4, 0xFF), Index(BUFF, Local1))                  // BUFF[i] lower byte
        ShiftRight(Local4, 8, Local4)
        Store (And(Local4, 0xFF), Index(BUFF, Add(Local1, 1)))          // BUFF[i] higher byte
        Increment (Local0)
      }
      Store (BUFF, IBUF)
    }
```

C语言部分简单描述下流程：

**1)** 在内存中Allocate一段Runtime可以访问的buffer

```c
typedef struct {
  UINT32                Iteration;
  UINT16                Buffer[0x20];
} ACPI_TEST_BUFFER;

Status = gBS->AllocatePool (
                EfiRuntimeServicesData,
                sizeof (ACPI_TEST_BUFFER),
                (VOID **) &mAcpiTestBuffer
                );
ZeroMem (mAcpiTestBuffer, sizeof (ACPI_TEST_BUFFER));
mAcpiTestBuffer->Iteration = 0x18;
```

**2)** Patch 这段buffer的地址和长度到ACPI Table中的OperationRegion关键字中。

参考Spec：[https://uefi.org/specs/ACPI/6.5/20_AML_Specification.html#defopregion](https://uefi.org/specs/ACPI/6.5/20_AML_Specification.html#defopregion)

通过检索如下关键字找到表中对应OperationRegion的Offset
```c
// DefOpRegion := OpRegionOp NameString RegionSpace RegionOffset RegionLen
// OpRegionOp := ExtOpPrefix 0x80
// ExtOpPrefix := 0x5B
// RegionSpace := ByteData  (0x00 SystemMemory)
// RegionOffset := TermArg => Integer (0x0A)
// RegionLen := TermArg => Integer
// TermArg := ExpressionOpcode | [DataObject] | ArgObj | LocalObj
// DataObject := [ComputationalData] | DefPackage | DefVarPackage
// ComputationalData := ByteConst | WordConst | [DWordConst] | QWordConst | String | ConstObj | RevisionOp | DefBuffer
// DWordConst := DWordPrefix DWordData
// DWordPrefix := 0x0C
```

结合上述在Spec中找到的定义，我们要找的Binary为：

```bash
5B 80 54 45 53 54 00 0C A5 A5 A5 A5 0C A5 A5 A5 A5
```

并将其中的两个`0xA5A5A5A5`替换为`mAcpiTestBuffer`的地址和长度。


## 运行部分

我们`find`命令查看，可以看到我们allocate的buffer地址成功被patch到`TEST`中(`000000006F342F18`)。

```bash
- find TEST
                       \_SB.TEST Region       00000000e15c3ac6 007 [SystemMemory] Addr 000000006F342F18 Len 0044
- find EXEC
                       \_SB.EXEC Method       0000000053e75605 007 Args 0 Len 00A8 Aml 00000000ae51bb6e
```

执行结果（这是BIOS log）：

```bash
# before execute
Dump data from 6F342F18, size: 0x44
6F338F18: 18 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  | ................
6F338F28: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  | ................
6F342F38: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  | ................
6F342F48: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  | ................
6F342F58: 00 00 00 00                                      | ....
# after execute and reset
Dump data from 6F342F18, size: 0x44
6F342F18: 18 00 00 00 01 00 01 00 02 00 03 00 05 00 08 00  | ................
6F342F28: 0D 00 15 00 22 00 37 00 59 00 90 00 E9 00 79 01  | ....".7.Y.....y.
6F342F38: 62 02 DB 03 3D 06 18 0A 55 10 6D 1A C2 2A 2F 45  | b...=...U.m..*/E
6F342F48: F1 6F 20 B5 00 00 00 00 00 00 00 00 00 00 00 00  | .o .............
6F342F58: 00 00 00 00                                      | ....
```

可以看到对应的内存被更改了。

## 一种错误示范

本来想图省事，用`CreateWordField`代替每个byte的读写和移位操作。

```c
    Method(EXCE, 0, Serialized) {
      Name (BUFF, Buffer(64){0}) //64 bytes = 512 bits.
      // BUFF[0] = 1
      // BUFF[1] = 1
      Store (0x1, Index(BUFF, 0))
      Store (0x1, Index(BUFF, 2))
      Store (2, Local0)
      // BUFF[i] = BUFF[i-2] + BUFF[i-1]
      // i in [2, ITER)
      While (LLess(Local0, ITER)) {
        Multiply(Local0, 2, Local1) // 1 Word = 2 Byte
        // 这东西不能在while里面用
        CreateWordField (BUFF, Subtract(Local1, 4), IDX1)
        CreateWordField (BUFF, Subtract(Local1, 2), IDX2)
        CreateWordField (BUFF, Local1, IDX3)
        Add (IDX1, IDX2, IDX3)
        Increment (Local0)
      }
      Store (BUFF, IBUF)
    }
```


结果在`While`的第二个loop中，报错：

```bash
3ACPI BIOS Error (bug): Failure creating named object [\_SB.EXCE.IDX1], AE_ALREADY_EXISTS (20220331/dsfield-184)
3ACPI Error: AE_ALREADY_EXISTS, CreateBufferField failure (20220331/dswload2-477)
3ACPI Error: Aborting method \_SB.EXCE due to previous error (AE_ALREADY_EXISTS) (20220331/psparse-529)
3ACPI Error: AE_ALREADY_EXISTS, while executing \_SB.EXCE from AML Debugger (20220331/dbexec-163)
Evaluation of \_SB.EXCE failed with status AE_ALREADY_EXISTS
```

即Object不能重复定义。

---

目前就这么多了，记笔记的时候并没有写这么多，很多都是在整理的时候拓展的，希望能帮到有需要的人。

如果后面还有值得一记的东西，还会继续整理。
