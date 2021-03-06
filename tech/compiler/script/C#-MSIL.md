# C# 的 MSIL 简介

## 1. 前言

### 1.1. `MSIL` 是什么？

 `Microsoft Intermediate Language` （MSIL）：微软中间语言。

### 1.2. `C#` 代码编译过程？

`C#` 源代码通过 `LC` (Language Complier) 转为 `IL` 代码，`IL` 主要包含一些元数据和中间语言指令；

`JIT 编译器` 把 `IL` 代码转为物理 `CPU` 能识别的机器代码。

如下图：

![C# IL 编译流程图](./images/cs-msil/cs_msil_workflow_chart.png)

* `Language Complier`（语言编译器）的作用：无论是 `VB code` 还是 `C# code` 都会被 `Language Compiler` 转换为 `MSIL`。

* `MSIL`（微软中间语言）的作用：`MSIL` 包含一些元数据和中间语言指令。

* `JIT Complier`（JIT编译器）的作用：根据系统环境将 `MSIL` 中间语言指令转换为机器码（`Native Code`）。

## 2. MSIL 的框架

![C# IL 编译流程图](./images/cs-msil/MSIL_framework.jpg)

* `Managed Heap`（托管堆）：这就是 `.NET` 中的托管堆，用来存放引用类型，它是由 `GC`（垃圾回收器自动进行回收）管理；

* `Evaluation Stack`（计算堆栈）：每个线程都有自己的线程栈，`IL` 里面的任何计算，都发生在 `Evaluation Stack` 上，其实就是一个 `Stack` 结构。可以 `Push`，也可以 `Pop`。

* `Call Stack`（调用堆栈）：也是一个 `Stack` 结构，调用堆栈保存了被调用函数的参数列表（`Argument List`），还保存了调用返回信息，以及被调用函数的局部变量（`Local Vars`），非常类似于物理 `CPU` 上的 `Call Stack`，只不过没有相应的 `寄存器` 变量。

上图的出处：[https://msdn.microsoft.com/zh-tw/library/dd229210.aspx](https://msdn.microsoft.com/zh-tw/library/dd229210.aspx)

## 3. IL 指令

### 3.1. 示例代码

用 `C#` 写一个简单控制台应用程序，代码如下：

```csharp
using System;

namespace ILDemo
{
    class Program
    {
        static void Main(string[] args)
        {
            int i = 1;
            int j = 2;
            int k = 3;
            int answer = i + j + k;
            Console.WriteLine("i + j + k = " + answer);
            Console.ReadKey();
        }
    }
}
```

### 3.2. 反编译

用 `ILDasm` 反编译上面代码的 `exe` 文件后得到的 `IL` 汇编如下：

```c#
.method private hidebysig static void Main(string[] args) cil managed
{
  .entrypoint   // 程序入口
  // 这个程序的的代码大小   42字节 (0x2a)
  .maxstack  2  // 计算堆栈(Evaluation Stack)的大小

  .locals init (
      [0] int32 i,
      [1] int32 j,
      [2] int32 k,
      [3] int32 answer
  ) // 定义 int32 类型的 i, j, k, answer

  IL_0000:  nop       // 无操作

  IL_0001:  ldc.i4.1  // 把整数值 1 (常量), 即 i = 1 压入计算堆栈最顶端, 相当于 push 1 到计算堆栈上
  IL_0002:  stloc.0   // 把计算堆栈顶部的值( i 的值)放到调用堆栈索引 0 处
  IL_0003:  ldc.i4.2  // 把整数值 2 (常量), 即 j = 2 压入计算堆栈最顶端, 相当于 push 2 到计算堆栈上
  IL_0004:  stloc.1   // 把计算堆栈顶部的值( j 的值)放到调用堆栈索引 1 处
  IL_0005:  ldc.i4.3  // 把整数值 3 (常量), 即 k = 3 压入计算堆栈最顶端, 相当于 push 3 到计算堆栈上
  IL_0006:  stloc.2   // 把计算堆栈顶部的值( k 的值)放到调用堆栈索引 2 处

  IL_0007:  ldloc.0   // 把调用堆栈索引为 0 处的值复制到计算堆栈上
  IL_0008:  ldloc.1   // 把调用堆栈索引为 1 处的值复制到计算堆栈上
  IL_0009:  add       // 相加, 即 (i + j)
  IL_000a:  ldloc.2   // 把调用堆栈索引为 2 处的值复制到计算堆栈上
  IL_000b:  add       // 相加, 即 ((i + j) + k)
  IL_000c:  stloc.3   // 把计算堆栈顶部的值(add 的值)放到调用堆栈索引 3 处, 即 (i + j + k)
  IL_000d:  ldstr ""i + j + k = ""  // 推送对元数据中存储的字符串的新对象引用。
  IL_0012:  ldloc.3   // 把调用堆栈索引为 3 处的值复制到计算堆栈上

  IL_0013:  box        [mscorlib]System.Int32     // 装箱
  IL_0018:  call       string [mscorlib]System.String::Concat(object, object)  // 调用内部方法
  IL_001d:  call       void [mscorlib]System.Console::WriteLine(string)        // 调用 WriteLine
  IL_0022:  nop        // 无操作
  IL_0023:  call       valuetype [mscorlib]System.ConsoleKeyInfo [mscorlib]System.Console::ReadKey()  // 调用 ConsoleKey
  IL_0028:  pop        // 弹出堆栈(对计算堆栈操作)
  IL_0029:  ret        // return, 返回
}
// end of method Program::Main
```

总结一下常用的 `IL` 指令：

* `.entrypoint`：指令表示 `CLR` 加载程序时，是首先从 `.entrypoint` 开始的，即从 `Main` 方法作为程序的入口函数；
* `stloc.X`：把计算堆栈顶部的值放到调用堆栈索引为 `X` 处的位置；
* `ldloc.X`：把调用堆栈 `X` 处位置的值复制到计算堆栈上；
* `.newobj`：用于创建引用类型的对象；
* `.ldstr`：用于创建 `String` 对象变量；
* `.newarr`：用于创建数组型对象；
* `.box`：在值类型转换为引用类型的对象时，将值类型拷贝至托管堆上分配内存；
* `.assembly`：指令告诉编译器，我们准备去用一个外部的类库（不是我们自己写的，而是提前编译好的）。

**关于 .maxstack 的理解**

上面的 `IL` 汇编代码中，看起来好像最多用到了 `3` 个计算堆栈的寄存器位置，但为什么 `.maxstack` 的值却是 `2` 呢？

这是因为，实际上，上面的 `IL` 代码中最多只用到了两个计算堆栈的寄存器位置。比如，一开始虽然“`ldc.i4.1`”，“`ldc.i4.2`”，“`ldc.i4.3`”指令分别加载了 `3` 个常量值到计算堆栈上，但每一个加载指令后面都有一条 `stloc.X` 指令，该指令把计算堆栈顶端的值写回到调用堆栈上了，下一条 `ldc.i4.X` 指令加载的值是可以覆盖原来调用堆栈最顶端的寄存器的，所以最多只需要一个计算堆栈的寄存器就可以了。

再看之后的两条 `add` 指令。第一条 `add` 指令，会对计算堆栈最顶端的两个 `int32` 的寄存器相加，并且把相加后的值写回到计算堆栈的最顶端。然后，第二条 `add` 指令之前，`ldloc.2` 把调用堆栈上的索引 `2` 位置的值复制到了计算堆栈上，同时，前面 `i + j` 相加后的值会被挤到第二位，第二条 `add` 指令还是对计算堆栈最顶端的两个 `int32` 的寄存器相加。所以，最多只需要两个计算堆栈的寄存器位置即可，后面的代码分析也同样不超过 `2`。

所以，对于上面的 `IL` 汇编代码，`.maxstack` 的值是 `2` 。

## 4. IL指令大全

-------------------------------------------------------------------------------
| 名称          | 说明                                 |
| ------------- | ------------------------------------ |
| Add           | 将两个值相加并将结果推送到计算堆栈上。|
| Add.Ovf       | 将两个整数相加，执行溢出检查，并且将结果推送到计算堆栈上。|
| Add.Ovf.Un    | 将两个无符号整数值相加，执行溢出检查，并且将结果推送到计算堆栈上。|
| And           | 计算两个值的按位“与”并将结果推送到计算堆栈上。|
| Arglist       | 返回指向当前方法的参数列表的非托管指针。|
| Beq           | 如果两个值相等，则将控制转移到目标指令。|
| Beq.S         | 如果两个值相等，则将控制转移到目标指令（短格式）。|
| Bge           | 如果第一个值大于或等于第二个值，则将控制转移到目标指令。|
| Bge.S         | 如果第一个值大于或等于第二个值，则将控制转移到目标指令（短格式）。|
| Bge.Un        | 当比较无符号整数值或不可排序的浮点型值时，如果第一个值大于第二个值，则将控制转移到目标指令。|
| Bge.Un.S      | 当比较无符号整数值或不可排序的浮点型值时，如果第一个值大于第二个值，则将控制转移到目标指令（短格式）。|
| Bgt           | 如果第一个值大于第二个值，则将控制转移到目标指令。|
| Bgt.S         | 如果第一个值大于第二个值，则将控制转移到目标指令（短格式）。|
| Bgt.Un        | 当比较无符号整数值或不可排序的浮点型值时，如果第一个值大于第二个值，则将控制转移到目标指令。|
| Bgt.Un.S      | 当比较无符号整数值或不可排序的浮点型值时，如果第一个值大于第二个值，则将控制转移到目标指令（短格式）。|
| Ble           | 如果第一个值小于或等于第二个值，则将控制转移到目标指令。|
| Ble.S         | 如果第一个值小于或等于第二个值，则将控制转移到目标指令（短格式）。|
| Ble.Un        | 当比较无符号整数值或不可排序的浮点型值时，如果第一个值小于或等于第二个值，则将控制转移到目标指令。|
| Ble.Un.S      | 当比较无符号整数值或不可排序的浮点值时，如果第一个值小于或等于第二个值，则将控制权转移到目标指令（短格式）。|
| Blt           | 如果第一个值小于第二个值，则将控制转移到目标指令。|
| Blt.S         | 如果第一个值小于第二个值，则将控制转移到目标指令（短格式）。|
| Blt.Un        | 当比较无符号整数值或不可排序的浮点型值时，如果第一个值小于第二个值，则将控制转移到目标指令。|
| Blt.Un.S      | 当比较无符号整数值或不可排序的浮点型值时，如果第一个值小于第二个值，则将控制转移到目标指令（短格式）。|
| Bne.Un        | 当两个无符号整数值或不可排序的浮点型值不相等时，将控制转移到目标指令。|
| Bne.Un.S      | 当两个无符号整数值或不可排序的浮点型值不相等时，将控制转移到目标指令（短格式）。|
| Box           | 将值类转换为对象引用（O 类型）。|
| Br            | 无条件地将控制转移到目标指令。|
| Br.S          | 无条件地将控制转移到目标指令（短格式）。|
| Break         | 向公共语言结构 (CLI) 发出信号以通知调试器已撞上了一个断点。|
| Brfalse       | 如果 value 为 false、空引用（Visual Basic 中的 Nothing）或零，则将控制转移到目标指令。|
| Brfalse.S     | 如果 value 为 false、空引用或零，则将控制转移到目标指令。|
| Brtrue        | 如果 value 为 true、非空或非零，则将控制转移到目标指令。|
| Brtrue.S      | 如果 value 为 true、非空或非零，则将控制转移到目标指令（短格式）。|
| Call          | 调用由传递的方法说明符指示的方法。|
| Calli         | 通过调用约定描述的参数调用在计算堆栈上指示的方法（作为指向入口点的指针）。|
| Callvirt      | 对对象调用后期绑定方法，并且将返回值推送到计算堆栈上。|
| Castclass     | 尝试将引用传递的对象转换为指定的类。|
| Ceq           | 比较两个值。如果这两个值相等，则将整数值 1 (int32) 推送到计算堆栈上；否则，将 0 (int32) 推送到计算堆栈上。|
| Cgt           | 比较两个值。如果第一个值大于第二个值，则将整数值 1 (int32) 推送到计算堆栈上；反之，将 0 (int32) 推送到计算堆栈上。|
| Cgt.Un        | 比较两个无符号的或不可排序的值。如果第一个值大于第二个值，则将整数值 1 (int32) 推送到计算堆栈上；反之，将 0 (int32) 推送到计算堆栈上。|
| Ckfinite      | 如果值不是有限数，则引发 ArithmeticException。|
| Clt           | 比较两个值。如果第一个值小于第二个值，则将整数值 1 (int32) 推送到计算堆栈上；反之，将 0 (int32) 推送到计算堆栈上。|
| Clt.Un        | 比较无符号的或不可排序的值 value1 和 value2。如果 value1 小于 value2，则将整数值 1 (int32 ) 推送到计算堆栈上；反之，将 0 ( int32 ) 推送到计算堆栈上。|
| Constrained   | 约束要对其进行虚方法调用的类型。|
| Conv.I        | 将位于计算堆栈顶部的值转换为 native int。|
| Conv.I1       | 将位于计算堆栈顶部的值转换为 int8，然后将其扩展（填充）为 int32。|
| Conv.I2       | 将位于计算堆栈顶部的值转换为 int16，然后将其扩展（填充）为 int32。|
| Conv.I4       | 将位于计算堆栈顶部的值转换为 int32。|
| Conv.I8       | 将位于计算堆栈顶部的值转换为 int64。|
| Conv.Ovf.I    | 将位于计算堆栈顶部的有符号值转换为有符号 native int，并在溢出时引发 OverflowException。|
| Conv.Ovf.I.Un | 将位于计算堆栈顶部的无符号值转换为有符号 native int，并在溢出时引发 OverflowException。|
| Conv.Ovf.I1   | 将位于计算堆栈顶部的有符号值转换为有符号 int8 并将其扩展为 int32，并在溢出时引发 OverflowException。|
| Conv.Ovf.I1.Un | 将位于计算堆栈顶部的无符号值转换为有符号 int8 并将其扩展为 int32，并在溢出时引发 OverflowException。|
| Conv.Ovf.I2   | 将位于计算堆栈顶部的有符号值转换为有符号 int16 并将其扩展为 int32，并在溢出时引发 OverflowException。|
| Conv.Ovf.I2.Un | 将位于计算堆栈顶部的无符号值转换为有符号 int16 并将其扩展为 int32，并在溢出时引发 OverflowException。|
| Conv.Ovf.I4   | 将位于计算堆栈顶部的有符号值转换为有符号 int32，并在溢出时引发 OverflowException。|
| Conv.Ovf.I4.Un | 将位于计算堆栈顶部的无符号值转换为有符号 int32，并在溢出时引发 OverflowException。|
| Conv.Ovf.I8   | 将位于计算堆栈顶部的有符号值转换为有符号 int64，并在溢出时引发 OverflowException。|
| Conv.Ovf.I8.Un | 将位于计算堆栈顶部的无符号值转换为有符号 int64，并在溢出时引发 OverflowException。|
| Conv.Ovf.U    | 将位于计算堆栈顶部的有符号值转换为 unsigned native int，并在溢出时引发 OverflowException。|
| Conv.Ovf.U.Un | 将位于计算堆栈顶部的无符号值转换为 unsigned native int，并在溢出时引发 OverflowException。|
| Conv.Ovf.U1   | 将位于计算堆栈顶部的有符号值转换为 unsigned int8 并将其扩展为 int32，并在溢出时引发 OverflowException。|
| Conv.Ovf.U1.Un | 将位于计算堆栈顶部的无符号值转换为 unsigned int8 并将其扩展为 int32，并在溢出时引发 OverflowException。|
| Conv.Ovf.U2   | 将位于计算堆栈顶部的有符号值转换为 unsigned int16 并将其扩展为 int32，并在溢出时引发 OverflowException。|
| Conv.Ovf.U2.Un | 将位于计算堆栈顶部的无符号值转换为 unsigned int16 并将其扩展为 int32，并在溢出时引发 OverflowException。|
| Conv.Ovf.U4   | 将位于计算堆栈顶部的有符号值转换为 unsigned int32，并在溢出时引发 OverflowException。|
| Conv.Ovf.U4.Un | 将位于计算堆栈顶部的无符号值转换为 unsigned int32，并在溢出时引发 OverflowException。|
| Conv.Ovf.U8   | 将位于计算堆栈顶部的有符号值转换为 unsigned int64，并在溢出时引发 OverflowException。|
| Conv.Ovf.U8.Un | 将位于计算堆栈顶部的无符号值转换为 unsigned int64，并在溢出时引发 OverflowException。|
| Conv.R.Un     | 将位于计算堆栈顶部的无符号整数值转换为 float32。|
| Conv.R4       | 将位于计算堆栈顶部的值转换为 float32。|
| Conv.R8       | 将位于计算堆栈顶部的值转换为 float64。|
| Conv.U        | 将位于计算堆栈顶部的值转换为 unsigned native int，然后将其扩展为 native int。|
| Conv.U1       | 将位于计算堆栈顶部的值转换为 unsigned int8，然后将其扩展为 int32。|
| Conv.U2       | 将位于计算堆栈顶部的值转换为 unsigned int16，然后将其扩展为 int32。|
| Conv.U4       | 将位于计算堆栈顶部的值转换为 unsigned int32，然后将其扩展为 int32。|
| Conv.U8       | 将位于计算堆栈顶部的值转换为 unsigned int64，然后将其扩展为 int64。|
| Cpblk         | 将指定数目的字节从源地址复制到目标地址。|
| Cpobj         | 将位于对象（&、* 或 native int 类型）地址的值类型复制到目标对象（&、* 或 native int 类型）的地址。|
| Div           | 将两个值相除并将结果作为浮点（F 类型）或商（int32 类型）推送到计算堆栈上。|
| Div.Un        | 两个无符号整数值相除并将结果 ( int32 ) 推送到计算堆栈上。|
| Dup           | 复制计算堆栈上当前最顶端的值，然后将副本推送到计算堆栈上。|
| Endfilter     | 将控制从异常的 filter 子句转移回公共语言结构 (CLI) 异常处理程序。|
| Endfinally    | 将控制从异常块的 fault 或 finally 子句转移回公共语言结构 (CLI) 异常处理程序。|
| Initblk       | 将位于特定地址的内存的指定块初始化为给定大小和初始值。|
| Initobj       | 将位于指定地址的值类型的每个字段初始化为空引用或适当的基元类型的 0。|
| Isinst        | 测试对象引用（O 类型）是否为特定类的实例。|
| Jmp           | 退出当前方法并跳至指定方法。|
| Ldarg         | 将参数（由指定索引值引用）加载到堆栈上。|
| Ldarg.0       | 将索引为 0 的参数加载到计算堆栈上。|
| Ldarg.1       | 将索引为 1 的参数加载到计算堆栈上。|
| Ldarg.2       | 将索引为 2 的参数加载到计算堆栈上。|
| Ldarg.3       | 将索引为 3 的参数加载到计算堆栈上。|
| Ldarg.S       | 将参数（由指定的短格式索引引用）加载到计算堆栈上。|
| Ldarga        | 将参数地址加载到计算堆栈上。|
| Ldarga.S      | 以短格式将参数地址加载到计算堆栈上。|
| Ldc.I4        | 将所提供的 int32 类型的值作为 int32 推送到计算堆栈上。|
| Ldc.I4.0      | 将整数值 0 作为 int32 推送到计算堆栈上。|
| Ldc.I4.1      | 将整数值 1 作为 int32 推送到计算堆栈上。|
| Ldc.I4.2      | 将整数值 2 作为 int32 推送到计算堆栈上。|
| Ldc.I4.3      | 将整数值 3 作为 int32 推送到计算堆栈上。|
| Ldc.I4.4      | 将整数值 4 作为 int32 推送到计算堆栈上。|
| Ldc.I4.5      | 将整数值 5 作为 int32 推送到计算堆栈上。|
| Ldc.I4.6      | 将整数值 6 作为 int32 推送到计算堆栈上。|
| Ldc.I4.7      | 将整数值 7 作为 int32 推送到计算堆栈上。|
| Ldc.I4.8      | 将整数值 8 作为 int32 推送到计算堆栈上。|
| Ldc.I4.M1     | 将整数值 -1 作为 int32 推送到计算堆栈上。|
| Ldc.I4.S      | 将提供的 int8 值作为 int32 推送到计算堆栈上（短格式）。|
| Ldc.I8        | 将所提供的 int64 类型的值作为 int64 推送到计算堆栈上。|
| Ldc.R4        | 将所提供的 float32 类型的值作为 F (float) 类型推送到计算堆栈上。|
| Ldc.R8        | 将所提供的 float64 类型的值作为 F (float) 类型推送到计算堆栈上。|
| Ldelem        | 按照指令中指定的类型，将指定数组索引中的元素加载到计算堆栈的顶部。|
| Ldelem.I      | 将位于指定数组索引处的 native int 类型的元素作为 native int 加载到计算堆栈的顶部。|
| Ldelem.I1     | 将位于指定数组索引处的 int8 类型的元素作为 int32 加载到计算堆栈的顶部。|
| Ldelem.I2     | 将位于指定数组索引处的 int16 类型的元素作为 int32 加载到计算堆栈的顶部。|
| Ldelem.I4     | 将位于指定数组索引处的 int32 类型的元素作为 int32 加载到计算堆栈的顶部。|
| Ldelem.I8     | 将位于指定数组索引处的 int64 类型的元素作为 int64 加载到计算堆栈的顶部。|
| Ldelem.R4     | 将位于指定数组索引处的 float32 类型的元素作为 F 类型（浮点型）加载到计算堆栈的顶部。|
| Ldelem.R8     | 将位于指定数组索引处的 float64 类型的元素作为 F 类型（浮点型）加载到计算堆栈的顶部。|
| Ldelem.Ref    | 将位于指定数组索引处的包含对象引用的元素作为 O 类型（对象引用）加载到计算堆栈的顶部。|
| Ldelem.U1     | 将位于指定数组索引处的 unsigned int8 类型的元素作为 int32 加载到计算堆栈的顶部。|
| Ldelem.U2     | 将位于指定数组索引处的 unsigned int16 类型的元素作为 int32 加载到计算堆栈的顶部。|
| Ldelem.U4     | 将位于指定数组索引处的 unsigned int32 类型的元素作为 int32 加载到计算堆栈的顶部。|
| Ldelema       | 将位于指定数组索引的数组元素的地址作为 & 类型（托管指针）加载到计算堆栈的顶部。|
| Ldfld         | 查找对象中其引用当前位于计算堆栈的字段的值。|
| Ldflda        | 查找对象中其引用当前位于计算堆栈的字段的地址。|
| Ldftn         | 将指向实现特定方法的本机代码的非托管指针（native int 类型）推送到计算堆栈上。|
| Ldind.I       | 将 native int 类型的值作为 native int 间接加载到计算堆栈上。|
| Ldind.I1      | 将 int8 类型的值作为 int32 间接加载到计算堆栈上。|
| Ldind.I2      | 将 int16 类型的值作为 int32 间接加载到计算堆栈上。|
| Ldind.I4      | 将 int32 类型的值作为 int32 间接加载到计算堆栈上。|
| Ldind.I8      | 将 int64 类型的值作为 int64 间接加载到计算堆栈上。|
| Ldind.R4      | 将 float32 类型的值作为 F (float) 类型间接加载到计算堆栈上。|
| Ldind.R8      | 将 float64 类型的值作为 F (float) 类型间接加载到计算堆栈上。|
| Ldind.Ref     | 将对象引用作为 O（对象引用）类型间接加载到计算堆栈上。|
| Ldind.U1      | 将 unsigned int8 类型的值作为 int32 间接加载到计算堆栈上。|
| Ldind.U2      | 将 unsigned int16 类型的值作为 int32 间接加载到计算堆栈上。|
| Ldind.U4      | 将 unsigned int32 类型的值作为 int32 间接加载到计算堆栈上。|
| Ldlen         | 将从零开始的、一维数组的元素的数目推送到计算堆栈上。|
| Ldloc         | 将指定索引处的局部变量加载到计算堆栈上。|
| Ldloc.0       | 将索引 0 处的局部变量加载到计算堆栈上。|
| Ldloc.1       | 将索引 1 处的局部变量加载到计算堆栈上。|
| Ldloc.2       | 将索引 2 处的局部变量加载到计算堆栈上。|
| Ldloc.3       | 将索引 3 处的局部变量加载到计算堆栈上。|
| Ldloc.S       | 将特定索引处的局部变量加载到计算堆栈上（短格式）。|
| Ldloca        | 将位于特定索引处的局部变量的地址加载到计算堆栈上。|
| Ldloca.S      | 将位于特定索引处的局部变量的地址加载到计算堆栈上（短格式）。|
| Ldnull        | 将空引用（O 类型）推送到计算堆栈上。|
| Ldobj         | 将地址指向的值类型对象复制到计算堆栈的顶部。|
| Ldsfld        | 将静态字段的值推送到计算堆栈上。|
| Ldsflda       | 将静态字段的地址推送到计算堆栈上。|
| Ldstr         | 推送对元数据中存储的字符串的新对象引用。|
| Ldtoken       | 将元数据标记转换为其运行时表示形式，并将其推送到计算堆栈上。|
| Ldvirtftn     | 将指向实现与指定对象关联的特定虚方法的本机代码的非托管指针（native int 类型）推送到计算堆栈上。|
| Leave         | 退出受保护的代码区域，无条件将控制转移到特定目标指令。|
| Leave.S       | 退出受保护的代码区域，无条件将控制转移到目标指令（缩写形式）。|
| Localloc      | 从本地动态内存池分配特定数目的字节并将第一个分配的字节的地址（瞬态指针，* 类型）推送到计算堆栈上。|
| Mkrefany      | 将对特定类型实例的类型化引用推送到计算堆栈上。|
| Mul           | 将两个值相乘并将结果推送到计算堆栈上。|
| Mul.Ovf       | 将两个整数值相乘，执行溢出检查，并将结果推送到计算堆栈上。|
| Mul.Ovf.Un    | 将两个无符号整数值相乘，执行溢出检查，并将结果推送到计算堆栈上。|
| Neg           | 对一个值执行求反并将结果推送到计算堆栈上。|
| Newarr        | 将对新的从零开始的一维数组（其元素属于特定类型）的对象引用推送到计算堆栈上。|
| Newobj        | 创建一个值类型的新对象或新实例，并将对象引用（O 类型）推送到计算堆栈上。|
| Nop           | 如果修补操作码，则填充空间。尽管可能消耗处理周期，但未执行任何有意义的操作。|
| Not           | 计算堆栈顶部整数值的按位求补并将结果作为相同的类型推送到计算堆栈上。|
| Or            | 计算位于堆栈顶部的两个整数值的按位求补并将结果推送到计算堆栈上。|
| Pop           | 移除当前位于计算堆栈顶部的值。|
| Prefix1       | 基础结构。此指令为保留指令。|
| Prefix2       | 基础结构。此指令为保留指令。|
| Prefix3       | 基础结构。此指令为保留指令。|
| Prefix4       | 基础结构。此指令为保留指令。|
| Prefix5       | 基础结构。此指令为保留指令。|
| Prefix6       | 基础结构。此指令为保留指令。|
| Prefix7       | 基础结构。此指令为保留指令。|
| Prefixref     | 基础结构。此指令为保留指令。|
| Readonly      | 指定后面的数组地址操作在运行时不执行类型检查，并且返回可变性受限的托管指针。|
| Refanytype    | 检索嵌入在类型化引用内的类型标记。|
| Refanyval     | 检索嵌入在类型化引用内的地址（& 类型）。|
| Rem           | 将两个值相除并将余数推送到计算堆栈上。|
| Rem.Un        | 将两个无符号值相除并将余数推送到计算堆栈上。|
| Ret           | 从当前方法返回，并将返回值（如果存在）从调用方的计算堆栈推送到被调用方的计算堆栈上。|
| Rethrow       | 再次引发当前异常。|
| Shl           | 将整数值左移（用零填充）指定的位数，并将结果推送到计算堆栈上。|
| Shr           | 将整数值右移（保留符号）指定的位数，并将结果推送到计算堆栈上。|
| Shr.Un        | 将无符号整数值右移（用零填充）指定的位数，并将结果推送到计算堆栈上。|
| Sizeof        | 将提供的值类型的大小（以字节为单位）推送到计算堆栈上。|
| Starg         | 将位于计算堆栈顶部的值存储到位于指定索引的参数槽中。|
| Starg.S       | 将位于计算堆栈顶部的值存储在参数槽中的指定索引处（短格式）。|
| Stelem        | 用计算堆栈中的值替换给定索引处的数组元素，其类型在指令中指定。|
| Stelem.I      | 用计算堆栈上的 native int 值替换给定索引处的数组元素。|
| Stelem.I1     | 用计算堆栈上的 int8 值替换给定索引处的数组元素。|
| Stelem.I2     | 用计算堆栈上的 int16 值替换给定索引处的数组元素。|
| Stelem.I4     | 用计算堆栈上的 int32 值替换给定索引处的数组元素。|
| Stelem.I8     | 用计算堆栈上的 int64 值替换给定索引处的数组元素。|
| Stelem.R4     | 用计算堆栈上的 float32 值替换给定索引处的数组元素。|
| Stelem.R8     | 用计算堆栈上的 float64 值替换给定索引处的数组元素。|
| Stelem.Ref    | 用计算堆栈上的对象 ref 值（O 类型）替换给定索引处的数组元素。|
| Stfld         | 用新值替换在对象引用或指针的字段中存储的值。|
| Stind.I       | 在所提供的地址存储 native int 类型的值。|
| Stind.I1      | 在所提供的地址存储 int8 类型的值。|
| Stind.I2      | 在所提供的地址存储 int16 类型的值。|
| Stind.I4      | 在所提供的地址存储 int32 类型的值。|
| Stind.I8      | 在所提供的地址存储 int64 类型的值。|
| Stind.R4      | 在所提供的地址存储 float32 类型的值。|
| Stind.R8      | 在所提供的地址存储 float64 类型的值。|
| Stind.Ref     | 存储所提供地址处的对象引用值。|
| Stloc         | 从计算堆栈的顶部弹出当前值并将其存储到指定索引处的局部变量列表中。|
| Stloc.0       | 从计算堆栈的顶部弹出当前值并将其存储到索引 0 处的局部变量列表中。|
| Stloc.1       | 从计算堆栈的顶部弹出当前值并将其存储到索引 1 处的局部变量列表中。|
| Stloc.2       | 从计算堆栈的顶部弹出当前值并将其存储到索引 2 处的局部变量列表中。|
| Stloc.3       | 从计算堆栈的顶部弹出当前值并将其存储到索引 3 处的局部变量列表中。|
| Stloc.S       | 从计算堆栈的顶部弹出当前值并将其存储在局部变量列表中的 index 处（短格式）。|
| Stobj         | 将指定类型的值从计算堆栈复制到所提供的内存地址中。|
| Stsfld        | 用来自计算堆栈的值替换静态字段的值。|
| Sub           | 从其他值中减去一个值并将结果推送到计算堆栈上。|
| Sub.Ovf       | 从另一值中减去一个整数值，执行溢出检查，并且将结果推送到计算堆栈上。|
| Sub.Ovf.Un    | 从另一值中减去一个无符号整数值，执行溢出检查，并且将结果推送到计算堆栈上。|
| Switch        | 实现跳转表。|
| Tailcall      | 执行后缀的方法调用指令，以便在执行实际调用指令前移除当前方法的堆栈帧。|
| Throw         | 引发当前位于计算堆栈上的异常对象。|
| Unaligned     | 指示当前位于计算堆栈上的地址可能没有与紧接的 ldind、stind、ldfld、stfld、ldobj、stobj、initblk 或 cpblk 指令的自然大小对齐。|
| Unbox         | 将值类型的已装箱的表示形式转换为其未装箱的形式。|
| Unbox.Any     | 将指令中指定类型的已装箱的表示形式转换成未装箱形式。|
| Volatile      | 指定当前位于计算堆栈顶部的地址可以是易失的，并且读取该位置的结果不能被缓存，或者对该地址的多个存储区不能被取消。|
| Xor           | 计算位于计算堆栈顶部的两个值的按位异或，并且将结果推送到计算堆栈上。|
-------------------------------------------------------------------------------

## 5. 参考文章

1. [`浅谈 C# 中的 IL`](https://blog.csdn.net/neil3611244/article/details/71211398)

    [https://blog.csdn.net/neil3611244/article/details/71211398](https://blog.csdn.net/neil3611244/article/details/71211398)

2. [`C# IL 入门`](https://blog.csdn.net/icebergliu1234/article/details/81322455)

    [https://blog.csdn.net/icebergliu1234/article/details/81322455](https://blog.csdn.net/icebergliu1234/article/details/81322455)

3. [`C# IL DASM 使用`](https://www.cnblogs.com/caokai520/p/4921706.html)

    [https://www.cnblogs.com/caokai520/p/4921706.html](https://www.cnblogs.com/caokai520/p/4921706.html)
