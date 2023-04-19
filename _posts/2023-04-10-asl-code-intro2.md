---
layout: post
title: ASL语言学习（2）- Debug，常用操作符
date: 2023-04-10 19:30:00.000000000 +08:00
excerpt: 本文介绍ASL语言的Debug方式，以及一些常用操作符的使用。
---

续上一篇[ASL语言学习（1）- 基本语法，作用域]({{site.url}}/2023/04/asl-code-intro/)，这一篇我们介绍ASL语言的一种debug方法，以及一些常用操作符的使用。

Reference:

[ACPI Source Language (ASL) Tutorial](https://acpica.org/sites/acpica/files/asl_tutorial_v20190625.pdf)

[ACPI Source Language (ASL) Reference](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html)

# Debug under Linux (Ubuntu)

ACPI table是由BIOS提供在OS下使用的Interface，debug要在OS下进行，这里介绍下Linux下debug的工具：`acpidbg`。

本人参考了这篇[文章](https://ubuntu.com/blog/acpi-aml-runtime-debugger-in-ubuntu-18-04-x64)做的环境配置，使用的系统是Ubuntu 22.04。

## 环境配置

acpidbg由linux kernel提供，要安装这个package：

```bash
sudo apt install linux-tools-`uname -r` linux-tools-generic
```

acpidbg需要在superuser下运行，可以用交互式命令
```bash
sudo apcidbg
- help

Summary of AML Debugger Commands


Namespace Access:
  Businfo                             Display system bus info
  Disassemble <Method>                Disassemble a control method
  ...

- exit
log read pipe closed.
```


也直接用batch mode：

```bash
sudo acpidbg -b help

Summary of AML Debugger Commands


Namespace Access:
  Businfo                             Display system bus info
  Disassemble <Method>                Disassemble a control method
  ...
```

## 常用命令

help message 提供了很多命令，这里介绍下常用的几个命令。

### `find`
查找Object的NamePath，同时可以预览其值。示例如下：

```bash
- find TEC1
                       \_SB.TEC1 Method       00000000d398a1f7 007 Args 1 Len 000D Aml 00000000c3104e44
```
如果有多个作用域同名Object的会一起打出来。

```bash
- find BUF1
                  \_SB.MTH1.BUF1 Buffer       00000000ca490f2e 005 Len 10 = 11 22 33 44 55 66 77 88 99 AA BB CC
                       \_SB.BUF1 Buffer       0000000085cc641a 007 Len 03 = 48 65 6C
```
还可以使用通配符`?`
```bash
- find TEC?
                       \_SB.TEC1 Method       00000000d398a1f7 007 Args 1 Len 000D Aml 00000000c3104e44
                       \_SB.TEC2 Method       00000000a0d9520d 007 Args 1 Len 0016 Aml 00000000631c9553
                       \_SB.TEC3 Method       00000000bdee4709 007 Args 1 Len 0016 Aml 0000000054f643e9
                       \_SB.TEC4 Method       00000000a89657a5 007 Args 1 Len 0016 Aml 00000000759d7803
                       \_SB.TEC5 Method       0000000004b8040a 007 Args 0 Len 0099 Aml 0000000059cf2ee9
                       \_SB.TEC6 Method       00000000ef623c07 007 Args 0 Len 0061 Aml 000000007fbd2141
                       \_SB.TEC7 Method       00000000eb393881 007 Args 0 Len 001D Aml 00000000567e0efb
```

### `dump`
显示一个Object的所有信息。示例如下：

```bash
- dump \_SB.BUF1
Object 0000000085cc641a: Namespace Node - Pathname: \_SB.BUF1
                Name : BUF1
                Type : 03 [Buffer]
               Flags : 0000
            Owner Id : 0007
         Object List : 00000000d43decde Buffer (Type 03)
              Parent : 000000005d923ff5 [_SB_]
               Child : 0000000000000000
                Peer : 000000009b68a860 [INT1]

Attached Object 00000000d43decde:
                Type : 03 [Buffer]
     Reference Count : 0001
               Flags : 04
         Object List : 0000000000000000 - No attached objects
              Length : 00000003
             Pointer : 00000000d1b3dd47
         Parent Node : 0000000085cc641a [BUF1]
```

### `execute`, `evaluate`
这两个是同一个命令，执行一个method。

```bash
Execute <Namepath> [Arguments] 
  [Arguments] formats:                Control method argument formats
     Hex Integer                      Integer
     "Ascii String"                   String
     (Hex Byte List)                  Buffer
         (01 42 7A BF)                Buffer example (4 bytes)
     [Package Element List]           Package
         [0x01 0x1234 "string"]       Package example (3 elements)
```

示例如下：
```bash
# 传入Integer
- execute \_SB.TEC1 0xABCD
Evaluating \_SB.TEC1
ACPI Debug:  "000000000000ABCD"
Evaluation of \_SB.TEC1 returned object 00000000c727936a, external buffer length 18
 [Integer] = 000000000000ABCD

# 传入Buffer
- execute \_SB.TEC1 (00 11 22 33)
Evaluating \_SB.TEC1
ACPI Debug:  "0x00 0x11 0x22 0x33"
Evaluation of \_SB.TEC1 returned object 00000000c727936a, external buffer length 20
 [Buffer] Length 04 =

# 传入String
- execute \_SB.TEC1 "Hello World"
Evaluating \_SB.TEC1
ACPI Debug:  "Hello World"
Evaluation of \_SB.TEC1 returned object 00000000c727936a, external buffer length 28
 [String] Length 0B = "Hello World"
```

### `debug`
`Debug`和`Execute`执行方法一样，不同的是可以单步运行。

```bash
Control Method Single-Step Execution:
  Arguments (or Args)                 Display method arguments
  Breakpoint <AmlOffset>              Set an AML execution breakpoint
  Call                                Run to next control method invocation
  Go                                  Allow method to run to completion
  Information                         Display info about the current method
  Into                                Step into (not over) a method call
  List [# of Aml Opcodes]             Display method ASL statements
  Locals                              Display method local variables
  Results                             Display method result stack
  Set <A|L> <#> <Value>               Set method data (Arguments/Locals)
  Stop                                Terminate control method
  Tree                                Display control method calling tree
  <Enter>                             Single step next AML opcode (over calls)
```

示例如下：
```bash
- debug \_SB.TEC1 0x0102
Evaluating \_SB.TEC1
AML Opcode: 0070 Store # 这里指出下一步Store操作，对应的代码并不显示。


% arguments  # 这里操作提示符会变成%, arguments命令打印所有传入参数
Initialized Arguments for Method [TEC1]:  (1 arguments defined for method invocation)
  Arg0:   000000003e1754d7 <Obj>           Integer 0000000000000102

% # 直接Enter执行下一步  Store(Arg0, Local0)
ArgObj:  00000000c36bdcdf [Local] 0 0000000000000000 Uninitialized               # Local0 执行前值 （入参1）
ArgObj:  00000000c8dc3c8d [Argument] 0 000000003e1754d7 Integer 0000000000000102 # Arg0   执行前值 （入参2）
ResultObj: 000000003e1754d7 <Obj>           Integer 0000000000000102             # Result值，这里即Local0执行后的值

AML Opcode: 0073 Concatenate


% locals #locals命令打印所有临时变量

Initialized Local Variables for Method [TEC1]:
  Local0: 00000000c8dc3c8d <Obj>           Integer 0000000000000102

% set L 0 0x010203 #Set命令可以改变Args和Locals的值
Local0: 0000000007cfb617 <Obj>           Integer 0000000000010203


% locals #再次查看，Local0被Set命令改变。

Initialized Local Variables for Method [TEC1]:
  Local0: 0000000007cfb617 <Obj>           Integer 0000000000010203
```


# 一些常用操作符
全部的操作符可以通过[ACPI Spec 19.6 ASL Operator Reference](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#asl-operator-reference)查找。
> 示例中的输出做出了一些简化以节省篇幅，便于阅读。

## [Sleep](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#sleep-milliseconds-sleep)
```c
// delays longer than 100 microseconds must use Sleep instead of Stall.
Sleep (MilliSeconds) //ms
```

## [Stall](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#stall-stall-for-a-short-time)

```c
Stall (MicroSeconds) //us
```

## [Printf](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#index-indexed-reference-to-member-object)
打印字符串到debug console，这个和C语言类似。
```c
Printf (FormatString, FormatArgs) => String
```

示例如下：
```c
    Method (TST1) {
      Printf ("Hello world")
    }
    Name (STR1, "it is a wonderful day")
    Method (TST2) {
      Printf ("Hello world, %o", STR1)
    }
```
执行结果：
```bash
- execute \_SB.TST1
ACPI Debug:  "Hello world"

- execute \_SB.TST2
ACPI Debug:  "Hello world, it is a wonderful day"
```

## [Concatenate](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#concatenate-concatenate-data)
可以拼接Buffer或String。
```c
Concatenate ( Source1, Source2, Result )  => Buffer or String
```

示例如下：
```c
    Name (STR1, "it is a wonderful day")
    Name (STR2, " - Aaron Li.")
    Method (TST3) {
      Store (Concatenate ("Hello world, ", STR1), Local0)
      Store (Concatenate (Local0, STR2), Local1)
      Printf ("%o", Local1)
    }
```
执行结果：
```bash
- execute \_SB.TST3
ACPI Debug:  "Hello world, it is a wonderful day - Aaron Li."
```

> 注：Printf其实是如下的Marco，`Store(xxx, Debug)` 即输出到debug console，参见[Debug](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#debug-debugger-output)。
```c
Printf ("%o: Unexpected value for %o, %o at line %o", Arg0, Arg1, Arg2, Arg3)
// This Printf macro expression evaluates to the following ASL operation:
Store (Concatenate (Concatenate (Concatenate (Concatenate
      (Concatenate (Concatenate (Concatenate ("", Arg0),
      ": Unexpected value for "), Arg1), ", "), Arg2),
      " at line "), Arg3), Debug)
```

> 注2：Concatenate拼接不同类型时会导致隐式转换，详见[表格](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#concatenate-data-types)。


## [ToUUID](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#touuid-convert-string-to-uuid-macro)
将UUID字符串转换成128-Bit Buffer。
```c
ToUUID (AsciiString)  => Buffer
```
示例如下：
```c
    Method (TST4) {
      Name (UID1, Buffer() {
        0x33, 0x22, 0x11, 0x00,
        0x55, 0x44,
        0x77, 0x66,
        0x88, 0x99, 0xAA, 0xBB, 0xCC, 0xDD, 0xEE, 0xFF
      })
      If(LEqual(UID1, ToUUID("00112233-4455-6677-8899-AABBCCDDEEFF"))){
        Printf ("Same UUID")
      } Else {
        Printf ("Not Same UUID")
      }
    }
```
执行结果：
```bash
- execute \_SB.TST4
ACPI Debug:  "Same UUID"
```

## [Index](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#index-indexed-reference-to-member-object)
上一篇文章简单提到了`Index`，这里重点说下`Index`返回值的问题，以便讲后面的`Derefof`。
```c
Index (Source, Index, Destination) => ObjectReference
```
`Index`的返回值均为`ObjectReference`类型：
1. 当`Source`为`Buffer`类型时，`Index`返回Buffer中第n个byte的`Buffer Filed`的Reference.
2. 当`Source`为`String`类型时，`Index`返回String中第n个character的`Buffer Filed`的Reference.
3. 当`Source`为`Package`类型时，`Index`返回Package中第n个`Object`的Reference.



## [Derefof](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#derefof-dereference-an-object-reference)
```c
DerefOf (Source) => Object // Source is an object reference
```

`Derefof`类似于C语言中的`*`，但如果按照ASL+的写法实际上是容易和C语言混淆的。
```c
// 类似于C的 (Arg0 + 4)
Index(Arg0, 4)           // Return Reference of the 4th buffer field of Arg0
Arg0[4]                  // Return Reference of the 4th buffer field of Arg0 (ASL+)
// 类似于C的 *(Arg0 + 4) 或 Arg0[4]
Derefof(Index(Arg0, 4))  // Return the 4th buffer field of Arg0
Derefof(Arg0[4])         // Return the 4th buffer field of Arg0 (ASL+)
```

示例如下：
```c
    Method (TST8, 1)
    {
      Store (Sizeof (Arg0), Local0)
      Store (Zero, Local1) // use this as the index value
      Printf ("The size of the buffer is %o", Local0)
      While (LLess(Local1, Local0)) { // Local1 < Local0
        Printf ("%o", Index (Arg0, Local1))
        Printf ("%o", Derefof (Index (Arg0, Local1)))
        Increment (Local1)
      }
    }
```
执行结果：
```bash
- execute \_SB.TST8 (00, 01, 02, 03, 04, 05)
ACPI Debug:  "The size of the buffer is 0000000000000006"
ACPI Debug:  "[Reference Object]"
ACPI Debug:  "0000000000000000"
ACPI Debug:  "[Reference Object]"
ACPI Debug:  "0000000000000001"
ACPI Debug:  "[Reference Object]"
ACPI Debug:  "0000000000000002"
ACPI Debug:  "[Reference Object]"
ACPI Debug:  "0000000000000003"
ACPI Debug:  "[Reference Object]"
ACPI Debug:  "0000000000000004"
ACPI Debug:  "[Reference Object]"
ACPI Debug:  "0000000000000005"
```

> 注：数组越界会直接导致程序中断

示例如下：
```c
    Method (TST9, 1)
    {
      Store (Sizeof (Arg0), Local0)
      Store (Zero, Local1) // use this as the index value
      Printf ("The size of the buffer is %o", Local0)
      While (LLessEqual (Local1, Local0)) { // Local1 <= Local0, it would cause overflow.
        Printf ("%o", Derefof (Index (Arg0, Local1)))
        Increment (Local1)
      }
    }
```
执行结果：
```bash
- execute \_SB.TST9 (00, 01, 02, 03, 04, 05)
Evaluating \_SB.TST9
ACPI Debug:  "The size of the buffer is 0000000000000006"
ACPI Debug:  "0000000000000000"
ACPI Debug:  "0000000000000001"
ACPI Debug:  "0000000000000002"
ACPI Debug:  "0000000000000003"
ACPI Debug:  "0000000000000004"
ACPI Debug:  "0000000000000005"
3ACPI BIOS Error (bug): AE_AML_BUFFER_LIMIT, Index (0x000000006) is beyond end of object (length 0x6) (20220331/exoparg2-393)
3ACPI Error: Aborting method \_SB.TST9 due to previous error (AE_AML_BUFFER_LIMIT) (20220331/psparse-529)
```

## [CreateWordField](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#createwordfield-create-16-bit-buffer-field)

给Buffer中某段偏移命名，生成一个16-Bit的Buffer Field。

> 注：`Derefof(Index())`生成的Buffer Field都是8-Bit。

```c
CreateWordField (SourceBuffer, ByteIndex, FieldName)
```

示例如下：（如下两个Method实现相同功能）
```c
    Name (BUF1, buffer() {0xff, 0x2f, 0xea, 0x5c})
    CreateWordField (BUF1, 0x01, WRD1)
    //XCH1 and XCH2 will assign 0x00 to BUFF[1] and BUFF[2]
    Method (TST5)
    {
      Store (0x01, Index(BUF1, 1))
      Store (0x02, Index(BUF1, 2))
      Return (BUF1)
    }
    Method (TST6) // Index operator is not needed
    {
      Store (0x0201, WRD1)
      Return (BUF1)
    }
```
![CreateWordField](/assets/images/2023-04-10-asl-code-intro2/CreateWordField.png)

执行结果：
```bash
- debug \_SB.TST5
Evaluating \_SB.TST5
ArgObj:  000000006a91ef18 <Node>          Name BUF1 Buffer(4) FF 2F EA 5C
ResultObj: 00000000aeb9b579 <Obj>           Buffer(4) FF 2F EA 5C # Buffer 初始值
AML Opcode: 0088 Index
...
AML Opcode: 0070 Store
...
ResultObj: 00000000aeb9b579 <Obj>           Buffer(4) FF 01 EA 5C # Store(0x01, Index(BUF1, 1)) 执行后
AML Opcode: 0088 Index
...
AML Opcode: 0070 Store
...
ResultObj: 00000000aeb9b579 <Obj>           Buffer(4) FF 01 02 5C # Store(0x02, Index(BUF1, 2)) 执行后
...


- debug \_SB.TST6
Evaluating \_SB.TST6
AML Opcode: 0070 Store
%
ArgObj:  0000000034fbfb56 <Node>          Name WRD1 BufferField 000000003d1c7d53
ArgObj:  000000009e794f9b <Obj>           Integer 0000000000000201
ResultObj: 000000009e794f9b <Obj>           Integer 0000000000000201
ArgObj:  000000006a91ef18 <Node>          Name BUF1 Buffer(4) FF 01 02 5C
ResultObj: 00000000aeb9b579 <Obj>           Buffer(4) FF 01 02 5C # Store (0x0201, WRD1) 执行后
...
```

> 注：除了`CreateWordField`以外，还有如下不同bit长度的关键字
> 1. [CreateBitField](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#createbitfield-create-1-bit-buffer-field)    Length: 1 bit
> 2. [CreateByteField](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#createbytefield-create-8-bit-buffer-field)   Length: 1 byte
> 3. [CreateWordField](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#createwordfield-create-16-bit-buffer-field)   Length: 2 bytes
> 4. [CreateDWordField](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#createdwordfield-create-32-bit-buffer-field)  Length: 4 bytes
> 5. [CreateQWordField](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#createqwordfield-create-64-bit-buffer-field)  Length: 8 bytes
> 6. [CreateField](https://uefi.org/specs/ACPI/6.5/19_ASL_Reference.html#createfield-create-arbitrary-length-buffer-field)       Length: 可变

---

后面还没整理完，下篇文章接着写。
