---
ms.openlocfilehash: dbea611280a644adc25247b9887986e129c59b68
ms.sourcegitcommit: a5e393b018b04dfa55aae0000290ca087b508495
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/14/2019
ms.locfileid: "72310360"
---
# <a name="unsafe-code"></a>不安全代码

上述章节C#中定义的核心语言与 C 和C++将指针省略为数据类型的方式不同。 相反， C#提供引用和创建由垃圾回收器管理的对象的功能。 此设计与其他功能C#结合使用比 C 或C++更安全。 在核心C#语言中，不能使用未初始化的变量、"无关联的" 指针或索引超出其界限的数组的表达式。 这样就消除了日常灾难 C 和C++程序的各种 bug。

尽管实际上每个指针类型都是在C++ C 中构造的，或是C#在中具有对应的引用类型，但在某些情况下，访问指针类型是必需的。 例如，在不访问指针的情况下，与基础操作系统、访问内存映射设备或实现时间关键算法之间的交互可能是不可能的或不切实际的。 为了满足这一需求C# ，提供了编写***不安全代码***的功能。

在不安全代码中，可以声明和操作指针、在指针和整型之间执行转换，以获取变量的地址等。 在某种意义上，编写不安全代码非常类似于在C#程序内编写 C 代码。

从开发人员和用户的角度来看，不安全代码实际上是一项 "安全" 功能。 必须使用修饰符 `unsafe`清楚地标记不安全代码，因此开发人员不可能意外地使用不安全的功能，并且执行引擎可以确保不受信任的环境中不能执行不安全的代码。

## <a name="unsafe-contexts"></a>不安全上下文

的不安全功能C#仅在不安全的上下文中可用。 通过在类型或成员的声明中包含 `unsafe` 修饰符或使用*unsafe_statement*，引入了不安全的上下文：

*  类、结构、接口或委托的声明可能包含 `unsafe` 修饰符，在这种情况下，该类型声明的整个文本范围（包括类、结构或接口的正文）被视为不安全的上下文。
*  字段、方法、属性、事件、索引器、运算符、实例构造函数、析构函数或静态构造函数的声明可能包含 `unsafe` 修饰符，在这种情况下，该成员声明的整个文本范围被视为不安全的上下文。
*  *Unsafe_statement*允许在*块*内使用不安全的上下文。 相关*块*的整个文本范围被视为不安全的上下文。

相关的语法生产如下所示。

```antlr
class_modifier_unsafe
    : 'unsafe'
    ;

struct_modifier_unsafe
    : 'unsafe'
    ;

interface_modifier_unsafe
    : 'unsafe'
    ;

delegate_modifier_unsafe
    : 'unsafe'
    ;

field_modifier_unsafe
    : 'unsafe'
    ;

method_modifier_unsafe
    : 'unsafe'
    ;

property_modifier_unsafe
    : 'unsafe'
    ;

event_modifier_unsafe
    : 'unsafe'
    ;

indexer_modifier_unsafe
    : 'unsafe'
    ;

operator_modifier_unsafe
    : 'unsafe'
    ;

constructor_modifier_unsafe
    : 'unsafe'
    ;

destructor_declaration_unsafe
    : attributes? 'extern'? 'unsafe'? '~' identifier '(' ')' destructor_body
    | attributes? 'unsafe'? 'extern'? '~' identifier '(' ')' destructor_body
    ;

static_constructor_modifiers_unsafe
    : 'extern'? 'unsafe'? 'static'
    | 'unsafe'? 'extern'? 'static'
    | 'extern'? 'static' 'unsafe'?
    | 'unsafe'? 'static' 'extern'?
    | 'static' 'extern'? 'unsafe'?
    | 'static' 'unsafe'? 'extern'?
    ;

embedded_statement_unsafe
    : unsafe_statement
    | fixed_statement
    ;

unsafe_statement
    : 'unsafe' block
    ;
```

示例中

```csharp
public unsafe struct Node
{
    public int Value;
    public Node* Left;
    public Node* Right;
}
```

结构声明中指定的 `unsafe` 修饰符导致结构声明的整个文本区成为不安全的上下文。 因此，可以将 `Left` 和 `Right` 字段声明为指针类型。 上面的示例也可以编写

```csharp
public struct Node
{
    public int Value;
    public unsafe Node* Left;
    public unsafe Node* Right;
}
```

此处，字段声明中的 `unsafe` 修饰符将导致这些声明被视为不安全的上下文。

除了建立不安全的上下文，因而允许使用指针类型外，`unsafe` 修饰符对类型或成员不起作用。 示例中

```csharp
public class A
{
    public unsafe virtual void F() {
        char* p;
        ...
    }
}

public class B: A
{
    public override void F() {
        base.F();
        ...
    }
}
```

`A` 的 `F` 方法上的 `unsafe` 修饰符只是导致 `F` 的文本范围成为不安全的上下文，在此上下文中，可以使用该语言的不安全功能。 在 `B`中的 `F` 重写中，无需重新指定 `unsafe` 修饰符--除非当然，`B` 中的 `F` 方法本身需要访问不安全功能。

当指针类型是方法签名的一部分时，情况会略有不同。

```csharp
public unsafe class A
{
    public virtual void F(char* p) {...}
}

public class B: A
{
    public unsafe override void F(char* p) {...}
}
```

此处，因为 `F`的签名包含指针类型，所以只能在不安全的上下文中写入。 但是，不安全上下文可通过以下方式引入：使整个类不安全（如 `A`中的情况），或在方法声明中包含 `unsafe` 修饰符，如 `B`中的情况。

## <a name="pointer-types"></a>指针类型

在不安全的上下文中，*类型*（[类型](types.md)）可能是*pointer_type*以及*value_type*或*reference_type*。 但是，还可以在不安全的上下文之外的 `typeof` 表达式（[匿名对象创建表达式](expressions.md#anonymous-object-creation-expressions)）中使用*pointer_type* ，因为这种用法是不安全的。

```antlr
type_unsafe
    : pointer_type
    ;
```

*Pointer_type*以*unmanaged_type*或关键字 `void`形式编写，后跟 `*` 标记：

```antlr
pointer_type
    : unmanaged_type '*'
    | 'void' '*'
    ;

unmanaged_type
    : type
    ;
```

在指针类型中 `*` 之前指定的类型称为指针类型的***引用类型***。 它表示指针类型值指向的变量的类型。

不同于引用（引用类型的值）时，不会由垃圾回收器跟踪指针-垃圾回收器不知道指针及其指向的数据。 出于此原因，不允许指针指向引用或包含引用的结构，并且指针的引用类型必须是*unmanaged_type*。

*Unmanaged_type*是*reference_type*或构造类型之外的任何类型，并且在任何嵌套级别都不包含*reference_type*或构造类型字段。 换句话说， *unmanaged_type*是以下项之一：

*  `sbyte`、`byte`、`short`、`ushort`、`int`、`uint`、`long`、`ulong`、`char`、`float`、`double`、`decimal`或 `bool`。
*  任何*enum_type*。
*  任何*pointer_type*。
*  任何不是构造类型并且只包含*unmanaged_type*的字段的用户定义的*struct_type* 。

混合使用指针和引用的直观规则是，引用（对象）的引用（对象）允许包含指针，但指针的引用不允许包含引用。

下表给出了指针类型的一些示例：

| __示例__ | __描述__                               |
|-------------|-----------------------------------------------|
| `byte*`     | 指向 `byte` 的指针                             |
| `char*`     | 指向 `char` 的指针                             |
| `int**`     | 指向 `int` 指针的指针                   |
| `int*[]`    | 指向 `int` 的指针的一维数组 |
| `void*`     | 指向未知类型的指针                       |

对于给定的实现，所有指针类型都必须具有相同的大小和表示形式。

与 C 和C++不同，在同一声明中声明多个指针时， C#在 `*` 中仅与基础类型一起编写，而不作为每个指针名称的前缀标点符号。 例如

```csharp
int* pi, pj;    // NOT as int *pi, *pj;
```

具有类型 `T*` 的指针的值表示 `T`类型的变量的地址。 指针间接寻址运算符 `*` （[指针间接寻址](unsafe-code.md#pointer-indirection)）可用于访问此变量。 例如，给定 `int*`类型的变量 `P`，expression `*P` 表示在 `P`中包含的地址处找到的 `int` 变量。

与对象引用类似，指针可能 `null`。 将间接寻址运算符应用于 `null` 指针将导致实现定义的行为。 具有值 `null` 的指针由所有位均为零表示。

`void*` 类型表示指向未知类型的指针。 由于引用类型未知，因此间接寻址运算符不能应用于类型 `void*`的指针，也不能对此类指针执行任何算术运算。 但是，类型 `void*` 的指针可以转换为任何其他指针类型（反之亦然）。

指针类型是一种单独的类型类别。 与引用类型和值类型不同，指针类型不从 `object` 继承，并且指针类型和 `object`之间不存在转换。 特别是，指针不支持装箱和取消装箱（[装箱和取消](types.md#boxing-and-unboxing)装箱）。 但允许在不同指针类型之间以及指针类型和整型之间进行转换。 [指针转换](unsafe-code.md#pointer-conversions)中对此进行了说明。

*Pointer_type*不能用作类型参数（[构造类型](types.md#constructed-types)），并且类型推理（[类型推理](expressions.md#type-inference)）对泛型方法调用失败，而这种方法调用会将类型参数推断为指针类型。

*Pointer_type*可以用作可变字段（[可变字段](classes.md#volatile-fields)）的类型。

尽管指针可以作为 `ref` 或 `out` 参数传递，但这样做可能会导致未定义的行为，因为在调用的方法返回时，指针可能设置为指向某个局部变量，而在调用的方法返回时，或其所指向的固定对象不再是固定的。 例如：

```csharp
using System;

class Test
{
    static int value = 20;

    unsafe static void F(out int* pi1, ref int* pi2) {
        int i = 10;
        pi1 = &i;

        fixed (int* pj = &value) {
            // ...
            pi2 = pj;
        }
    }

    static void Main() {
        int i = 10;
        unsafe {
            int* px1;
            int* px2 = &i;

            F(out px1, ref px2);

            Console.WriteLine("*px1 = {0}, *px2 = {1}",
                *px1, *px2);    // undefined behavior
        }
    }
}
```

方法可以返回某种类型的值，并且该类型可以是指针。 例如，当给定指向连续 `int`s 序列的指针、序列的元素计数以及其他 `int` 值时，如果发生匹配，以下方法将返回该序列中该值的地址;否则，它将返回 `null`：

```csharp
unsafe static int* Find(int* pi, int size, int value) {
    for (int i = 0; i < size; ++i) {
        if (*pi == value) 
            return pi;
        ++pi;
    }
    return null;
}
```

在不安全的上下文中，有多个构造可用于对指针进行操作：

*  `*` 运算符可用于执行指针间接寻址（[指针间接](unsafe-code.md#pointer-indirection)寻址）。
*  `->` 运算符可用于通过指针（[指针成员访问](unsafe-code.md#pointer-member-access)）访问结构的成员。
*  `[]` 运算符可用于对指针（[指针元素访问](unsafe-code.md#pointer-element-access)）进行索引。
*  `&` 运算符可用于获取变量的地址（[运算符的地址](unsafe-code.md#the-address-of-operator)）。
*  `++` 和 `--` 运算符可用来递增和递减指针（[指针递增和递减](unsafe-code.md#pointer-increment-and-decrement)）。
*  `+` 和 `-` 运算符可用于执行指针算法（[指针算法](unsafe-code.md#pointer-arithmetic)）。
*  可以使用 `==`、`!=`、`<`、`>`、`<=`和 `=>` 运算符来比较指针（[指针比较](unsafe-code.md#pointer-comparison)）。
*  `stackalloc` 运算符可用于从调用堆栈（[固定大小缓冲区](unsafe-code.md#fixed-size-buffers)）分配内存。
*  可以使用 `fixed` 语句来暂时修复变量，以便可以获取其地址（[fixed 语句](unsafe-code.md#the-fixed-statement)）。

## <a name="fixed-and-moveable-variables"></a>固定和可移动变量

Address 运算符（[address 运算符](unsafe-code.md#the-address-of-operator)）和 `fixed` 语句（[fixed 语句](unsafe-code.md#the-fixed-statement)）将变量分为两类：***固定变量***和***可移动变量***。

固定变量驻留在不受垃圾回收器操作影响的存储位置中。 （固定变量的示例包括局部变量、值参数和通过取消引用指针而创建的变量。）另一方面，可移动变量驻留在由垃圾回收器进行重定位或处置的存储位置。 （可移动变量的示例包括对象中的字段和数组的元素。）

`&` 运算符（[地址运算符](unsafe-code.md#the-address-of-operator)）允许在不限制的情况下获取固定变量的地址。 但是，由于可移动变量可能会由垃圾回收器重定位或处置，因此只能使用 `fixed` 语句（[fixed 语句](unsafe-code.md#the-fixed-statement)）获取可移动变量的地址，并且该地址在 `fixed` 语句的持续时间内保持有效。

确切地说，固定变量是以下项之一：

*  由引用本地变量或值参数的*simple_name* （[简单名称](expressions.md#simple-names)）产生的变量，除非该变量是由匿名函数捕获的。
*  由 `V.I`格式的*member_access* （[成员访问](expressions.md#member-access)）产生的变量，其中 `V` 是*struct_type*的固定变量。
*  由 `*P`的窗体的*pointer_indirection_expression* （[指针间接寻址](unsafe-code.md#pointer-indirection)）、`P->I`形式的*pointer_member_access* （[指针成员访问](unsafe-code.md#pointer-member-access)）或 pointer_element_access 形式的 *`P[E]`* （[指针元素访问](unsafe-code.md#pointer-element-access)）生成的变量。

所有其他变量归类为可移动变量。

请注意，静态字段归类为可移动变量。 另请注意，`ref` 或 `out` 参数归类为可移动变量，即使为参数提供的参数是固定变量。 最后请注意，通过取消引用指针生成的变量始终归类为固定变量。

## <a name="pointer-conversions"></a>指针转换

在不安全的上下文中，可以使用隐式转换集（[隐式转换](conversions.md#implicit-conversions)）进行扩展，以包括以下隐式指针转换：

*  从任何*pointer_type*到类型的 `void*`。
*  从 `null` 文本到任何*pointer_type*。

此外，在不安全的上下文中，可使用的显式转换集（[显式转换](conversions.md#explicit-conversions)）进行扩展，以包括以下显式指针转换：

*  从任何*pointer_type*到任何其他*pointer_type*。
*  从 `sbyte`、`byte`、`short`、`ushort`、`int`、`uint`、`long`或 `ulong` 到任何*pointer_type*。
*  从任何*pointer_type*到 `sbyte`、`byte`、`short`、`ushort`、`int`、`uint`、`long`或 `ulong`。

最后，在不安全的上下文中，标准隐式转换集（[标准隐式转换](conversions.md#standard-implicit-conversions)）包括以下指针转换：

*  从任何*pointer_type*到类型的 `void*`。

两个指针类型之间的转换决不会更改实际指针的值。 换句话说，从一种指针类型到另一种类型的转换不会影响由指针提供的基础地址。

当一种指针类型转换为另一种类型时，如果结果指针没有正确对齐所引用的类型，则该行为是不确定的。 通常，"正确对齐" 这一概念是可传递的：如果指向类型 `A` 的指针为指向类型 `B`的指针正确对齐，而后者又为指向类型 `C`的指针正确对齐，则指向类型 `A` 的指针将正确对齐到类型 `C`的指针。

请考虑以下情况：通过指向其他类型的指针访问具有一种类型的变量：

```csharp
char c = 'A';
char* pc = &c;
void* pv = pc;
int* pi = (int*)pv;
int i = *pi;         // undefined
*pi = 123456;        // undefined
```

将指针类型转换为指向字节的指针时，结果将指向该变量的最低寻址字节。 结果的后续增量，最大为变量的大小，产生指向该变量的其余字节的指针。 例如，下面的方法以十六进制值的形式显示 double 中的八个字节中的每个字节：

```csharp
using System;

class Test
{
    unsafe static void Main() {
      double d = 123.456e23;
        unsafe {
           byte* pb = (byte*)&d;
            for (int i = 0; i < sizeof(double); ++i)
               Console.Write("{0:X2} ", *pb++);
            Console.WriteLine();
        }
    }
}
```

当然，生成的输出取决于 endian。

指针和整数之间的映射是实现定义的。 但是，在带有线性地址空间的 32 * 和64位 CPU 体系结构上，转换到整数类型或从整型转换到整数类型的方式通常与 `uint` 或 `ulong` 值的转换方式完全相同。

### <a name="pointer-arrays"></a>指针数组

在不安全的上下文中，可以构造指针的数组。 对于指针数组，只允许使用某些适用于其他数组类型的转换：

*  从任何*array_type*到 `System.Array` 以及它所实现的接口的隐[式引用转换](conversions.md#implicit-reference-conversions)也适用于指针数组。 但是，通过 `System.Array` 或它实现的接口访问数组元素的任何尝试都将导致运行时异常，因为指针类型无法转换为 `object`。
*  从一维数组类型 `S[]` 隐式和显式引用转换（[隐式](conversions.md#implicit-reference-conversions)引用转换、[显式引用转换](conversions.md#explicit-reference-conversions)）到 `System.Collections.Generic.IList<T>` 及其泛型基接口从不应用于指针数组，因为指针类型不能用作类型参数，并且不能从指针类型转换为非指针类型。
*  从 `System.Array` 显式引用转换（[显式引用](conversions.md#explicit-reference-conversions)转换）以及它为任何*array_type*实现的接口适用于指针数组。
*  从 `System.Collections.Generic.IList<S>` 将显式引用转换（[显式引用转换](conversions.md#explicit-reference-conversions)）及其基接口转换为一维数组类型 `T[]` 从不适用于指针数组，因为指针类型不能用作类型参数，并且不能从指针类型转换为非指针类型。

这些限制意味着对[foreach 语句](statements.md#the-foreach-statement)中所述数组的 `foreach` 语句的扩展不能应用于指针数组。 相反，窗体的 foreach 语句

```csharp
foreach (V v in x) embedded_statement
```

其中，`x` 的类型为 `T[,,...,]`形式的数组类型，`N` 为维数减1，`T` 或 `V` 是指针类型，使用嵌套的 for 循环进行扩展，如下所示：

```csharp
{
    T[,,...,] a = x;
    for (int i0 = a.GetLowerBound(0); i0 <= a.GetUpperBound(0); i0++)
    for (int i1 = a.GetLowerBound(1); i1 <= a.GetUpperBound(1); i1++)
    ...
    for (int iN = a.GetLowerBound(N); iN <= a.GetUpperBound(N); iN++) {
        V v = (V)a.GetValue(i0,i1,...,iN);
        embedded_statement
    }
}
```

`a`、`i0`、`i1`、...、`iN` 的变量对 `x` 或*程序的任何*其他源代码都不可见或不可访问。 变量 `v` 在嵌入语句中是只读的。 如果没有从 `T` （元素类型）到 `V`的显式转换（[指针转换](unsafe-code.md#pointer-conversions)），则会生成错误，并且不会执行任何其他步骤。 如果 `x` 的值 `null`，则在运行时将引发 `System.NullReferenceException`。

## <a name="pointers-in-expressions"></a>表达式中的指针

在不安全的上下文中，表达式可能产生指针类型的结果，但在不安全的上下文外部，表达式是指针类型的编译时错误。 在不安全的上下文中，如果任何*simple_name* （[简单名称](expressions.md#simple-names)）、 *member_access* （[成员访问](expressions.md#member-access)）、 *invocation_expression* （[调用表达式](expressions.md#invocation-expressions)）或*element_access* （[元素访问](expressions.md#element-access)）是指针类型，则会发生编译时错误。

在不安全的上下文中， *primary_no_array_creation_expression* （[主表达式](expressions.md#primary-expressions)）和*unary_expression* （[一元运算符](expressions.md#unary-operators)）生产允许以下附加构造：

```antlr
primary_no_array_creation_expression_unsafe
    : pointer_member_access
    | pointer_element_access
    | sizeof_expression
    ;

unary_expression_unsafe
    : pointer_indirection_expression
    | addressof_expression
    ;
```

以下各节介绍了这些构造。 语法暗示了 unsafe 运算符的优先级和关联性。

### <a name="pointer-indirection"></a>指针间接寻址

*Pointer_indirection_expression*包含一个星号（`*`），后跟一个*unary_expression*。

```antlr
pointer_indirection_expression
    : '*' unary_expression
    ;
```

一元 `*` 运算符表示指针间接寻址，用于获取指针所指向的变量。 计算 `*P`的结果（其中 `P` 是指针类型 `T*`的表达式）是 `T`类型的变量。 将一元 `*` 运算符应用到 `void*` 类型的表达式或不是指针类型的表达式时，会发生编译时错误。

向 `null` 指针应用一元 `*` 运算符的效果是实现定义的。 特别是，不能保证此操作引发 `System.NullReferenceException`。

如果向指针分配了无效的值，则一元 `*` 运算符的行为是不确定的。 用于通过一元 `*` 运算符取消对指针的引用的无效值是为指向的类型（请参阅[指针转换](unsafe-code.md#pointer-conversions)中的示例）的不一致的地址，以及变量生存期结束后的变量地址。

出于明确的赋值分析目的，通过计算形式 `*P` 的表达式所生成的变量被视为初始赋值（[最初分配的变量](variables.md#initially-assigned-variables)）。

### <a name="pointer-member-access"></a>指针成员访问

*Pointer_member_access*包含一个*primary_expression*，后跟一个 "`->`" 标记，后跟一个*标识符*和一个可选的*type_argument_list*。

```antlr
pointer_member_access
    : primary_expression '->' identifier
    ;
```

在形式 `P->I`的指针成员访问中，`P` 必须是除 `void*`之外的指针类型的表达式，并且 `I` 必须表示 `P` 点的类型的可访问成员。

形式 `P->I` 的指针成员访问权限完全按 `(*P).I`进行计算。 有关指针间接寻址运算符（`*`）的说明，请参阅[指针间接寻址](unsafe-code.md#pointer-indirection)。 有关成员访问运算符（`.`）的说明，请参阅[成员访问](expressions.md#member-access)。

示例中

```csharp
using System;

struct Point
{
    public int x;
    public int y;

    public override string ToString() {
        return "(" + x + "," + y + ")";
    }
}

class Test
{
    static void Main() {
        Point point;
        unsafe {
            Point* p = &point;
            p->x = 10;
            p->y = 20;
            Console.WriteLine(p->ToString());
        }
    }
}
```

`->` 运算符用于访问字段并通过指针调用结构的方法。 由于 `P->I` 的操作完全等同于 `(*P).I`，因此，`Main` 方法的编写方式同样好：

```csharp
class Test
{
    static void Main() {
        Point point;
        unsafe {
            Point* p = &point;
            (*p).x = 10;
            (*p).y = 20;
            Console.WriteLine((*p).ToString());
        }
    }
}
```

### <a name="pointer-element-access"></a>指针元素访问

*Pointer_element_access*包含一个*primary_no_array_creation_expression*后跟一个括在 "`[`" 和 "`]`" 的表达式。

```antlr
pointer_element_access
    : primary_no_array_creation_expression '[' expression ']'
    ;
```

在 `P[E]`形式的指针元素访问中，`P` 必须是 `void*`的指针类型的表达式，而 `E` 必须是可以隐式转换为 `int`、`uint`、`long`或 `ulong`的表达式。

`P[E]` 的窗体的指针元素访问都精确地按 `*(P + E)`进行计算。 有关指针间接寻址运算符（`*`）的说明，请参阅[指针间接寻址](unsafe-code.md#pointer-indirection)。 有关指针加法运算符（`+`）的说明，请参阅[指针算法](unsafe-code.md#pointer-arithmetic)。

示例中

```csharp
class Test
{
    static void Main() {
        unsafe {
            char* p = stackalloc char[256];
            for (int i = 0; i < 256; i++) p[i] = (char)i;
        }
    }
}
```

指针元素访问用于在 `for` 循环中初始化字符缓冲区。 由于 `P[E]` 的操作完全等同于 `*(P + E)`，因此，该示例可能同样已写入：

```csharp
class Test
{
    static void Main() {
        unsafe {
            char* p = stackalloc char[256];
            for (int i = 0; i < 256; i++) *(p + i) = (char)i;
        }
    }
}
```

指针元素访问运算符不会检查超出界限的错误，并且在访问超出界限的元素时的行为不确定。 这与 C 和C++相同。

### <a name="the-address-of-operator"></a>Address 运算符

*Addressof_expression*由 "and" 符（`&`） *unary_expression*开头。

```antlr
addressof_expression
    : '&' unary_expression
    ;
```

给定 `T` 类型为的表达式 `E`，并将其归类为固定变量（固定变量[和可移动变量](unsafe-code.md#fixed-and-moveable-variables)），构造 `&E` 计算 `E`指定的变量的地址。 结果的类型为 `T*`，并归类为值。 如果 `E` 未分类为变量，则会发生编译时错误，如果 `E` 归类为只读局部变量，则会发生编译时错误; 如果 `E` 表示可移动的变量，则会发生编译时错误。 在最后一种情况下，可以使用 fixed 语句（[fixed 语句](unsafe-code.md#the-fixed-statement)）来暂时 "修复" 该变量，然后获取其地址。 如[成员访问](expressions.md#member-access)中所述，在定义 `readonly` 字段的结构或类的实例构造函数或静态构造函数之外，该字段被视为值而非变量。 因此无法采用其地址。 同样，不能采用常量的地址。

`&` 运算符不需要将其自变量指定为明确赋值，但在执行 `&` 操作后，该运算符应用于的变量在执行操作的执行路径中被视为明确赋值。 编程人员负责确保在这种情况下正确地初始化变量。

示例中

```csharp
using System;

class Test
{
    static void Main() {
        int i;
        unsafe {
            int* p = &i;
            *p = 123;
        }
        Console.WriteLine(i);
    }
}
```

`i` 在用于初始化 `p`的 `&i` 操作后被视为已明确赋值。 对 `*p` 的赋值实际上会初始化 `i`，但包含此初始化是程序员的责任，并且如果删除分配，则不会发生编译时错误。

为 `&` 运算符提供明确赋值的规则，以便可以避免本地变量的冗余初始化。 例如，许多外部 Api 采用一个指向结构的指针，该结构由 API 填充。 调用此类 Api 通常会传递本地结构变量的地址，并且如果没有规则，则需要对结构变量进行冗余初始化。

### <a name="pointer-increment-and-decrement"></a>指针增量和减量

在不安全的上下文中，`++` 和 `--` 运算符（[后缀递增和递减运算符](expressions.md#postfix-increment-and-decrement-operators)以及[前缀增量和减量运算符](expressions.md#prefix-increment-and-decrement-operators)）可应用于除 `void*`之外的所有类型的指针变量。 因此，对于每个指针类型 `T*`，将隐式定义以下运算符：

```csharp
T* operator ++(T* x);
T* operator --(T* x);
```

运算符分别与 `x + 1` 和 `x - 1`生成相同的结果（[指针算法](unsafe-code.md#pointer-arithmetic)）。 换言之，对于类型 `T*`的指针变量，`++` 运算符将 `sizeof(T)` 添加到变量中包含的地址，并且 `--` 运算符将从该变量中包含的地址减去 `sizeof(T)`。

如果指针增量或减量运算溢出了指针类型的域，则结果是实现定义的，但不会产生异常。

### <a name="pointer-arithmetic"></a>指针算术

在不安全的上下文中，`+` 和 `-` 运算符（[加法运算符](expressions.md#addition-operator)和[减法运算符](expressions.md#subtraction-operator)）可应用于除 `void*`之外的所有指针类型的值。 因此，对于每个指针类型 `T*`，将隐式定义以下运算符：

```csharp
T* operator +(T* x, int y);
T* operator +(T* x, uint y);
T* operator +(T* x, long y);
T* operator +(T* x, ulong y);

T* operator +(int x, T* y);
T* operator +(uint x, T* y);
T* operator +(long x, T* y);
T* operator +(ulong x, T* y);

T* operator -(T* x, int y);
T* operator -(T* x, uint y);
T* operator -(T* x, long y);
T* operator -(T* x, ulong y);

long operator -(T* x, T* y);
```

给定表达式 `P` 指针类型 `T*` 和类型 `int`、`uint`、`long`或 `ulong`的表达式 `N`，表达式 `P + N` 和 `N + P` 会计算类型 `T*` 的指针值，结果是将 `N * sizeof(T)` 添加到 `P`给定的地址。 同样，表达式 `P - N` 会计算类型 `T*` 的指针值，因为从 `P`给定的地址中减去 `N * sizeof(T)` 产生的结果。

给定指针类型 `T*`的两个表达式（`P` 和 `Q`），表达式 `P - Q` 计算 `P` 和 `Q` 给定的地址之间的差异，然后将该差异除以 `sizeof(T)`。 结果的类型始终 `long`。 实际上，`P - Q` 是 `((long)(P) - (long)(Q)) / sizeof(T)`计算得出的。

例如：

```csharp
using System;

class Test
{
    static void Main() {
        unsafe {
            int* values = stackalloc int[20];
            int* p = &values[1];
            int* q = &values[15];
            Console.WriteLine("p - q = {0}", p - q);
            Console.WriteLine("q - p = {0}", q - p);
        }
    }
}
```

生成输出的：

```console
p - q = -14
q - p = 14
```

如果指针算术运算溢出了指针类型的域，则会以实现定义的方式截断结果，但不会产生异常。

### <a name="pointer-comparison"></a>指针比较

在不安全的上下文中，`==`、`!=`、`<`、`>`、`<=`和 `=>` 运算符（[关系和类型测试运算符](expressions.md#relational-and-type-testing-operators)）可应用于所有指针类型的值。 指针比较运算符是：

```csharp
bool operator ==(void* x, void* y);
bool operator !=(void* x, void* y);
bool operator <(void* x, void* y);
bool operator >(void* x, void* y);
bool operator <=(void* x, void* y);
bool operator >=(void* x, void* y);
```

由于存在从任何指针类型到 `void*` 类型的隐式转换，因此可以使用这些运算符比较任何指针类型的操作数。 比较运算符比较两个操作数给定的地址，就像它们是无符号整数。

### <a name="the-sizeof-operator"></a>Sizeof 运算符

`sizeof` 运算符返回给定类型的变量所占用的字节数。 指定为 `sizeof` 的操作数的类型必须是*unmanaged_type* （[指针类型](unsafe-code.md#pointer-types)）。

```antlr
sizeof_expression
    : 'sizeof' '(' unmanaged_type ')'
    ;
```

`sizeof` 运算符的结果是 `int`类型的值。 对于某些预定义类型，`sizeof` 运算符产生一个常数值，如下表所示。


| __表达式__   | __结果__ |
|------------------|------------|
| `sizeof(sbyte)`  | `1`        |
| `sizeof(byte)`   | `1`        |
| `sizeof(short)`  | `2`        |
| `sizeof(ushort)` | `2`        |
| `sizeof(int)`    | `4`        |
| `sizeof(uint)`   | `4`        |
| `sizeof(long)`   | `8`        |
| `sizeof(ulong)`  | `8`        |
| `sizeof(char)`   | `2`        |
| `sizeof(float)`  | `4`        |
| `sizeof(double)` | `8`        |
| `sizeof(bool)`   | `1`        |

对于所有其他类型，`sizeof` 运算符的结果是实现定义的，并归类为值而不是常量。

成员打包到结构中的顺序是未指定的。

出于对齐目的，在结构的开头、结构和结构的末尾，可能有未命名的填充。 用作填充的位数不确定。

当应用于具有结构类型的操作数时，结果为该类型的变量中的总字节数，包括任何空白。

## <a name="the-fixed-statement"></a>Fixed 语句

在不安全的上下文中， *embedded_statement* （[语句](statements.md)）生产允许使用其他构造，即 `fixed` 语句，该语句用于 "修复" 可移动变量，使其地址在语句的持续时间内保持不变。

```antlr
fixed_statement
    : 'fixed' '(' pointer_type fixed_pointer_declarators ')' embedded_statement
    ;

fixed_pointer_declarators
    : fixed_pointer_declarator (','  fixed_pointer_declarator)*
    ;

fixed_pointer_declarator
    : identifier '=' fixed_pointer_initializer
    ;

fixed_pointer_initializer
    : '&' variable_reference
    | expression
    ;
```

每个*fixed_pointer_declarator*都声明给定*pointer_type*的局部变量，并使用相应*fixed_pointer_initializer*计算出的地址初始化该局部变量。 在 `fixed` 语句中声明的局部变量可在该变量的声明右侧和 `fixed` 语句的*embedded_statement*中的任何*fixed_pointer_initializer*进行访问。 `fixed` 语句声明的局部变量被视为只读。 如果嵌入的语句尝试修改此局部变量（通过赋值或 `++` 和 `--` 运算符），或将其作为 `ref` 或 `out` 参数传递，则会发生编译时错误。

*Fixed_pointer_initializer*可以是以下项之一：

*  标记 "`&`" 后跟*variable_reference* （[确定明确赋值的精确规则](variables.md#precise-rules-for-determining-definite-assignment)）到非托管类型 `T`的可移动变量（[固定和可移动变量](unsafe-code.md#fixed-and-moveable-variables)），前提是该类型 `T*` 可隐式转换为 `fixed` 语句中给定的指针类型。 在这种情况下，初始值设定项将计算给定变量的地址，并且在 `fixed` 语句的持续时间内，该变量可保证保持为固定地址。
*  带有非托管类型 `T`元素的*array_type*的表达式，前提是该类型 `T*` 可隐式转换为 `fixed` 语句中给定的指针类型。 在这种情况下，初始值设定项计算数组中第一个元素的地址，并且在 `fixed` 语句的持续时间内保证整个数组保持为固定的地址。 如果数组表达式为 null 或数组包含零个元素，则初始值设定项将计算等于零的地址。
*  如果类型 `char*` 可隐式转换为 `fixed` 语句中给定的指针类型，则为 `string`类型的表达式。 在这种情况下，初始值设定项将计算字符串中第一个字符的地址，并且在 `fixed` 语句的持续时间内保证整个字符串保持为固定地址。 如果字符串表达式为 null，则 `fixed` 语句的行为是实现定义的。
*  如果固定大小缓冲区成员的类型可隐式转换为在 `fixed` 语句中给定的指针类型，则引用可移动变量的固定大小缓冲区成员的*simple_name*或*member_access* 。 在这种情况下，初始值设定项将计算一个指针，该指针指向固定大小缓冲区的第一个元素（[表达式中的固定大小缓冲区](unsafe-code.md#fixed-size-buffers-in-expressions)），并且在 `fixed` 语句的持续时间内保证固定大小缓冲区保持为固定地址。

对于由*fixed_pointer_initializer*计算得出的每个地址，`fixed` 语句可确保在 `fixed` 语句的持续时间内，垃圾回收器不会将该地址引用的变量重定位或释放。 例如，如果由*fixed_pointer_initializer*计算的地址引用一个对象的字段或数组实例的一个元素，则 `fixed` 语句保证在该语句的生存期内不会重新定位或释放包含对象实例。

程序员应负责确保 `fixed` 语句创建的指针不会在执行这些语句的范围之外。 例如，当 `fixed` 语句创建的指针传递到外部 Api 时，程序员应负责确保 Api 不保留这些指针的内存。

固定对象可能会导致堆碎片（因为它们无法移动）。 因此，仅当绝对必要时才应修复对象，而只应在尽可能最短的时间进行修复。

示例

```csharp
class Test
{
    static int x;
    int y;

    unsafe static void F(int* p) {
        *p = 1;
    }

    static void Main() {
        Test t = new Test();
        int[] a = new int[10];
        unsafe {
            fixed (int* p = &x) F(p);
            fixed (int* p = &t.y) F(p);
            fixed (int* p = &a[0]) F(p);
            fixed (int* p = a) F(p);
        }
    }
}
```

演示 `fixed` 语句的几次使用。 第一条语句修复并获取静态字段的地址，第二条语句修复并获取实例字段的地址，第三条语句修复并获取数组元素的地址。 在每种情况下，使用 regular `&` 运算符都是错误的，因为变量全都归类为可移动变量。

上面示例中的第四个 `fixed` 语句将产生类似于第三个的结果。

此 `fixed` 语句的示例使用 `string`：

```csharp
class Test
{
    static string name = "xx";

    unsafe static void F(char* p) {
        for (int i = 0; p[i] != '\0'; ++i)
            Console.WriteLine(p[i]);
    }

    static void Main() {
        unsafe {
            fixed (char* p = name) F(p);
            fixed (char* p = "xx") F(p);
        }
    }
}
```

在不安全的上下文数组元素中，一维数组的元素按索引顺序存储，从索引 `0` 开始，以 index `Length - 1`结束。 对于多维数组，将存储数组元素，这样最右边的维度的索引将首先增加，然后是下一个左侧维度，依此类推。 在 `fixed` 语句中获取指向数组实例 `a`的指针 `p`，指针值范围从 `p` 到 `p + a.Length - 1` 表示数组中元素的地址。 同样，从 `p[0]` 到 `p[a.Length - 1]` 的变量表示实际数组元素。 考虑到数组的存储方式，可以将任何维度的数组视为线性数组。

例如：

```csharp
using System;

class Test
{
    static void Main() {
        int[,,] a = new int[2,3,4];
        unsafe {
            fixed (int* p = a) {
                for (int i = 0; i < a.Length; ++i)    // treat as linear
                    p[i] = i;
            }
        }

        for (int i = 0; i < 2; ++i)
            for (int j = 0; j < 3; ++j) {
                for (int k = 0; k < 4; ++k)
                    Console.Write("[{0},{1},{2}] = {3,2} ", i, j, k, a[i,j,k]);
                Console.WriteLine();
            }
    }
}
```

生成输出的：

```console
[0,0,0] =  0 [0,0,1] =  1 [0,0,2] =  2 [0,0,3] =  3
[0,1,0] =  4 [0,1,1] =  5 [0,1,2] =  6 [0,1,3] =  7
[0,2,0] =  8 [0,2,1] =  9 [0,2,2] = 10 [0,2,3] = 11
[1,0,0] = 12 [1,0,1] = 13 [1,0,2] = 14 [1,0,3] = 15
[1,1,0] = 16 [1,1,1] = 17 [1,1,2] = 18 [1,1,3] = 19
[1,2,0] = 20 [1,2,1] = 21 [1,2,2] = 22 [1,2,3] = 23
```

示例中

```csharp
class Test
{
    unsafe static void Fill(int* p, int count, int value) {
        for (; count != 0; count--) *p++ = value;
    }

    static void Main() {
        int[] a = new int[100];
        unsafe {
            fixed (int* p = a) Fill(p, 100, -1);
        }
    }
}
```

使用 `fixed` 语句修复数组，以便可以将其地址传递给采用指针的方法。

在下面的示例中：

```csharp
unsafe struct Font
{
    public int size;
    public fixed char name[32];
}

class Test
{
    unsafe static void PutString(string s, char* buffer, int bufSize) {
        int len = s.Length;
        if (len > bufSize) len = bufSize;
        for (int i = 0; i < len; i++) buffer[i] = s[i];
        for (int i = len; i < bufSize; i++) buffer[i] = (char)0;
    }

    Font f;

    unsafe static void Main()
    {
        Test test = new Test();
        test.f.size = 10;
        fixed (char* p = test.f.name) {
            PutString("Times New Roman", p, 32);
        }
    }
}
```

fixed 语句用于修复结构的固定大小缓冲区，使其地址可用作指针。

通过修复字符串实例产生 `char*` 值始终指向以 null 结尾的字符串。 在 `s``p` 获取指向字符串实例的指针的固定语句中，指针值从 `p` 到 `p + s.Length - 1` 表示字符串中的字符的地址，指针值 `p + s.Length` 始终指向 null 字符（值为 `'\0'`的字符）。

通过固定指针修改托管类型的对象可能导致未定义的行为。 例如，因为字符串是不可变的，所以，程序员应负责确保不会修改指向固定字符串的指针所引用的字符。

在调用需要 "C 样式" 字符串的外部 Api 时，字符串的自动 null 终止非常方便。 但请注意，字符串实例允许包含 null 字符。 如果存在此类 null 字符，则该字符串将在被视为以 null 结尾的 `char*`时出现截断。

## <a name="fixed-size-buffers"></a>固定大小的缓冲区

固定大小的缓冲区用于将 "C 样式" 内联数组声明为结构的成员，主要用于与非托管 Api 建立交互。

### <a name="fixed-size-buffer-declarations"></a>固定大小的缓冲区声明

***固定大小缓冲区***是表示给定类型的变量的固定长度缓冲区存储的成员。 固定大小的缓冲区声明引入给定元素类型的一个或多个固定大小的缓冲区。 固定大小的缓冲区只允许在结构声明中使用，并且只能在不安全的上下文中出现（[不安全](unsafe-code.md#unsafe-contexts)上下文）。

```antlr
struct_member_declaration_unsafe
    : fixed_size_buffer_declaration
    ;

fixed_size_buffer_declaration
    : attributes? fixed_size_buffer_modifier* 'fixed' buffer_element_type fixed_size_buffer_declarator+ ';'
    ;

fixed_size_buffer_modifier
    : 'new'
    | 'public'
    | 'protected'
    | 'internal'
    | 'private'
    | 'unsafe'
    ;

buffer_element_type
    : type
    ;

fixed_size_buffer_declarator
    : identifier '[' constant_expression ']'
    ;
```

固定大小的缓冲区声明可以包含一组属性（[特性](attributes.md)）、一个 `new` 修饰符（[修饰符](classes.md#modifiers)）、四个访问修饰符（[类型参数和约束](classes.md#type-parameters-and-constraints)）和 `unsafe` 修饰符（[不安全上下文](unsafe-code.md#unsafe-contexts)）的有效组合。 特性和修饰符适用于由固定大小缓冲区声明声明的所有成员。 同一修饰符在固定大小的缓冲区声明中多次出现是错误的。

固定大小的缓冲区声明不允许包含 `static` 修饰符。

固定大小缓冲区声明的 buffer 元素类型指定声明引入的缓冲区的元素类型。 Buffer 元素类型必须是预定义的类型之一 `sbyte`、`byte`、`short`、`ushort`、`int`、`uint`、`long`、`ulong`、`char`、`float`、`double`或 `bool`。

Buffer 元素类型后跟固定大小缓冲区声明符的列表，其中每个声明符都引入一个新成员。 固定大小的缓冲区声明符包含一个命名成员的标识符，后跟一个括在 `[` 和 `]` 令牌中的常量表达式。 常数表达式表示该固定大小缓冲区声明符引入的成员中的元素数目。 常数表达式的类型必须可隐式转换为类型 `int`，且值必须为非零正整数。

保证固定大小缓冲区的元素在内存中按顺序排列。

声明多个固定大小缓冲区的固定大小缓冲区声明等效于具有相同属性和元素类型的单个固定大小缓冲区声明的多个声明。 例如

```csharp
unsafe struct A
{
   public fixed int x[5], y[10], z[100];
}
```

等效于

```csharp
unsafe struct A
{
   public fixed int x[5];
   public fixed int y[10];
   public fixed int z[100];
}
```

### <a name="fixed-size-buffers-in-expressions"></a>表达式中固定大小的缓冲区

固定大小缓冲区成员的成员查找（[运算符](expressions.md#operators)）与字段的成员查找完全相同。

使用*simple_name* （[类型推理](expressions.md#type-inference)）或*member_access* （对[动态重载决策进行编译时检查](expressions.md#compile-time-checking-of-dynamic-overload-resolution)），可以在表达式中引用固定大小的缓冲区。

当将固定大小的缓冲区成员作为简单名称引用时，其效果与窗体 `this.I`的成员访问相同，其中 `I` 为固定大小缓冲区成员。

在窗体 `E.I`的成员访问中，如果 `E` 为结构类型，并且该结构类型中的 `I` 的成员查找标识固定大小成员，则按如下方式计算 `E.I`：

*  如果表达式 `E.I` 未出现在不安全的上下文中，则会发生编译时错误。
*  如果 `E` 归类为值，则会发生编译时错误。
*  否则，如果 `E` 是可移动变量（[固定和可移动变量](unsafe-code.md#fixed-and-moveable-variables)），并且表达式 `E.I` 不是*fixed_pointer_initializer* （[Fixed 语句](unsafe-code.md#the-fixed-statement)），则会发生编译时错误。
*  否则，`E` 引用了一个固定变量，并且该表达式的结果是一个指向 `E`中固定大小缓冲区成员 `I` 的第一个元素的指针。 结果的类型为 `S*`，其中 `S` 是 `I`的元素类型，并归类为值。

可以使用第一个元素的指针操作访问固定大小缓冲区的后续元素。 与对数组的访问不同，对固定大小缓冲区的元素的访问是不安全的操作，并且不会进行范围检查。

下面的示例声明并使用具有固定大小缓冲区成员的结构。

```csharp
unsafe struct Font
{
    public int size;
    public fixed char name[32];
}

class Test
{
    unsafe static void PutString(string s, char* buffer, int bufSize) {
        int len = s.Length;
        if (len > bufSize) len = bufSize;
        for (int i = 0; i < len; i++) buffer[i] = s[i];
        for (int i = len; i < bufSize; i++) buffer[i] = (char)0;
    }

    unsafe static void Main()
    {
        Font f;
        f.size = 10;
        PutString("Times New Roman", f.name, 32);
    }
}
```

### <a name="definite-assignment-checking"></a>明确赋值检查

固定大小的缓冲区不受明确的赋值检查（[明确赋值](variables.md#definite-assignment)）的限制，并且忽略固定大小的缓冲区成员，以便对结构类型变量进行明确的赋值检查。

当固定大小缓冲区成员的最外面的包含结构变量是静态变量、类实例的实例变量或数组元素时，固定大小缓冲区的元素会自动初始化为其默认值（[默认值](variables.md#default-values)）。 在所有其他情况下，固定大小缓冲区的初始内容是不确定的。

## <a name="stack-allocation"></a>堆栈分配

在不安全的上下文中，局部变量声明（[局部变量声明](statements.md#local-variable-declarations)）可能包括从调用堆栈中分配内存的堆栈分配初始值设定项。

```antlr
local_variable_initializer_unsafe
    : stackalloc_initializer
    ;

stackalloc_initializer
    : 'stackalloc' unmanaged_type '[' expression ']'
    ;
```

*Unmanaged_type*指示将存储在新分配的位置的项的类型，并且*表达式*指示这些项的数目。 它们一起指定了所需的分配大小。 由于堆栈分配的大小不能为负，因此将项的数目指定为计算结果为负值的*constant_expression*是编译时错误。

格式 `stackalloc T[E]` 的堆栈分配初始值设定项需要 `T` 为非托管类型（[指针类型](unsafe-code.md#pointer-types)），并且 `E` 是 `int`类型的表达式。 构造从调用堆栈分配 `E * sizeof(T)` 字节，并将 `T*`类型的指针返回到新分配的块。 如果 `E` 为负值，则行为是不确定的。 如果 `E` 为零，则不进行任何分配，并且返回的指针是实现定义的。 如果内存不足，无法分配指定大小的块，则会引发 `System.StackOverflowException`。

新分配的内存的内容未定义。

`catch` 或 `finally` 块（[try 语句](statements.md#the-try-statement)）中不允许使用堆栈分配初始值设定项。

无法显式释放使用 `stackalloc`分配的内存。 在函数成员执行过程中创建的所有堆栈分配的内存块将在该函数成员返回时自动被丢弃。 这对应于 `alloca` 函数，这是一个在 C 和C++实现中常见的扩展。

示例中

```csharp
using System;

class Test
{
    static string IntToString(int value) {
        int n = value >= 0? value: -value;
        unsafe {
            char* buffer = stackalloc char[16];
            char* p = buffer + 16;
            do {
                *--p = (char)(n % 10 + '0');
                n /= 10;
            } while (n != 0);
            if (value < 0) *--p = '-';
            return new string(p, 0, (int)(buffer + 16 - p));
        }
    }

    static void Main() {
        Console.WriteLine(IntToString(12345));
        Console.WriteLine(IntToString(-999));
    }
}
```

`stackalloc` 初始值设定项用在 `IntToString` 方法中，用于在堆栈上分配16个字符的缓冲区。 当该方法返回时，缓冲区会自动丢弃。

## <a name="dynamic-memory-allocation"></a>动态内存分配

除 `stackalloc` 运算符外， C#不提供用于管理非垃圾回收内存的预定义构造。 此类服务通常通过支持类库来提供，也可以从基础操作系统直接导入。 例如，下面的 `Memory` 类演示了如何从C#访问基础操作系统的堆函数：

```csharp
using System;
using System.Runtime.InteropServices;

public static unsafe class Memory
{
    // Handle for the process heap. This handle is used in all calls to the
    // HeapXXX APIs in the methods below.
    private static readonly IntPtr s_heap = GetProcessHeap();

    // Allocates a memory block of the given size. The allocated memory is
    // automatically initialized to zero.
    public static void* Alloc(int size)
    {
        void* result = HeapAlloc(s_heap, HEAP_ZERO_MEMORY, (UIntPtr)size);
        if (result == null) throw new OutOfMemoryException();
        return result;
    }

    // Copies count bytes from src to dst. The source and destination
    // blocks are permitted to overlap.
    public static void Copy(void* src, void* dst, int count)
    {
        byte* ps = (byte*)src;
        byte* pd = (byte*)dst;
        if (ps > pd)
        {
            for (; count != 0; count--) *pd++ = *ps++;
        }
        else if (ps < pd)
        {
            for (ps += count, pd += count; count != 0; count--) *--pd = *--ps;
        }
    }

    // Frees a memory block.
    public static void Free(void* block)
    {
        if (!HeapFree(s_heap, 0, block)) throw new InvalidOperationException();
    }

    // Re-allocates a memory block. If the reallocation request is for a
    // larger size, the additional region of memory is automatically
    // initialized to zero.
    public static void* ReAlloc(void* block, int size)
    {
        void* result = HeapReAlloc(s_heap, HEAP_ZERO_MEMORY, block, (UIntPtr)size);
        if (result == null) throw new OutOfMemoryException();
        return result;
    }

    // Returns the size of a memory block.
    public static int SizeOf(void* block)
    {
        int result = (int)HeapSize(s_heap, 0, block);
        if (result == -1) throw new InvalidOperationException();
        return result;
    }

    // Heap API flags
    private const int HEAP_ZERO_MEMORY = 0x00000008;

    // Heap API functions
    [DllImport("kernel32")]
    private static extern IntPtr GetProcessHeap();

    [DllImport("kernel32")]
    private static extern void* HeapAlloc(IntPtr hHeap, int flags, UIntPtr size);

    [DllImport("kernel32")]
    private static extern bool HeapFree(IntPtr hHeap, int flags, void* block);

    [DllImport("kernel32")]
    private static extern void* HeapReAlloc(IntPtr hHeap, int flags, void* block, UIntPtr size);

    [DllImport("kernel32")]
    private static extern UIntPtr HeapSize(IntPtr hHeap, int flags, void* block);
}
```

下面给出了使用 `Memory` 类的示例：

```csharp
class Test
{
    static unsafe void Main()
    {
        byte* buffer = null;
        try
        {
            const int Size = 256;
            buffer = (byte*)Memory.Alloc(Size);
            for (int i = 0; i < Size; i++) buffer[i] = (byte)i;
            byte[] array = new byte[Size];
            fixed (byte* p = array) Memory.Copy(buffer, p, Size);
            for (int i = 0; i < Size; i++) Console.WriteLine(array[i]);
        }
        finally
        {
            if (buffer != null) Memory.Free(buffer);
        }
    }
}
```

该示例通过 `Memory.Alloc` 分配256字节的内存，并使用从0到255的值增加内存块。 然后，它分配一个256元素字节数组，并使用 `Memory.Copy` 将内存块的内容复制到字节数组中。 最后，使用 `Memory.Free` 释放内存块，并在控制台上输出字节数组的内容。
