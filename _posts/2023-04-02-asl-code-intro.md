---
layout: post
title: ASL语言学习（1）- 基本语法，作用域
date: 2023-04-02 14:00:00.000000000 +08:00
excerpt: 本文介绍ASL语言的基本语法，作用域以及NamePath。
---

最近的工作中用到了很多ASL的代码，之前一直一知半解的看，发现到用的时候不熟练还是很影响效率。

ASL代码的中文教程几乎没有，为了方便自己以后看，也方便别人检索，这里根据[ACPI Source Language (ASL) Tutorial](https://acpica.org/sites/acpica/files/asl_tutorial_v20190625.pdf) 以及 [ACPI Source Language (ASL) Reference](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html)，把我平时记的笔记整理成人话。这一篇简单介绍下一些基本的表达式用法。

# ASL与AML
ASL - ACPI Source Language
AML - ACPI Machine Language

ASL 代码文件通过 iASL compiler 生成 AML字节码文件，每一个table即一个ASL文件，生成一个对应的AML文件包含在Firmware中。

# Grammar 语法
ASL的表达式(Statement)声明对象(Object)
```c
Object := ObjectType FixedList VariableList
```

其中  
`Operator` 是ACPI中规定的操作符，如`DefinitionBlock`，`Name`，`Method`，`Scope`  
`FixedList` 是一组固定长度的变量，根据不同的`Operator`，有不同的长度，也可能为空，写法：(a, b, c)，有时尾部的变量可以省略（Default值生效）  
`VariableList`是一组不定长度的变量，写法：{x, y, z}，对于某些`Operator`，这里的变量(argument)可以是嵌套的对象。  

整体就是有点C语言函数的样子。（虽然长得像但是有很显著的差异）  
```c
Operator (FixedVariableA, FixedVariableB, FixedVariableC) {
  VariableX,
  VariableY,
}
```

下面介绍一些常用的操作符与其对应的表达式

## [DefinationBlock](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#definitionblock-declare-definition-block) - ASL的基础

每个`DefinitionBlock`定义一个ACPI table，所有的ASL code都需要写到`DefinitionBlock`内（作为`VariableList`），一个标准的`DefinitionBlock`如下：

```c
DefinitionBlock (
    AMLFileName,
    TableSignature,     // Signature of the AML file (could be DSDT or SSDT) (4-character string)
    ComplianceRevision, // A value of 2 or greater enables 64-bit arithmetic; a value of 1 or less enables 32-bit arithmetic (8 bit unsigned integer)
    OEMID,              // (6-character string)
    TableID,            // (8-character string)
    OEMRevision         // (32-bit number)
    )
{
    //ASL Code
}
```

注意`ComplianceRevision`小于2时，`NameSpace`定义的Integer Object为32bit，反之则为64bit。

## [Name](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#name-declare-named-object) - ASL的“变量”

`Name`可以用来定义Integer, String, Buffer, Package等等对象：
```c
// ObjectName 必须小于等于4个字母，可以下划线开头
Name (ObjectName, Object)
```
**Object Types:**  

• Integer - An unsigned 64-bit or 32-bit integer. Size取决于`ComplianceRevision`  
• String - A null-terminated ASCII string. 字符串  
• Buffer - An array of **bytes**. 类似数组  
• Package - An array of ASL objects.   
```c
DefinitionBlock ("", DSDT, 2, "", "", 0x0)
{
    Name (OBJ0, 0x1234)                            // Integer (DW or DD)
    Name (OBJ1, "Hello world")                     // String
    Name (BUF1, Buffer(3){0x00, 0x01, 0x02})       // Buffer  (Byte)
    Name (BUF2, Buffer(){0x00, 0x01, 0x02, 0x03})  // Buffer 长度自动初始化
    Name (PKG1, Package(3){0x1234, "Hello world", INT1}) // Package
    Name (PKG2, Package(){INT1, "Good bye"})             // Package 长度自动初始化
    Name (PKG3,
        Package(){
            Package() {0x00, 0x01, 0x02},
            Package() {0x03, 0x04, 0x05}
        }) // Package in Package also Package
    Name (PKG4, Package(){
        "ASL is fun",
        Package() {0xff, 0xfe, 0xfd, 0xfc, fb}})
    Name (PKG5, Package(){
        0x4321,
        Buffer() {0x1}
    })
}
```



## [OperationRegions](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#operationregion-declare-operation-region) and [Fields](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#field-declare-field-objects) - ASL的“内存指针”

```c
OperationRegion (RegionName, RegionSpace, Offset, Length)
Field ( RegionName, AccessType, LockRule, UpdateRule ) {FieldUnitList}
```
（这个“操作区”，描述成“内存指针”也不是很贴切，总之OperationRegion类似一个指针，Field类似这个指针对应的结构体定义。）

这是ASL中用来访问系统内存或者IO Space的方法。

通常在C语言中有对应的结构体。

```c
DefinitionBlock ("", "DSDT", 2, "", "", 0x1)
{
    //             (RegionName, RegionSpace,  Offset,  Length)
    OperationRegion(OPR1,       SystemMemory, 0x10000, 0x5)
    Field (OPR1)
    {
        FLD1, 8
        FLD2, 8
        Offset (3), //Start the next field unit at byte offset 3
        FLD3, 4
        FLD4, 12
    }
}
```

上面表达式生成的OperationRegion如下：

![fields.png](/assets/images/2023-04-02-asl-code-intro/fields.png)
（本图片来自[ACPI Source Language (ASL) Tutorial](https://acpica.org/sites/acpica/files/asl_tutorial_v20190625.pdf)）

> 注：OperationRegion也是ObjectType的一种

# Name Space 作用域

（不确定这么翻译准不准确。）

除了上文中提到Object Type以外，ASL语言还支持很多其他的Object Type，如：

• Device - Device or bus object. 硬件对象  
• Object Reference - A reference to an object created by RefOf, Index, or CondRefOf operators. 类似指针  
• Method - Control Method (Executable AML function). 类似函数  

详细参见： [All Data Types](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#asl-data-types)


## [Scope](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#scope-open-named-scope) - ASL中的作用域


Scope和其他语言的scope一样，提供了一个作用域，在ASL中独特的是这个作用域需要与实际硬件的Location相关。

```c
Scope (Location) {ObjectList}
```
Location - [Predefined Root Namespaces](https://uefi.org/specs/ACPI/6.5/05_ACPI_Software_Programming_Model.html?highlight=namepath#namespaces-defined-under-the-namespace-root)：

1. `\`: 根节点
2. `_SB`: System Bus
3. `_GPE`: General Purpose Event
4. `_PR`: Processor NameSpace
5. `_TZ`: Thermal Zone

> 注：在创建SSDT的时候通常需要使用Scope来更改namespace location，以便在DSDT定义的NameSpace中创建对象。


## [Device](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#device-declare-device-package) - ASL中描述硬件的作用域

```c
Device (DeviceName) {TermList}
```

每个Device下是独立的作用域。

Device可以嵌套Device。一般来讲Device都定义在Scope里面。

下面这段代码定义了System Bus上的UART(COM0)与USB EHCI Host Controller(USB0)，其中USB0中包含Root Hub(RHUB)。

```c
// 这段代码来自edk2-platforms\Silicon\Qemu\SbsaQemu\AcpiTables\Dsdt.asl
DefinitionBlock ("DsdtTable.aml", "DSDT",
                 EFI_ACPI_6_0_DIFFERENTIATED_SYSTEM_DESCRIPTION_TABLE_REVISION,
                 "LINARO", "SBSAQEMU", FixedPcdGet32 (PcdAcpiDefaultOemRevision)) {
  Scope (_SB) {
    // UART PL011
    Device (COM0) {
      Name (_HID, "ARMH0011")
      Name (_UID, Zero)
      ...
    }
    // USB EHCI Host Controller
    Device (USB0) {
        Name (_HID, "LNRO0D20")
        Name (_CID, "PNP0D20")
        ...
        // Root Hub
        Device (RHUB) {
            Name (_ADR, 0x00000000)  // Address of Root Hub should be 0 as per ACPI 5.0 spec
            ...
        } // USB0_RHUB
    } // USB0
```

## [NamePath](https://uefi.org/specs/ACPI/6.5/05_ACPI_Software_Programming_Model.html?highlight=namepath#acpi-namespace)

在ASL中每一个Object都有一个NamePath，这个NamePath也标示出其作用域的范围。

举个例子：

![namepath.png](/assets/images/2023-04-02-asl-code-intro/namepath.png)

> 注：NamePath中符号的意义  
> `\` - 根节点  
> `.` - 子节点  
> `^` - 上一层节点  
> `_` - Reserved by Specification

# Method 执行函数

## [Method](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#method-declare-control-method) - ASL的“函数”
```c
Method ( MethodName, NumArgs, SerializeRule, SyncLevel, ReturnType, ParameterTypes ) {TermList}
```
Method的参数比较多，下面详细介绍一下：

1. MethodName：顾名思义
2. NumArgs：入参的个数，Method最多可以传递 **7 个参数**，在Method里用 Arg0~Arg6 表示，不可以自定义。 (Default: 0)
3. SerializeRule：当函数声明为 **Serialized**，内存中仅能存在一个实例（即不可重入）。(Default: NotSerialized)
4. SyncLevel： Synchronization level (0 - 15). (Default: 0) （这一块没有深入的了解，目前看到的代码都是0）
5. ReturnType： 定义返回值的Object Type. (Default: 不限制)
6. ParameterTypes： 定义入参的Object Type，每个入参依次定义。(Default: 不限制)

此外：

1. Method最多可以用 **8 个局部变量**，用 Local0~Local7，不可以自定义，使用前需要初始化。
2. ASL methods can be called by other ASL methods or by the OS through the AML interpreter.
3. 除了MethodName，其他均为可选参数。

示例：
```c
// In this function
// MethodName     -- _DSM
// NumArgs        -- 4 (max 7, Arg0 ~ Arg6)
// SerializeRule  -- Serialized
// SyncLevel      -- 0
// ReturnType     -- {BuffObj, PkgObj}  // 2 kind possible return types
// ParameterTypes -- {BuffObj, IntObj, IntObj, PkgObj} // 4 input arg types in order

Method (_DSM, 4, Serialized, 0, {BuffObj, PkgObj}, {BuffObj, IntObj, IntObj, PkgObj}) {
  ...
}
```

## [运算符 (Operators)](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html?highlight=synclevel#asl-2-0-symbolic-operators-and-expressions)
<table>
    <tr>
        <td align="center" colspan = 2><strong>Math operators</strong></td>
        <td align="center" colspan = 2><strong>Logical operators</strong></td>
        <td align="center" colspan = 2><strong>Assignment operators</strong></td>
    </tr>
    <tr>
        <td align="center"><strong>ASL 2.0</strong></td>
        <td align="center"><strong>Legacy ASL</strong></td>
        <td align="center"><strong>ASL 2.0</strong></td>
        <td align="center"><strong>Legacy ASL</strong></td>
        <td align="center"><strong>ASL 2.0</strong></td>
        <td align="center"><strong>Legacy ASL</strong></td>
    </tr>
    <tr>
        <td align="center">Z = X + Y</td>
        <td align="center">Add (X, Y, Z)</td>
        <td align="center">(X == Y)</td>
        <td align="center">LEqual (X, Y)</td>
        <td align="center">X = Y</td>
        <td align="center">Store (Y, X)</td>
    </tr>
    <tr>
        <td align="center">Z = X / Y</td>
        <td align="center">Divide (X, Y, , Z)</td>
        <td align="center">(X != Y)</td>
        <td align="center">LNotEqual (X, Y)</td>
        <td align="center">X += Y</td>
        <td align="center">Add (X, Y, X)</td>
    </tr>
    <tr>
        <td align="center">Z = X % Y</td>
        <td align="center">Mod (X, Y, Z)</td>
        <td align="center">(X &lt; Y)</td>
        <td align="center">LLess (X, Y)</td>
        <td align="center">X /= Y</td>
        <td align="center">Divide (X, Y, , X)</td>
    </tr>
    <tr>
        <td align="center">Z = X * Y</td>
        <td align="center">Multiply (X, Y, Z)</td>
        <td align="center">(X &gt; Y)</td>
        <td align="center">LGreater (X, Y)</td>
        <td align="center">X %= Y</td>
        <td align="center">Mod (X, Y, X)</td>
    </tr>
    <tr>
        <td align="center">Z = X - Y</td>
        <td align="center">Subtract (X, Y, Z)</td>
        <td align="center">(X &lt;= Y)</td>
        <td align="center">LLessEqual (X, Y)</td>
        <td align="center">X *= Y</td>
        <td align="center">Multiply (X, Y, X)</td>
    </tr>
    <tr>
        <td align="center">Z = X &lt;&lt; Y</td>
        <td align="center">ShiftLeft (X, Y, Z)</td>
        <td align="center">(X &gt;= Y)</td>
        <td align="center">LGreaterEqual (X, Y)</td>
        <td align="center">X -= Y</td>
        <td align="center">Subtract (X, Y, X)</td>
    </tr>
    <tr>
        <td align="center">Z = X &gt;&gt; Y</td>
        <td align="center">ShiftRight (X, Y, Z)</td>
        <td align="center">(X &amp;&amp; Y)</td>
        <td align="center">LAnd (X, Y)</td>
        <td align="center">X &lt;&lt;= Y</td>
        <td align="center">ShiftLeft (X, Y, X)</td>
    </tr>
    <tr>
        <td align="center">Z = X &amp; Y</td>
        <td align="center">And (X, Y, Z)</td>
        <td align="center">(X || Y)</td>
        <td align="center">LOr (X, Y)</td>
        <td align="center">X &gt;&gt;= Y</td>
        <td align="center">ShiftRight (X, Y, X)</td>
    </tr>
    <tr>
        <td align="center">Z = X | Y</td>
        <td align="center">Or (X, Y, Z)</td>
        <td align="center">!X</td>
        <td align="center">LNot (X)</td>
        <td align="center">X &amp;= Y</td>
        <td align="center">And (X, Y, X)</td>
    </tr>
    <tr>
        <td align="center">Z = X ^ Y</td>
        <td align="center">Xor (X, Y, Z)</td>
        <td align="center" colspan = 2><strong>Miscellaneous</strong></td>
        <td align="center">X |= Y</td>
        <td align="center">Or (X, Y, X)</td>
    </tr>
    <tr>
        <td align="center">Z = ~X</td>
        <td align="center">Not (X, Z)</td>
        <td align="center"><strong>ASL 2.0</strong></td>
        <td align="center"><strong>Legacy ASL</strong></td>
        <td align="center">X ^= Y</td>
        <td align="center">Xor (X, Y, X)</td>
    </tr>
    <tr>
        <td align="center">X++</td>
        <td align="center">Increment (X)</td>
        <td align="center">Z = X[Y]</td>
        <td align="center">Index (X, Y, Z)</td>
        <td align="center"></td>
        <td align="center"></td>
    </tr>
    <tr>
        <td align="center">X–</td>
        <td align="center">Decrement (X)</td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"></td>
        <td align="center"></td>
    </tr>
</table>

<!-- //todo: 当前写到这里 -->
## [流程控制 (Control Flow)](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html?highlight=synclevel#asl-operator-reference)

ASL的Control Flow与C语言基本一致。包括:

[If](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html?highlight=synclevel#if-conditional-execution), [Elseif](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html?highlight=synclevel#elseif-alternate-conditional-execution), [Else](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html?highlight=synclevel#else-alternate-execution) -- 条件选择

[Switch](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html?highlight=synclevel#switch-select-code-to-execute-based-on-expression), [Case](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html?highlight=synclevel#case-expression-for-conditional-execution), [Default](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html?highlight=synclevel#default-default-execution-path-in-switch) -- 多分支选择

[For](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html?highlight=synclevel#for-conditional-loop), [While](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html?highlight=synclevel#while-conditional-loop), [Break](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html?highlight=synclevel#break-break-from-while), [Continue](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html?highlight=synclevel#continue-continue-innermost-enclosing-while) -- 循环

这里举个例：
```c
DefinitionBlock ("", "DSDT", 2, "", "", 0x0)
{
  // Lagecy ASL
  Method (PRT1, 1) {
    Store(Arg0, Local0) //Local0 = Arg0
    While (Local0) {
      Mod (Local0, 2, Local1) //Local1 = Local0 % 2
      If (Local1) {
        Printf ("%o is odd", Local0)
      } Else {
        Printf ("%o is even", Local0)
      }
      Decrement (Local0)	//Local0--;
    }
  }
  // ASL2
  Method (PRT2, 1)
  {
    Local0 = Arg0
    For (Local0 = 0, Local0 < Arg0, Local0++)
    {
      Switch(Local0) {
        Case(5) {
          PRT1(Local0)
        }
        Case(10) {
          Break
        }
        Default {
          Continue
        }
      }
      Printf ("Now is %o", Local0)
    }
  }
}
```

---

后面还没整理完，下篇文章接着写。