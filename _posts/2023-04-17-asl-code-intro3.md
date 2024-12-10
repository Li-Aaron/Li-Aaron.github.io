---
layout: post
title: ASL语言学习（3）- 隐式转换，Method调用
date: 2023-04-17 19:30:00.000000000 +08:00
description: 本文介绍ASL语言的部分隐式转换与Method调用。
author: aaron-li
categories: [代码相关, BIOS]
tags: [asl]
---

续上两篇，这一篇我们介绍ASL语言的部分隐式转换与Method调用  
[ASL语言学习（1）- 基本语法，作用域]({{site.url}}/2023/04/asl-code-intro/)，  
[ASL语言学习（2）- Debug，常用操作符]({{site.url}}/2023/04/asl-code-intro2/)  

Reference:

[ACPI Source Language (ASL) Tutorial](https://acpica.org/sites/acpica/files/asl_tutorial_v20190625.pdf)

[ACPI Source Language (ASL) Reference](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html)

# [隐式转换](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#implicit-data-type-conversions)

上一篇文章中提到Concatenate拼接不同类型时会导致隐式转换。

这一段我们主要研究下ASL语言中的常见的隐式转换，以Store为例。参考：[Rules for Storing and Copying Objects](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#rules-for-storing-and-copying-objects)

> 注：如`Add(X, Y, Z)`这种操作符，相当于`Store(Add(X, Y), Z)`，和Store所使用的隐式转换一致。

```c
Store (Source, Destination) => DataRefObject
// Destination = Source => DataRefObject
```

为了便于实验，我们先定义如下的Object：

```c
Name (BUF1, Buffer(3) {0})
Name (INT1, 0x12345678)
Name (STR1, "Hello World")
```

## Store to LocalX variables

这种情况会删除之前LocalX保存的object，不存在隐式转换。

示例如下：

```c
// Arg0 -> Local0
Method (TEC1, 1) {
  Store (Arg0, Local0)
  Printf ("%o", Local0)
  Return (Local0)
}
```

执行结果：

```bash
- execute \_SB.TEC1 0xABCD
ACPI Debug:  "000000000000ABCD"

- execute \_SB.TEC1 (00 11 22 33)
ACPI Debug:  "0x00 0x11 0x22 0x33"

- execute \_SB.TEC1 "Hello World"
ACPI Debug:  "Hello World"
```

## Store to Named Object

这种情况会隐式转换成Named Object当前的Object Type。

详细的转换规则参考：[Data Type Conversion Rules](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#data-type-conversion-rules)。

### Store to Buffer

在这种情况下，我们测试结果如下：  
1. Integer - Buffer (超出部分截断，只保留低位；不足补0)  
2. Buffer - Buffer (超出部分截断；不足补0)  
3. String - Buffer (保存对应字符ascii码，超出部分截断；不足补0)  

示例如下：

```c
// Arg0 -> BUF1
Method (TEC2, 1) {
  Store (Arg0, BUF1)
  Printf ("%o", BUF1)
  Return (BUF1)
}
```

执行结果：

```bash
- execute \_SB.TEC2 0xABCD
ACPI Debug:  "0xCD 0xAB 0x00"
- execute \_SB.TEC2 0xDEADBEEF
ACPI Debug:  "0xEF 0xBE 0xAD"
- execute \_SB.TEC2 (00 11)
ACPI Debug:  "0x00 0x11 0x00"
- execute \_SB.TEC2 (00 11 22 33 44 55 66 77 88 99 AA BB CC DD EE FF)
ACPI Debug:  "0x00 0x11 0x22"
- execute \_SB.TEC2 "Hi"
ACPI Debug:  "0x48 0x69 0x00"
- execute \_SB.TEC2 "Hello World My Friend"
ACPI Debug:  "0x48 0x65 0x6C"
```

### Store to Integer

在这种情况下，我们测试结果如下：  
1. Buffer - Integer (超出部分截断；不足补0)  
2. String - Integer (从第一个字符开始，转换16进制数，直到第一个不是16进制数的字符。超出截断**尾部**，不足**头部**补0)   

示例如下：

```c
// Arg0 -> INT1
Method (TEC3, 1) {
  Store (Arg0, INT1)
  Printf ("%o", INT1)
  Return (INT1)
}
```

执行结果：

```bash
- execute \_SB.TEC3 (00 11)
ACPI Debug:  "0000000000001100"
- execute \_SB.TEC3 (00 11 22 33 44 55 66 77 88 99 AA BB CC DD EE FF)
ACPI Debug:  "7766554433221100"
- execute \_SB.TEC3 "Hi"
ACPI Debug:  "0000000000000000"
- execute \_SB.TEC3 "Hello World My Friend"
ACPI Debug:  "0000000000000000"
- execute \_SB.TEC3 "AB1800"
ACPI Debug:  "0000000000AB1800"
- execute \_SB.TEC3 "AB18H00"
ACPI Debug:  "000000000000AB18"
- execute \_SB.TEC3 "AB18DEADBEEFDEADBEEF"
ACPI Debug:  "AB18DEADBEEFDEAD"
```

### Store to String
1. Integer - String (转换为8/16位16进制字符的字符串)  
2. Buffer - String (转换为`0x00 0x11 0x22`样式的字符串)  
3. String - String (与原长度无关，直接替换为新字符串，与Buffer - Buffer不同)  

示例如下：

```c
// Arg0 -> STR1
Method (TEC4, 1) {
  Store (Arg0, STR1)
  Printf ("%o", STR1)
  Return (STR1)
}
```

执行结果：

```bash
- execute \_SB.TEC4 0xABCD
ACPI Debug:  "000000000000ABCD"
- execute \_SB.TEC4 0xDEADBEEF
ACPI Debug:  "00000000DEADBEEF"
- execute \_SB.TEC4 (00 11)
ACPI Debug:  "0x00 0x11"
- execute \_SB.TEC4 (00 11 22 33 44 55 66 77 88 99 AA BB CC DD EE FF)
ACPI Debug:  "0x00 0x11 0x22 0x33 0x44 0x55 0x66 0x77 0x88 0x99 0xAA 0xBB 0xCC 0xDD 0xEE 0xFF"
- execute \_SB.TEC4 "Hi"
ACPI Debug:  "Hi"
- execute \_SB.TEC4 "Hello World My Friend"
ACPI Debug:  "Hello World My Friend"
```

> 注：结合Store to buffer/integer中的打印，我们可以看出`Printf (FormatString, FormatArgs)`也带有一种隐式转换(FormatArg->String)

## ObjectReference

ObjectReference 不支持任何形式的隐式转换

示例如下：

```c
// ObjectReference -> INT1
Method (TEC7) {
  Store (Index(BUF1, 0), INT1)
  Printf ("%o", INT1)
  Return (INT1)
}
```

执行结果：

```bash
- execute \_SB.TEC7
Evaluating \_SB.TEC7
3ACPI Error: Cannot assign type [Reference] to [Integer] (must be type Int/Str/Buf) (20220331/exstoren-90)
3ACPI Error: Aborting method \_SB.TEC7 due to previous error (AE_AML_OPERAND_TYPE) (20220331/psparse-529)
```

> 注：这里还尝试过ObjectReference - Buffer/String，结果会导致Kernal Panic。  

<!-- // todo: 写到这 -->
# Method 调用

ASL中的Method可以互相调用，和C语言类似。也可以实现传值、传引用。

我们回顾一下C语言的函数调用：传值调用（入参为值），传址/引用调用（入参为地址）。

> 注：
> 实际上这么理解不完全正确，C语言的函数调用其实都是将入参**复制一份**到stack上，再函数返回时，stack上的入参均会被回收，所以本质上都是传值调用。  
> 所谓C语言的传址调用，即声明传入参数是个pointer，然后对pointer指向的地址里面的内容进行修改。  

而在[ACPI Spec 5.5.2.2](https://uefi.org/specs/ACPI/6.5/05_ACPI_Software_Programming_Model.html#method-calling-convention)中提到：  
> The calling convention for control methods can best be described as call-by-reference-constant. In this convention, objects passed as arguments are passed by “reference”, meaning that they are not copied to new objects as they are passed to the called control method (A calling convention that copies objects or object wrappers during a call is known as call-by-value or call-by-copy).  
> However, unlike a pure call-by-reference convention, the ability of the called control method to modify arguments is extremely limited. This reduces aliasing issues such as when a called method unexpectedly modifies a object or variable that has been passed as an argument by the caller. In effect, the arguments that are passed to control methods are passed as constants that cannot be modified except under specific controlled circumstances.  


即ASL的Method都是引用调用，而调用过程中将其值作为常量使其无法被更改（除非入参包含引用）。

我们测试下ASL语言对不同类型入参进行修改：

## 入参为 Integer

Integer不包含引用。

示例如下：

```c
    Name (INTA, 0x5A5A)
    Method (T000, 1)
    {
      Store (0x1234, Arg0)
      Printf ("Arg0 is %o", Arg0)
    }
    Method (TSTA)
    {
      T000 (INTA)
      Printf ("INTA is %o", INTA)
    }
```

执行结果：

```bash
- execute \_SB.TSTA
ACPI Debug:  "Arg0 is 0000000000001234"
ACPI Debug:  "INTA is 0000000000005A5A"
```

我们直接改变入参Arg0的时候，INTA的值并没有发生改变。(这里Arg0变成和LocalX一样的存在)

> When an ArgX term is used as a target operand in an ASL statement, the existing ArgX object is not modified. Instead, the new object replaces the existing object and the ArgX term effectively becomes a LocalX term.

## 入参为 Buffer/Package

Buffer/Package则包含引用。

示例如下：

```c
    Name (INTB, 0x5A5A)
    Name (PKGB, Package() {INTB})
    Name (BUFB, Buffer() {0xEE})
    Method (T001, 1)
    {
      Store (0x1234, Index(Arg0, 0)) // 这里并不是对Arg0直接做出修改，而是对Arg0中的第一个元素进行修改。
      Printf ("Arg0[0] is %o", DerefOf (Index (Arg0, 0)))
    }
    Method (TSTB)
    {
      T001 (PKGB)
      T001 (BUFB)
      Printf ("INTB is %o", INTB)
      Printf ("PKGB[0] is %o", DerefOf (Index (PKGB, 0)))
      Printf ("BUFB[0] is %o", DerefOf (Index (BUFB, 0)))
    }
```

执行结果：

```bash
- execute \_SB.TSTB
ACPI Debug:  "Arg0[0] is 0000000000001234"
ACPI Debug:  "Arg0[0] is 0000000000000034"
ACPI Debug:  "INTB is 0000000000005A5A"
ACPI Debug:  "PKGB[0] is 0000000000001234"
ACPI Debug:  "BUFB[0] is 0000000000000034"
```

通过`Index(Arg0, 0)`对PKGB/BUFB中第一个元素进行修改的时候，它们的值发生了改变。

> The only exception to the read-only argument rule is if an ArgX term contains an Object Reference created via the RefOf ASL operator. In this case, the use of the ArgX term as a target operand will cause any existing object stored at the ACPI name referred to by the RefOf operation to be overwritten.

## 入参为 Reference (Index)

之前提到过`Index`的返回值是`ObjectReference`类型。
如果试一下直接把`Index(PKGC, 0)`作为参数，是否能直接修改其值呢。

示例如下：

```c
    Name (INTC, 0x5A5A)
    Name (PKGC, Package() {INTC})
    Name (BUFC, Buffer() {0xEE})
    Method (T002, 1)
    {
      Printf ("Arg0 is %o", Arg0)
      Printf ("DerefOf(Arg0) is %o", DerefOf (Arg0))
      Store (0x1234, Arg0) 
      Printf ("Arg0 is %o", Arg0)
    }
    Method (TSTC)
    {
      T002 (Index(PKGC, 0))
      T002 (Index(BUFC, 0))
      Printf ("PKGC[0] is %o", DerefOf (Index (PKGC, 0)))
      Printf ("BUFC[0] is %o", DerefOf (Index (BUFC, 0)))
    }
```

执行结果：

```bash
- execute \_SB.TSTC
ACPI Debug:  "Arg0 is [Reference Object]"
ACPI Debug:  "DerefOf(Arg0) is 0000000000005A5A"
ACPI Debug:  "Arg0 is 0000000000001234"
ACPI Debug:  "Arg0 is [Reference Object]"
ACPI Debug:  "DerefOf(Arg0) is 00000000000000EE"
ACPI Debug:  "Arg0 is 0000000000001234"
ACPI Debug:  "INTC is 0000000000005A5A"
ACPI Debug:  "PKGC[0] is 0000000000005A5A"
ACPI Debug:  "BUFC[0] is 00000000000000EE"
```

结果`Arg0`的Type变成了Integer。

如果我们将Store语句变为`Store (0x1234, DerefOf (Arg0))`，则会报如下错误：

```bash
3ACPI Error: Needed type [Reference], found [Integer] 00000000efa09de7 (20220331/exresop-66)
3ACPI Error: AE_AML_OPERAND_TYPE, While resolving operands for [Store] (20220331/dswexec-431)
```

## 入参为 Reference (RefOf)

这样我们再测试一下直接传一个Object的`RefOf`。

[`RefOf`](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#refof-create-object-reference)和上一篇提到的`DerefOf`正好相反，是获取一个Object的Reference。

```c
RefOf (Object) => ObjectReference
```

示例如下：

```c
    Name (INTD, 0x5A5A)
    Method (T003, 1)
    {
      Printf ("Arg0 is %o", Arg0)
      Printf ("DerefOf(Arg0) is %o", DerefOf (Arg0))
      Store (0x1234, Arg0)
      Printf ("Arg0 is %o", Arg0)
      Printf ("DerefOf(Arg0) is %o", DerefOf (Arg0))
    }
    Method (TSTD)
    {
      T003 (RefOf(INTD))
      Printf ("INTD is %o", INTD)
    }
```

执行结果：

```bash
- execute _SB.TSTD
ACPI Debug:  "Arg0 is [Reference Object]"
ACPI Debug:  "DerefOf(Arg0) is 0000000000005A5A"
ACPI Debug:  "Arg0 is [Reference Object]"
ACPI Debug:  "DerefOf(Arg0) is 0000000000001234"
ACPI Debug:  "INTD is 0000000000001234"
```

可以看到INTD的值成功被改变。

## 两种 Reference 的区别

我们用debug的方式执行这两个method，发现`T002 (Index(PKGC, 0))`的Arg0，类型为`Index`。而`T003 (RefOf(INTD))`的Arg0，类型为`RefOf`。

```bash
- debug \_SB.TSTC
...
AML Opcode: 0035 -MethodCall-
% into
ArgObj:  00000000af3c52e7 [Index] 00000000b0957730 Integer 0000000000005A5A

- debug \_SB.TSTD
...
AML Opcode: 0035 -MethodCall-
% into
ArgObj:  00000000d5ccae9c [RefOf] <Node>          Name INTD Integer 0000000000005A5A
```

目前并没有在Spec上面找到`Index`和`RefOf`在ReturnType上有什么区别。

```c
RefOf (Object) => ObjectReference
Index (Source, Index, Destination) => ObjectReference
// Destination = Source [ Index ] => ObjectReference
```

> 注：个人建议，ASL code里面不要玩**花活**，要写simple code，毕竟ACPI基本是和硬件相关，代码都不会太复杂。

## 跨表调用

这一段主要是讲如何跨ACPI表调用Method，其实和C语言很像。使用[`External`](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#external-declare-external-objects)关键字：

```c
External ( ObjectName, ObjectType, ReturnType, ParameterTypes )
```

1. ObjectName：顾名思义  
2. ObjectType：入参的类型，如IntObj，BuffObj，MethodObj.（Default：UnknownObj）  
3. ReturnType：如果ObjectType是MethodObj，定义返回值的Object Type. (Default: 不限制)  
4. ParameterTypes：如果ObjectType是MethodObj，定义入参的Object Type，每个入参依次定义。(Default: 不限制)   

然后在调用之前，需要check existance。（ASL是解释型语言，external method可能存在也可能不存在，不像C语言会报LINK ERROR）。

Check existance建议使用[`Condrefof`](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#condrefof-create-object-reference-conditionally)关键字：

```c
CondRefOf (Source, Result) => Boolean
```
如果RefOf的Source不存在，直接会出现fatal，而如果CondRefOf的Source不存在，则会return false。

示例如下：

```c
External (\_SB.TESS, MethodObj)
...
  If (CondRefOf (\_SB.TESS)) {
    \_SB.TESS(INT1)
  }
...
```

---

后面还没整理完，下篇文章接着写。
