---
ms.openlocfilehash: dbea611280a644adc25247b9887986e129c59b68
ms.sourcegitcommit: a5e393b018b04dfa55aae0000290ca087b508495
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/14/2019
ms.locfileid: "72310360"
---
# <a name="unsafe-code"></a><span data-ttu-id="61be0-101">不安全代码</span><span class="sxs-lookup"><span data-stu-id="61be0-101">Unsafe code</span></span>

<span data-ttu-id="61be0-102">上述章节C#中定义的核心语言与 C 和C++将指针省略为数据类型的方式不同。</span><span class="sxs-lookup"><span data-stu-id="61be0-102">The core C# language, as defined in the preceding chapters, differs notably from C and C++ in its omission of pointers as a data type.</span></span> <span data-ttu-id="61be0-103">相反， C#提供引用和创建由垃圾回收器管理的对象的功能。</span><span class="sxs-lookup"><span data-stu-id="61be0-103">Instead, C# provides references and the ability to create objects that are managed by a garbage collector.</span></span> <span data-ttu-id="61be0-104">此设计与其他功能C#结合使用比 C 或C++更安全。</span><span class="sxs-lookup"><span data-stu-id="61be0-104">This design, coupled with other features, makes C# a much safer language than C or C++.</span></span> <span data-ttu-id="61be0-105">在核心C#语言中，不能使用未初始化的变量、"无关联的" 指针或索引超出其界限的数组的表达式。</span><span class="sxs-lookup"><span data-stu-id="61be0-105">In the core C# language it is simply not possible to have an uninitialized variable, a "dangling" pointer, or an expression that indexes an array beyond its bounds.</span></span> <span data-ttu-id="61be0-106">这样就消除了日常灾难 C 和C++程序的各种 bug。</span><span class="sxs-lookup"><span data-stu-id="61be0-106">Whole categories of bugs that routinely plague C and C++ programs are thus eliminated.</span></span>

<span data-ttu-id="61be0-107">尽管实际上每个指针类型都是在C++ C 中构造的，或是C#在中具有对应的引用类型，但在某些情况下，访问指针类型是必需的。</span><span class="sxs-lookup"><span data-stu-id="61be0-107">While practically every pointer type construct in C or C++ has a reference type counterpart in C#, nonetheless, there are situations where access to pointer types becomes a necessity.</span></span> <span data-ttu-id="61be0-108">例如，在不访问指针的情况下，与基础操作系统、访问内存映射设备或实现时间关键算法之间的交互可能是不可能的或不切实际的。</span><span class="sxs-lookup"><span data-stu-id="61be0-108">For example, interfacing with the underlying operating system, accessing a memory-mapped device, or implementing a time-critical algorithm may not be possible or practical without access to pointers.</span></span> <span data-ttu-id="61be0-109">为了满足这一需求C# ，提供了编写***不安全代码***的功能。</span><span class="sxs-lookup"><span data-stu-id="61be0-109">To address this need, C# provides the ability to write ***unsafe code***.</span></span>

<span data-ttu-id="61be0-110">在不安全代码中，可以声明和操作指针、在指针和整型之间执行转换，以获取变量的地址等。</span><span class="sxs-lookup"><span data-stu-id="61be0-110">In unsafe code it is possible to declare and operate on pointers, to perform conversions between pointers and integral types, to take the address of variables, and so forth.</span></span> <span data-ttu-id="61be0-111">在某种意义上，编写不安全代码非常类似于在C#程序内编写 C 代码。</span><span class="sxs-lookup"><span data-stu-id="61be0-111">In a sense, writing unsafe code is much like writing C code within a C# program.</span></span>

<span data-ttu-id="61be0-112">从开发人员和用户的角度来看，不安全代码实际上是一项 "安全" 功能。</span><span class="sxs-lookup"><span data-stu-id="61be0-112">Unsafe code is in fact a "safe" feature from the perspective of both developers and users.</span></span> <span data-ttu-id="61be0-113">必须使用修饰符 `unsafe`清楚地标记不安全代码，因此开发人员不可能意外地使用不安全的功能，并且执行引擎可以确保不受信任的环境中不能执行不安全的代码。</span><span class="sxs-lookup"><span data-stu-id="61be0-113">Unsafe code must be clearly marked with the modifier `unsafe`, so developers can't possibly use unsafe features accidentally, and the execution engine works to ensure that unsafe code cannot be executed in an untrusted environment.</span></span>

## <a name="unsafe-contexts"></a><span data-ttu-id="61be0-114">不安全上下文</span><span class="sxs-lookup"><span data-stu-id="61be0-114">Unsafe contexts</span></span>

<span data-ttu-id="61be0-115">的不安全功能C#仅在不安全的上下文中可用。</span><span class="sxs-lookup"><span data-stu-id="61be0-115">The unsafe features of C# are available only in unsafe contexts.</span></span> <span data-ttu-id="61be0-116">通过在类型或成员的声明中包含 `unsafe` 修饰符或使用*unsafe_statement*，引入了不安全的上下文：</span><span class="sxs-lookup"><span data-stu-id="61be0-116">An unsafe context is introduced by including an `unsafe` modifier in the declaration of a type or member, or by employing an *unsafe_statement*:</span></span>

*  <span data-ttu-id="61be0-117">类、结构、接口或委托的声明可能包含 `unsafe` 修饰符，在这种情况下，该类型声明的整个文本范围（包括类、结构或接口的正文）被视为不安全的上下文。</span><span class="sxs-lookup"><span data-stu-id="61be0-117">A declaration of a class, struct, interface, or delegate may include an `unsafe` modifier, in which case the entire textual extent of that type declaration (including the body of the class, struct, or interface) is considered an unsafe context.</span></span>
*  <span data-ttu-id="61be0-118">字段、方法、属性、事件、索引器、运算符、实例构造函数、析构函数或静态构造函数的声明可能包含 `unsafe` 修饰符，在这种情况下，该成员声明的整个文本范围被视为不安全的上下文。</span><span class="sxs-lookup"><span data-stu-id="61be0-118">A declaration of a field, method, property, event, indexer, operator, instance constructor, destructor, or static constructor may include an `unsafe` modifier, in which case the entire textual extent of that member declaration is considered an unsafe context.</span></span>
*  <span data-ttu-id="61be0-119">*Unsafe_statement*允许在*块*内使用不安全的上下文。</span><span class="sxs-lookup"><span data-stu-id="61be0-119">An *unsafe_statement* enables the use of an unsafe context within a *block*.</span></span> <span data-ttu-id="61be0-120">相关*块*的整个文本范围被视为不安全的上下文。</span><span class="sxs-lookup"><span data-stu-id="61be0-120">The entire textual extent of the associated *block* is considered an unsafe context.</span></span>

<span data-ttu-id="61be0-121">相关的语法生产如下所示。</span><span class="sxs-lookup"><span data-stu-id="61be0-121">The associated grammar productions are shown below.</span></span>

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

<span data-ttu-id="61be0-122">示例中</span><span class="sxs-lookup"><span data-stu-id="61be0-122">In the example</span></span>

```csharp
public unsafe struct Node
{
    public int Value;
    public Node* Left;
    public Node* Right;
}
```

<span data-ttu-id="61be0-123">结构声明中指定的 `unsafe` 修饰符导致结构声明的整个文本区成为不安全的上下文。</span><span class="sxs-lookup"><span data-stu-id="61be0-123">the `unsafe` modifier specified in the struct declaration causes the entire textual extent of the struct declaration to become an unsafe context.</span></span> <span data-ttu-id="61be0-124">因此，可以将 `Left` 和 `Right` 字段声明为指针类型。</span><span class="sxs-lookup"><span data-stu-id="61be0-124">Thus, it is possible to declare the `Left` and `Right` fields to be of a pointer type.</span></span> <span data-ttu-id="61be0-125">上面的示例也可以编写</span><span class="sxs-lookup"><span data-stu-id="61be0-125">The example above could also be written</span></span>

```csharp
public struct Node
{
    public int Value;
    public unsafe Node* Left;
    public unsafe Node* Right;
}
```

<span data-ttu-id="61be0-126">此处，字段声明中的 `unsafe` 修饰符将导致这些声明被视为不安全的上下文。</span><span class="sxs-lookup"><span data-stu-id="61be0-126">Here, the `unsafe` modifiers in the field declarations cause those declarations to be considered unsafe contexts.</span></span>

<span data-ttu-id="61be0-127">除了建立不安全的上下文，因而允许使用指针类型外，`unsafe` 修饰符对类型或成员不起作用。</span><span class="sxs-lookup"><span data-stu-id="61be0-127">Other than establishing an unsafe context, thus permitting the use of pointer types, the `unsafe` modifier has no effect on a type or a member.</span></span> <span data-ttu-id="61be0-128">示例中</span><span class="sxs-lookup"><span data-stu-id="61be0-128">In the example</span></span>

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

<span data-ttu-id="61be0-129">`A` 的 `F` 方法上的 `unsafe` 修饰符只是导致 `F` 的文本范围成为不安全的上下文，在此上下文中，可以使用该语言的不安全功能。</span><span class="sxs-lookup"><span data-stu-id="61be0-129">the `unsafe` modifier on the `F` method in `A` simply causes the textual extent of `F` to become an unsafe context in which the unsafe features of the language can be used.</span></span> <span data-ttu-id="61be0-130">在 `B`中的 `F` 重写中，无需重新指定 `unsafe` 修饰符--除非当然，`B` 中的 `F` 方法本身需要访问不安全功能。</span><span class="sxs-lookup"><span data-stu-id="61be0-130">In the override of `F` in `B`, there is no need to re-specify the `unsafe` modifier -- unless, of course, the `F` method in `B` itself needs access to unsafe features.</span></span>

<span data-ttu-id="61be0-131">当指针类型是方法签名的一部分时，情况会略有不同。</span><span class="sxs-lookup"><span data-stu-id="61be0-131">The situation is slightly different when a pointer type is part of the method's signature</span></span>

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

<span data-ttu-id="61be0-132">此处，因为 `F`的签名包含指针类型，所以只能在不安全的上下文中写入。</span><span class="sxs-lookup"><span data-stu-id="61be0-132">Here, because `F`'s signature includes a pointer type, it can only be written in an unsafe context.</span></span> <span data-ttu-id="61be0-133">但是，不安全上下文可通过以下方式引入：使整个类不安全（如 `A`中的情况），或在方法声明中包含 `unsafe` 修饰符，如 `B`中的情况。</span><span class="sxs-lookup"><span data-stu-id="61be0-133">However, the unsafe context can be introduced by either making the entire class unsafe, as is the case in `A`, or by including an `unsafe` modifier in the method declaration, as is the case in `B`.</span></span>

## <a name="pointer-types"></a><span data-ttu-id="61be0-134">指针类型</span><span class="sxs-lookup"><span data-stu-id="61be0-134">Pointer types</span></span>

<span data-ttu-id="61be0-135">在不安全的上下文中，*类型*（[类型](types.md)）可能是*pointer_type*以及*value_type*或*reference_type*。</span><span class="sxs-lookup"><span data-stu-id="61be0-135">In an unsafe context, a *type* ([Types](types.md)) may be a *pointer_type* as well as a *value_type* or a *reference_type*.</span></span> <span data-ttu-id="61be0-136">但是，还可以在不安全的上下文之外的 `typeof` 表达式（[匿名对象创建表达式](expressions.md#anonymous-object-creation-expressions)）中使用*pointer_type* ，因为这种用法是不安全的。</span><span class="sxs-lookup"><span data-stu-id="61be0-136">However, a *pointer_type* may also be used in a `typeof` expression ([Anonymous object creation expressions](expressions.md#anonymous-object-creation-expressions)) outside of an unsafe context as such usage is not unsafe.</span></span>

```antlr
type_unsafe
    : pointer_type
    ;
```

<span data-ttu-id="61be0-137">*Pointer_type*以*unmanaged_type*或关键字 `void`形式编写，后跟 `*` 标记：</span><span class="sxs-lookup"><span data-stu-id="61be0-137">A *pointer_type* is written as an *unmanaged_type* or the keyword `void`, followed by a `*` token:</span></span>

```antlr
pointer_type
    : unmanaged_type '*'
    | 'void' '*'
    ;

unmanaged_type
    : type
    ;
```

<span data-ttu-id="61be0-138">在指针类型中 `*` 之前指定的类型称为指针类型的***引用类型***。</span><span class="sxs-lookup"><span data-stu-id="61be0-138">The type specified before the `*` in a pointer type is called the ***referent type*** of the pointer type.</span></span> <span data-ttu-id="61be0-139">它表示指针类型值指向的变量的类型。</span><span class="sxs-lookup"><span data-stu-id="61be0-139">It represents the type of the variable to which a value of the pointer type points.</span></span>

<span data-ttu-id="61be0-140">不同于引用（引用类型的值）时，不会由垃圾回收器跟踪指针-垃圾回收器不知道指针及其指向的数据。</span><span class="sxs-lookup"><span data-stu-id="61be0-140">Unlike references (values of reference types), pointers are not tracked by the garbage collector -- the garbage collector has no knowledge of pointers and the data to which they point.</span></span> <span data-ttu-id="61be0-141">出于此原因，不允许指针指向引用或包含引用的结构，并且指针的引用类型必须是*unmanaged_type*。</span><span class="sxs-lookup"><span data-stu-id="61be0-141">For this reason a pointer is not permitted to point to a reference or to a struct that contains references, and the referent type of a pointer must be an *unmanaged_type*.</span></span>

<span data-ttu-id="61be0-142">*Unmanaged_type*是*reference_type*或构造类型之外的任何类型，并且在任何嵌套级别都不包含*reference_type*或构造类型字段。</span><span class="sxs-lookup"><span data-stu-id="61be0-142">An *unmanaged_type* is any type that isn't a *reference_type* or constructed type, and doesn't contain *reference_type* or constructed type fields at any level of nesting.</span></span> <span data-ttu-id="61be0-143">换句话说， *unmanaged_type*是以下项之一：</span><span class="sxs-lookup"><span data-stu-id="61be0-143">In other words, an *unmanaged_type* is one of the following:</span></span>

*  <span data-ttu-id="61be0-144">`sbyte`、`byte`、`short`、`ushort`、`int`、`uint`、`long`、`ulong`、`char`、`float`、`double`、`decimal`或 `bool`。</span><span class="sxs-lookup"><span data-stu-id="61be0-144">`sbyte`, `byte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`, `char`, `float`, `double`, `decimal`, or `bool`.</span></span>
*  <span data-ttu-id="61be0-145">任何*enum_type*。</span><span class="sxs-lookup"><span data-stu-id="61be0-145">Any *enum_type*.</span></span>
*  <span data-ttu-id="61be0-146">任何*pointer_type*。</span><span class="sxs-lookup"><span data-stu-id="61be0-146">Any *pointer_type*.</span></span>
*  <span data-ttu-id="61be0-147">任何不是构造类型并且只包含*unmanaged_type*的字段的用户定义的*struct_type* 。</span><span class="sxs-lookup"><span data-stu-id="61be0-147">Any user-defined *struct_type* that is not a constructed type and contains fields of *unmanaged_type*s only.</span></span>

<span data-ttu-id="61be0-148">混合使用指针和引用的直观规则是，引用（对象）的引用（对象）允许包含指针，但指针的引用不允许包含引用。</span><span class="sxs-lookup"><span data-stu-id="61be0-148">The intuitive rule for mixing of pointers and references is that referents of references (objects) are permitted to contain pointers, but referents of pointers are not permitted to contain references.</span></span>

<span data-ttu-id="61be0-149">下表给出了指针类型的一些示例：</span><span class="sxs-lookup"><span data-stu-id="61be0-149">Some examples of pointer types are given in the table below:</span></span>

| <span data-ttu-id="61be0-150">__示例__</span><span class="sxs-lookup"><span data-stu-id="61be0-150">__Example__</span></span> | <span data-ttu-id="61be0-151">__描述__</span><span class="sxs-lookup"><span data-stu-id="61be0-151">__Description__</span></span>                               |
|-------------|-----------------------------------------------|
| `byte*`     | <span data-ttu-id="61be0-152">指向 `byte` 的指针</span><span class="sxs-lookup"><span data-stu-id="61be0-152">Pointer to `byte`</span></span>                             |
| `char*`     | <span data-ttu-id="61be0-153">指向 `char` 的指针</span><span class="sxs-lookup"><span data-stu-id="61be0-153">Pointer to `char`</span></span>                             |
| `int**`     | <span data-ttu-id="61be0-154">指向 `int` 指针的指针</span><span class="sxs-lookup"><span data-stu-id="61be0-154">Pointer to pointer to `int`</span></span>                   |
| `int*[]`    | <span data-ttu-id="61be0-155">指向 `int` 的指针的一维数组</span><span class="sxs-lookup"><span data-stu-id="61be0-155">Single-dimensional array of pointers to `int`</span></span> |
| `void*`     | <span data-ttu-id="61be0-156">指向未知类型的指针</span><span class="sxs-lookup"><span data-stu-id="61be0-156">Pointer to unknown type</span></span>                       |

<span data-ttu-id="61be0-157">对于给定的实现，所有指针类型都必须具有相同的大小和表示形式。</span><span class="sxs-lookup"><span data-stu-id="61be0-157">For a given implementation, all pointer types must have the same size and representation.</span></span>

<span data-ttu-id="61be0-158">与 C 和C++不同，在同一声明中声明多个指针时， C#在 `*` 中仅与基础类型一起编写，而不作为每个指针名称的前缀标点符号。</span><span class="sxs-lookup"><span data-stu-id="61be0-158">Unlike C and C++, when multiple pointers are declared in the same declaration, in C# the `*` is written along with the underlying type only, not as a prefix punctuator on each pointer name.</span></span> <span data-ttu-id="61be0-159">例如</span><span class="sxs-lookup"><span data-stu-id="61be0-159">For example</span></span>

```csharp
int* pi, pj;    // NOT as int *pi, *pj;
```

<span data-ttu-id="61be0-160">具有类型 `T*` 的指针的值表示 `T`类型的变量的地址。</span><span class="sxs-lookup"><span data-stu-id="61be0-160">The value of a pointer having type `T*` represents the address of a variable of type `T`.</span></span> <span data-ttu-id="61be0-161">指针间接寻址运算符 `*` （[指针间接寻址](unsafe-code.md#pointer-indirection)）可用于访问此变量。</span><span class="sxs-lookup"><span data-stu-id="61be0-161">The pointer indirection operator `*` ([Pointer indirection](unsafe-code.md#pointer-indirection)) may be used to access this variable.</span></span> <span data-ttu-id="61be0-162">例如，给定 `int*`类型的变量 `P`，expression `*P` 表示在 `P`中包含的地址处找到的 `int` 变量。</span><span class="sxs-lookup"><span data-stu-id="61be0-162">For example, given a variable `P` of type `int*`, the expression `*P` denotes the `int` variable found at the address contained in `P`.</span></span>

<span data-ttu-id="61be0-163">与对象引用类似，指针可能 `null`。</span><span class="sxs-lookup"><span data-stu-id="61be0-163">Like an object reference, a pointer may be `null`.</span></span> <span data-ttu-id="61be0-164">将间接寻址运算符应用于 `null` 指针将导致实现定义的行为。</span><span class="sxs-lookup"><span data-stu-id="61be0-164">Applying the indirection operator to a `null` pointer results in implementation-defined behavior.</span></span> <span data-ttu-id="61be0-165">具有值 `null` 的指针由所有位均为零表示。</span><span class="sxs-lookup"><span data-stu-id="61be0-165">A pointer with value `null` is represented by all-bits-zero.</span></span>

<span data-ttu-id="61be0-166">`void*` 类型表示指向未知类型的指针。</span><span class="sxs-lookup"><span data-stu-id="61be0-166">The `void*` type represents a pointer to an unknown type.</span></span> <span data-ttu-id="61be0-167">由于引用类型未知，因此间接寻址运算符不能应用于类型 `void*`的指针，也不能对此类指针执行任何算术运算。</span><span class="sxs-lookup"><span data-stu-id="61be0-167">Because the referent type is unknown, the indirection operator cannot be applied to a pointer of type `void*`, nor can any arithmetic be performed on such a pointer.</span></span> <span data-ttu-id="61be0-168">但是，类型 `void*` 的指针可以转换为任何其他指针类型（反之亦然）。</span><span class="sxs-lookup"><span data-stu-id="61be0-168">However, a pointer of type `void*` can be cast to any other pointer type (and vice versa).</span></span>

<span data-ttu-id="61be0-169">指针类型是一种单独的类型类别。</span><span class="sxs-lookup"><span data-stu-id="61be0-169">Pointer types are a separate category of types.</span></span> <span data-ttu-id="61be0-170">与引用类型和值类型不同，指针类型不从 `object` 继承，并且指针类型和 `object`之间不存在转换。</span><span class="sxs-lookup"><span data-stu-id="61be0-170">Unlike reference types and value types, pointer types do not inherit from `object` and no conversions exist between pointer types and `object`.</span></span> <span data-ttu-id="61be0-171">特别是，指针不支持装箱和取消装箱（[装箱和取消](types.md#boxing-and-unboxing)装箱）。</span><span class="sxs-lookup"><span data-stu-id="61be0-171">In particular, boxing and unboxing ([Boxing and unboxing](types.md#boxing-and-unboxing)) are not supported for pointers.</span></span> <span data-ttu-id="61be0-172">但允许在不同指针类型之间以及指针类型和整型之间进行转换。</span><span class="sxs-lookup"><span data-stu-id="61be0-172">However, conversions are permitted between different pointer types and between pointer types and the integral types.</span></span> <span data-ttu-id="61be0-173">[指针转换](unsafe-code.md#pointer-conversions)中对此进行了说明。</span><span class="sxs-lookup"><span data-stu-id="61be0-173">This is described in [Pointer conversions](unsafe-code.md#pointer-conversions).</span></span>

<span data-ttu-id="61be0-174">*Pointer_type*不能用作类型参数（[构造类型](types.md#constructed-types)），并且类型推理（[类型推理](expressions.md#type-inference)）对泛型方法调用失败，而这种方法调用会将类型参数推断为指针类型。</span><span class="sxs-lookup"><span data-stu-id="61be0-174">A *pointer_type* cannot be used as a type argument ([Constructed types](types.md#constructed-types)), and type inference ([Type inference](expressions.md#type-inference)) fails on generic method calls that would have inferred a type argument to be a pointer type.</span></span>

<span data-ttu-id="61be0-175">*Pointer_type*可以用作可变字段（[可变字段](classes.md#volatile-fields)）的类型。</span><span class="sxs-lookup"><span data-stu-id="61be0-175">A *pointer_type* may be used as the type of a volatile field ([Volatile fields](classes.md#volatile-fields)).</span></span>

<span data-ttu-id="61be0-176">尽管指针可以作为 `ref` 或 `out` 参数传递，但这样做可能会导致未定义的行为，因为在调用的方法返回时，指针可能设置为指向某个局部变量，而在调用的方法返回时，或其所指向的固定对象不再是固定的。</span><span class="sxs-lookup"><span data-stu-id="61be0-176">Although pointers can be passed as `ref` or `out` parameters, doing so can cause undefined behavior, since the pointer may well be set to point to a local variable which no longer exists when the called method returns, or the fixed object to which it used to point, is no longer fixed.</span></span> <span data-ttu-id="61be0-177">例如：</span><span class="sxs-lookup"><span data-stu-id="61be0-177">For example:</span></span>

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

<span data-ttu-id="61be0-178">方法可以返回某种类型的值，并且该类型可以是指针。</span><span class="sxs-lookup"><span data-stu-id="61be0-178">A method can return a value of some type, and that type can be a pointer.</span></span> <span data-ttu-id="61be0-179">例如，当给定指向连续 `int`s 序列的指针、序列的元素计数以及其他 `int` 值时，如果发生匹配，以下方法将返回该序列中该值的地址;否则，它将返回 `null`：</span><span class="sxs-lookup"><span data-stu-id="61be0-179">For example, when given a pointer to a contiguous sequence of `int`s, that sequence's element count, and some other `int` value, the following method returns the address of that value in that sequence, if a match occurs; otherwise it returns `null`:</span></span>

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

<span data-ttu-id="61be0-180">在不安全的上下文中，有多个构造可用于对指针进行操作：</span><span class="sxs-lookup"><span data-stu-id="61be0-180">In an unsafe context, several constructs are available for operating on pointers:</span></span>

*  <span data-ttu-id="61be0-181">`*` 运算符可用于执行指针间接寻址（[指针间接](unsafe-code.md#pointer-indirection)寻址）。</span><span class="sxs-lookup"><span data-stu-id="61be0-181">The `*` operator may be used to perform pointer indirection ([Pointer indirection](unsafe-code.md#pointer-indirection)).</span></span>
*  <span data-ttu-id="61be0-182">`->` 运算符可用于通过指针（[指针成员访问](unsafe-code.md#pointer-member-access)）访问结构的成员。</span><span class="sxs-lookup"><span data-stu-id="61be0-182">The `->` operator may be used to access a member of a struct through a pointer ([Pointer member access](unsafe-code.md#pointer-member-access)).</span></span>
*  <span data-ttu-id="61be0-183">`[]` 运算符可用于对指针（[指针元素访问](unsafe-code.md#pointer-element-access)）进行索引。</span><span class="sxs-lookup"><span data-stu-id="61be0-183">The `[]` operator may be used to index a pointer ([Pointer element access](unsafe-code.md#pointer-element-access)).</span></span>
*  <span data-ttu-id="61be0-184">`&` 运算符可用于获取变量的地址（[运算符的地址](unsafe-code.md#the-address-of-operator)）。</span><span class="sxs-lookup"><span data-stu-id="61be0-184">The `&` operator may be used to obtain the address of a variable ([The address-of operator](unsafe-code.md#the-address-of-operator)).</span></span>
*  <span data-ttu-id="61be0-185">`++` 和 `--` 运算符可用来递增和递减指针（[指针递增和递减](unsafe-code.md#pointer-increment-and-decrement)）。</span><span class="sxs-lookup"><span data-stu-id="61be0-185">The `++` and `--` operators may be used to increment and decrement pointers ([Pointer increment and decrement](unsafe-code.md#pointer-increment-and-decrement)).</span></span>
*  <span data-ttu-id="61be0-186">`+` 和 `-` 运算符可用于执行指针算法（[指针算法](unsafe-code.md#pointer-arithmetic)）。</span><span class="sxs-lookup"><span data-stu-id="61be0-186">The `+` and `-` operators may be used to perform pointer arithmetic ([Pointer arithmetic](unsafe-code.md#pointer-arithmetic)).</span></span>
*  <span data-ttu-id="61be0-187">可以使用 `==`、`!=`、`<`、`>`、`<=`和 `=>` 运算符来比较指针（[指针比较](unsafe-code.md#pointer-comparison)）。</span><span class="sxs-lookup"><span data-stu-id="61be0-187">The `==`, `!=`, `<`, `>`, `<=`, and `=>` operators may be used to compare pointers ([Pointer comparison](unsafe-code.md#pointer-comparison)).</span></span>
*  <span data-ttu-id="61be0-188">`stackalloc` 运算符可用于从调用堆栈（[固定大小缓冲区](unsafe-code.md#fixed-size-buffers)）分配内存。</span><span class="sxs-lookup"><span data-stu-id="61be0-188">The `stackalloc` operator may be used to allocate memory from the call stack ([Fixed size buffers](unsafe-code.md#fixed-size-buffers)).</span></span>
*  <span data-ttu-id="61be0-189">可以使用 `fixed` 语句来暂时修复变量，以便可以获取其地址（[fixed 语句](unsafe-code.md#the-fixed-statement)）。</span><span class="sxs-lookup"><span data-stu-id="61be0-189">The `fixed` statement may be used to temporarily fix a variable so its address can be obtained ([The fixed statement](unsafe-code.md#the-fixed-statement)).</span></span>

## <a name="fixed-and-moveable-variables"></a><span data-ttu-id="61be0-190">固定和可移动变量</span><span class="sxs-lookup"><span data-stu-id="61be0-190">Fixed and moveable variables</span></span>

<span data-ttu-id="61be0-191">Address 运算符（[address 运算符](unsafe-code.md#the-address-of-operator)）和 `fixed` 语句（[fixed 语句](unsafe-code.md#the-fixed-statement)）将变量分为两类：***固定变量***和***可移动变量***。</span><span class="sxs-lookup"><span data-stu-id="61be0-191">The address-of operator ([The address-of operator](unsafe-code.md#the-address-of-operator)) and the `fixed` statement ([The fixed statement](unsafe-code.md#the-fixed-statement)) divide variables into two categories: ***Fixed variables*** and ***moveable variables***.</span></span>

<span data-ttu-id="61be0-192">固定变量驻留在不受垃圾回收器操作影响的存储位置中。</span><span class="sxs-lookup"><span data-stu-id="61be0-192">Fixed variables reside in storage locations that are unaffected by operation of the garbage collector.</span></span> <span data-ttu-id="61be0-193">（固定变量的示例包括局部变量、值参数和通过取消引用指针而创建的变量。）另一方面，可移动变量驻留在由垃圾回收器进行重定位或处置的存储位置。</span><span class="sxs-lookup"><span data-stu-id="61be0-193">(Examples of fixed variables include local variables, value parameters, and variables created by dereferencing pointers.) On the other hand, moveable variables reside in storage locations that are subject to relocation or disposal by the garbage collector.</span></span> <span data-ttu-id="61be0-194">（可移动变量的示例包括对象中的字段和数组的元素。）</span><span class="sxs-lookup"><span data-stu-id="61be0-194">(Examples of moveable variables include fields in objects and elements of arrays.)</span></span>

<span data-ttu-id="61be0-195">`&` 运算符（[地址运算符](unsafe-code.md#the-address-of-operator)）允许在不限制的情况下获取固定变量的地址。</span><span class="sxs-lookup"><span data-stu-id="61be0-195">The `&` operator ([The address-of operator](unsafe-code.md#the-address-of-operator)) permits the address of a fixed variable to be obtained without restrictions.</span></span> <span data-ttu-id="61be0-196">但是，由于可移动变量可能会由垃圾回收器重定位或处置，因此只能使用 `fixed` 语句（[fixed 语句](unsafe-code.md#the-fixed-statement)）获取可移动变量的地址，并且该地址在 `fixed` 语句的持续时间内保持有效。</span><span class="sxs-lookup"><span data-stu-id="61be0-196">However, because a moveable variable is subject to relocation or disposal by the garbage collector, the address of a moveable variable can only be obtained using a `fixed` statement ([The fixed statement](unsafe-code.md#the-fixed-statement)), and that address remains valid only for the duration of that `fixed` statement.</span></span>

<span data-ttu-id="61be0-197">确切地说，固定变量是以下项之一：</span><span class="sxs-lookup"><span data-stu-id="61be0-197">In precise terms, a fixed variable is one of the following:</span></span>

*  <span data-ttu-id="61be0-198">由引用本地变量或值参数的*simple_name* （[简单名称](expressions.md#simple-names)）产生的变量，除非该变量是由匿名函数捕获的。</span><span class="sxs-lookup"><span data-stu-id="61be0-198">A variable resulting from a *simple_name* ([Simple names](expressions.md#simple-names)) that refers to a local variable or a value parameter, unless the variable is captured by an anonymous function.</span></span>
*  <span data-ttu-id="61be0-199">由 `V.I`格式的*member_access* （[成员访问](expressions.md#member-access)）产生的变量，其中 `V` 是*struct_type*的固定变量。</span><span class="sxs-lookup"><span data-stu-id="61be0-199">A variable resulting from a *member_access* ([Member access](expressions.md#member-access)) of the form `V.I`, where `V` is a fixed variable of a *struct_type*.</span></span>
*  <span data-ttu-id="61be0-200">由 `*P`的窗体的*pointer_indirection_expression* （[指针间接寻址](unsafe-code.md#pointer-indirection)）、`P->I`形式的*pointer_member_access* （[指针成员访问](unsafe-code.md#pointer-member-access)）或 pointer_element_access 形式的 *`P[E]`* （[指针元素访问](unsafe-code.md#pointer-element-access)）生成的变量。</span><span class="sxs-lookup"><span data-stu-id="61be0-200">A variable resulting from a *pointer_indirection_expression* ([Pointer indirection](unsafe-code.md#pointer-indirection)) of the form `*P`, a *pointer_member_access* ([Pointer member access](unsafe-code.md#pointer-member-access)) of the form `P->I`, or a *pointer_element_access* ([Pointer element access](unsafe-code.md#pointer-element-access)) of the form `P[E]`.</span></span>

<span data-ttu-id="61be0-201">所有其他变量归类为可移动变量。</span><span class="sxs-lookup"><span data-stu-id="61be0-201">All other variables are classified as moveable variables.</span></span>

<span data-ttu-id="61be0-202">请注意，静态字段归类为可移动变量。</span><span class="sxs-lookup"><span data-stu-id="61be0-202">Note that a static field is classified as a moveable variable.</span></span> <span data-ttu-id="61be0-203">另请注意，`ref` 或 `out` 参数归类为可移动变量，即使为参数提供的参数是固定变量。</span><span class="sxs-lookup"><span data-stu-id="61be0-203">Also note that a `ref` or `out` parameter is classified as a moveable variable, even if the argument given for the parameter is a fixed variable.</span></span> <span data-ttu-id="61be0-204">最后请注意，通过取消引用指针生成的变量始终归类为固定变量。</span><span class="sxs-lookup"><span data-stu-id="61be0-204">Finally, note that a variable produced by dereferencing a pointer is always classified as a fixed variable.</span></span>

## <a name="pointer-conversions"></a><span data-ttu-id="61be0-205">指针转换</span><span class="sxs-lookup"><span data-stu-id="61be0-205">Pointer conversions</span></span>

<span data-ttu-id="61be0-206">在不安全的上下文中，可以使用隐式转换集（[隐式转换](conversions.md#implicit-conversions)）进行扩展，以包括以下隐式指针转换：</span><span class="sxs-lookup"><span data-stu-id="61be0-206">In an unsafe context, the set of available implicit conversions ([Implicit conversions](conversions.md#implicit-conversions)) is extended to include the following implicit pointer conversions:</span></span>

*  <span data-ttu-id="61be0-207">从任何*pointer_type*到类型的 `void*`。</span><span class="sxs-lookup"><span data-stu-id="61be0-207">From any *pointer_type* to the type `void*`.</span></span>
*  <span data-ttu-id="61be0-208">从 `null` 文本到任何*pointer_type*。</span><span class="sxs-lookup"><span data-stu-id="61be0-208">From the `null` literal to any *pointer_type*.</span></span>

<span data-ttu-id="61be0-209">此外，在不安全的上下文中，可使用的显式转换集（[显式转换](conversions.md#explicit-conversions)）进行扩展，以包括以下显式指针转换：</span><span class="sxs-lookup"><span data-stu-id="61be0-209">Additionally, in an unsafe context, the set of available explicit conversions ([Explicit conversions](conversions.md#explicit-conversions)) is extended to include the following explicit pointer conversions:</span></span>

*  <span data-ttu-id="61be0-210">从任何*pointer_type*到任何其他*pointer_type*。</span><span class="sxs-lookup"><span data-stu-id="61be0-210">From any *pointer_type* to any other *pointer_type*.</span></span>
*  <span data-ttu-id="61be0-211">从 `sbyte`、`byte`、`short`、`ushort`、`int`、`uint`、`long`或 `ulong` 到任何*pointer_type*。</span><span class="sxs-lookup"><span data-stu-id="61be0-211">From `sbyte`, `byte`, `short`, `ushort`, `int`, `uint`, `long`, or `ulong` to any *pointer_type*.</span></span>
*  <span data-ttu-id="61be0-212">从任何*pointer_type*到 `sbyte`、`byte`、`short`、`ushort`、`int`、`uint`、`long`或 `ulong`。</span><span class="sxs-lookup"><span data-stu-id="61be0-212">From any *pointer_type* to `sbyte`, `byte`, `short`, `ushort`, `int`, `uint`, `long`, or `ulong`.</span></span>

<span data-ttu-id="61be0-213">最后，在不安全的上下文中，标准隐式转换集（[标准隐式转换](conversions.md#standard-implicit-conversions)）包括以下指针转换：</span><span class="sxs-lookup"><span data-stu-id="61be0-213">Finally, in an unsafe context, the set of standard implicit conversions ([Standard implicit conversions](conversions.md#standard-implicit-conversions)) includes the following pointer conversion:</span></span>

*  <span data-ttu-id="61be0-214">从任何*pointer_type*到类型的 `void*`。</span><span class="sxs-lookup"><span data-stu-id="61be0-214">From any *pointer_type* to the type `void*`.</span></span>

<span data-ttu-id="61be0-215">两个指针类型之间的转换决不会更改实际指针的值。</span><span class="sxs-lookup"><span data-stu-id="61be0-215">Conversions between two pointer types never change the actual pointer value.</span></span> <span data-ttu-id="61be0-216">换句话说，从一种指针类型到另一种类型的转换不会影响由指针提供的基础地址。</span><span class="sxs-lookup"><span data-stu-id="61be0-216">In other words, a conversion from one pointer type to another has no effect on the underlying address given by the pointer.</span></span>

<span data-ttu-id="61be0-217">当一种指针类型转换为另一种类型时，如果结果指针没有正确对齐所引用的类型，则该行为是不确定的。</span><span class="sxs-lookup"><span data-stu-id="61be0-217">When one pointer type is converted to another, if the resulting pointer is not correctly aligned for the pointed-to type, the behavior is undefined if the result is dereferenced.</span></span> <span data-ttu-id="61be0-218">通常，"正确对齐" 这一概念是可传递的：如果指向类型 `A` 的指针为指向类型 `B`的指针正确对齐，而后者又为指向类型 `C`的指针正确对齐，则指向类型 `A` 的指针将正确对齐到类型 `C`的指针。</span><span class="sxs-lookup"><span data-stu-id="61be0-218">In general, the concept "correctly aligned" is transitive: if a pointer to type `A` is correctly aligned for a pointer to type `B`, which, in turn, is correctly aligned for a pointer to type `C`, then a pointer to type `A` is correctly aligned for a pointer to type `C`.</span></span>

<span data-ttu-id="61be0-219">请考虑以下情况：通过指向其他类型的指针访问具有一种类型的变量：</span><span class="sxs-lookup"><span data-stu-id="61be0-219">Consider the following case in which a variable having one type is accessed via a pointer to a different type:</span></span>

```csharp
char c = 'A';
char* pc = &c;
void* pv = pc;
int* pi = (int*)pv;
int i = *pi;         // undefined
*pi = 123456;        // undefined
```

<span data-ttu-id="61be0-220">将指针类型转换为指向字节的指针时，结果将指向该变量的最低寻址字节。</span><span class="sxs-lookup"><span data-stu-id="61be0-220">When a pointer type is converted to a pointer to byte, the result points to the lowest addressed byte of the variable.</span></span> <span data-ttu-id="61be0-221">结果的后续增量，最大为变量的大小，产生指向该变量的其余字节的指针。</span><span class="sxs-lookup"><span data-stu-id="61be0-221">Successive increments of the result, up to the size of the variable, yield pointers to the remaining bytes of that variable.</span></span> <span data-ttu-id="61be0-222">例如，下面的方法以十六进制值的形式显示 double 中的八个字节中的每个字节：</span><span class="sxs-lookup"><span data-stu-id="61be0-222">For example, the following method displays each of the eight bytes in a double as a hexadecimal value:</span></span>

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

<span data-ttu-id="61be0-223">当然，生成的输出取决于 endian。</span><span class="sxs-lookup"><span data-stu-id="61be0-223">Of course, the output produced depends on endianness.</span></span>

<span data-ttu-id="61be0-224">指针和整数之间的映射是实现定义的。</span><span class="sxs-lookup"><span data-stu-id="61be0-224">Mappings between pointers and integers are implementation-defined.</span></span> <span data-ttu-id="61be0-225">但是，在带有线性地址空间的 32 \* 和64位 CPU 体系结构上，转换到整数类型或从整型转换到整数类型的方式通常与 `uint` 或 `ulong` 值的转换方式完全相同。</span><span class="sxs-lookup"><span data-stu-id="61be0-225">However, on 32\* and 64-bit CPU architectures with a linear address space, conversions of pointers to or from integral types typically behave exactly like conversions of `uint` or `ulong` values, respectively, to or from those integral types.</span></span>

### <a name="pointer-arrays"></a><span data-ttu-id="61be0-226">指针数组</span><span class="sxs-lookup"><span data-stu-id="61be0-226">Pointer arrays</span></span>

<span data-ttu-id="61be0-227">在不安全的上下文中，可以构造指针的数组。</span><span class="sxs-lookup"><span data-stu-id="61be0-227">In an unsafe context, arrays of pointers can be constructed.</span></span> <span data-ttu-id="61be0-228">对于指针数组，只允许使用某些适用于其他数组类型的转换：</span><span class="sxs-lookup"><span data-stu-id="61be0-228">Only some of the conversions that apply to other array types are allowed on pointer arrays:</span></span>

*  <span data-ttu-id="61be0-229">从任何*array_type*到 `System.Array` 以及它所实现的接口的隐[式引用转换](conversions.md#implicit-reference-conversions)也适用于指针数组。</span><span class="sxs-lookup"><span data-stu-id="61be0-229">The implicit reference conversion ([Implicit reference conversions](conversions.md#implicit-reference-conversions)) from any *array_type* to `System.Array` and the interfaces it implements also applies to pointer arrays.</span></span> <span data-ttu-id="61be0-230">但是，通过 `System.Array` 或它实现的接口访问数组元素的任何尝试都将导致运行时异常，因为指针类型无法转换为 `object`。</span><span class="sxs-lookup"><span data-stu-id="61be0-230">However, any attempt to access the array elements through `System.Array` or the interfaces it implements will result in an exception at run-time, as pointer types are not convertible to `object`.</span></span>
*  <span data-ttu-id="61be0-231">从一维数组类型 `S[]` 隐式和显式引用转换（[隐式](conversions.md#implicit-reference-conversions)引用转换、[显式引用转换](conversions.md#explicit-reference-conversions)）到 `System.Collections.Generic.IList<T>` 及其泛型基接口从不应用于指针数组，因为指针类型不能用作类型参数，并且不能从指针类型转换为非指针类型。</span><span class="sxs-lookup"><span data-stu-id="61be0-231">The implicit and explicit reference conversions ([Implicit reference conversions](conversions.md#implicit-reference-conversions), [Explicit reference conversions](conversions.md#explicit-reference-conversions)) from a single-dimensional array type `S[]` to `System.Collections.Generic.IList<T>` and its generic base interfaces never apply to pointer arrays, since pointer types cannot be used as type arguments, and there are no conversions from pointer types to non-pointer types.</span></span>
*  <span data-ttu-id="61be0-232">从 `System.Array` 显式引用转换（[显式引用](conversions.md#explicit-reference-conversions)转换）以及它为任何*array_type*实现的接口适用于指针数组。</span><span class="sxs-lookup"><span data-stu-id="61be0-232">The explicit reference conversion ([Explicit reference conversions](conversions.md#explicit-reference-conversions)) from `System.Array` and the interfaces it implements to any *array_type* applies to pointer arrays.</span></span>
*  <span data-ttu-id="61be0-233">从 `System.Collections.Generic.IList<S>` 将显式引用转换（[显式引用转换](conversions.md#explicit-reference-conversions)）及其基接口转换为一维数组类型 `T[]` 从不适用于指针数组，因为指针类型不能用作类型参数，并且不能从指针类型转换为非指针类型。</span><span class="sxs-lookup"><span data-stu-id="61be0-233">The explicit reference conversions ([Explicit reference conversions](conversions.md#explicit-reference-conversions)) from `System.Collections.Generic.IList<S>` and its base interfaces to a single-dimensional array type `T[]` never applies to pointer arrays, since pointer types cannot be used as type arguments, and there are no conversions from pointer types to non-pointer types.</span></span>

<span data-ttu-id="61be0-234">这些限制意味着对[foreach 语句](statements.md#the-foreach-statement)中所述数组的 `foreach` 语句的扩展不能应用于指针数组。</span><span class="sxs-lookup"><span data-stu-id="61be0-234">These restrictions mean that the expansion for the `foreach` statement over arrays described in [The foreach statement](statements.md#the-foreach-statement) cannot be applied to pointer arrays.</span></span> <span data-ttu-id="61be0-235">相反，窗体的 foreach 语句</span><span class="sxs-lookup"><span data-stu-id="61be0-235">Instead, a foreach statement of the form</span></span>

```csharp
foreach (V v in x) embedded_statement
```

<span data-ttu-id="61be0-236">其中，`x` 的类型为 `T[,,...,]`形式的数组类型，`N` 为维数减1，`T` 或 `V` 是指针类型，使用嵌套的 for 循环进行扩展，如下所示：</span><span class="sxs-lookup"><span data-stu-id="61be0-236">where the type of `x` is an array type of the form `T[,,...,]`, `N` is the number of dimensions minus 1 and `T` or `V` is a pointer type, is expanded using nested for-loops as follows:</span></span>

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

<span data-ttu-id="61be0-237">`a`、`i0`、`i1`、...、`iN` 的变量对 `x` 或*程序的任何*其他源代码都不可见或不可访问。</span><span class="sxs-lookup"><span data-stu-id="61be0-237">The variables `a`, `i0`, `i1`, ..., `iN` are not visible to or accessible to `x` or the *embedded_statement* or any other source code of the program.</span></span> <span data-ttu-id="61be0-238">变量 `v` 在嵌入语句中是只读的。</span><span class="sxs-lookup"><span data-stu-id="61be0-238">The variable `v` is read-only in the embedded statement.</span></span> <span data-ttu-id="61be0-239">如果没有从 `T` （元素类型）到 `V`的显式转换（[指针转换](unsafe-code.md#pointer-conversions)），则会生成错误，并且不会执行任何其他步骤。</span><span class="sxs-lookup"><span data-stu-id="61be0-239">If there is not an explicit conversion ([Pointer conversions](unsafe-code.md#pointer-conversions)) from `T` (the element type) to `V`, an error is produced and no further steps are taken.</span></span> <span data-ttu-id="61be0-240">如果 `x` 的值 `null`，则在运行时将引发 `System.NullReferenceException`。</span><span class="sxs-lookup"><span data-stu-id="61be0-240">If `x` has the value `null`, a `System.NullReferenceException` is thrown at run-time.</span></span>

## <a name="pointers-in-expressions"></a><span data-ttu-id="61be0-241">表达式中的指针</span><span class="sxs-lookup"><span data-stu-id="61be0-241">Pointers in expressions</span></span>

<span data-ttu-id="61be0-242">在不安全的上下文中，表达式可能产生指针类型的结果，但在不安全的上下文外部，表达式是指针类型的编译时错误。</span><span class="sxs-lookup"><span data-stu-id="61be0-242">In an unsafe context, an expression may yield a result of a pointer type, but outside an unsafe context it is a compile-time error for an expression to be of a pointer type.</span></span> <span data-ttu-id="61be0-243">在不安全的上下文中，如果任何*simple_name* （[简单名称](expressions.md#simple-names)）、 *member_access* （[成员访问](expressions.md#member-access)）、 *invocation_expression* （[调用表达式](expressions.md#invocation-expressions)）或*element_access* （[元素访问](expressions.md#element-access)）是指针类型，则会发生编译时错误。</span><span class="sxs-lookup"><span data-stu-id="61be0-243">In precise terms, outside an unsafe context a compile-time error occurs if any *simple_name* ([Simple names](expressions.md#simple-names)), *member_access* ([Member access](expressions.md#member-access)), *invocation_expression* ([Invocation expressions](expressions.md#invocation-expressions)), or *element_access* ([Element access](expressions.md#element-access)) is of a pointer type.</span></span>

<span data-ttu-id="61be0-244">在不安全的上下文中， *primary_no_array_creation_expression* （[主表达式](expressions.md#primary-expressions)）和*unary_expression* （[一元运算符](expressions.md#unary-operators)）生产允许以下附加构造：</span><span class="sxs-lookup"><span data-stu-id="61be0-244">In an unsafe context, the *primary_no_array_creation_expression* ([Primary expressions](expressions.md#primary-expressions)) and *unary_expression* ([Unary operators](expressions.md#unary-operators)) productions permit the following additional constructs:</span></span>

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

<span data-ttu-id="61be0-245">以下各节介绍了这些构造。</span><span class="sxs-lookup"><span data-stu-id="61be0-245">These constructs are described in the following sections.</span></span> <span data-ttu-id="61be0-246">语法暗示了 unsafe 运算符的优先级和关联性。</span><span class="sxs-lookup"><span data-stu-id="61be0-246">The precedence and associativity of the unsafe operators is implied by the grammar.</span></span>

### <a name="pointer-indirection"></a><span data-ttu-id="61be0-247">指针间接寻址</span><span class="sxs-lookup"><span data-stu-id="61be0-247">Pointer indirection</span></span>

<span data-ttu-id="61be0-248">*Pointer_indirection_expression*包含一个星号（`*`），后跟一个*unary_expression*。</span><span class="sxs-lookup"><span data-stu-id="61be0-248">A *pointer_indirection_expression* consists of an asterisk (`*`) followed by a *unary_expression*.</span></span>

```antlr
pointer_indirection_expression
    : '*' unary_expression
    ;
```

<span data-ttu-id="61be0-249">一元 `*` 运算符表示指针间接寻址，用于获取指针所指向的变量。</span><span class="sxs-lookup"><span data-stu-id="61be0-249">The unary `*` operator denotes pointer indirection and is used to obtain the variable to which a pointer points.</span></span> <span data-ttu-id="61be0-250">计算 `*P`的结果（其中 `P` 是指针类型 `T*`的表达式）是 `T`类型的变量。</span><span class="sxs-lookup"><span data-stu-id="61be0-250">The result of evaluating `*P`, where `P` is an expression of a pointer type `T*`, is a variable of type `T`.</span></span> <span data-ttu-id="61be0-251">将一元 `*` 运算符应用到 `void*` 类型的表达式或不是指针类型的表达式时，会发生编译时错误。</span><span class="sxs-lookup"><span data-stu-id="61be0-251">It is a compile-time error to apply the unary `*` operator to an expression of type `void*` or to an expression that isn't of a pointer type.</span></span>

<span data-ttu-id="61be0-252">向 `null` 指针应用一元 `*` 运算符的效果是实现定义的。</span><span class="sxs-lookup"><span data-stu-id="61be0-252">The effect of applying the unary `*` operator to a `null` pointer is implementation-defined.</span></span> <span data-ttu-id="61be0-253">特别是，不能保证此操作引发 `System.NullReferenceException`。</span><span class="sxs-lookup"><span data-stu-id="61be0-253">In particular, there is no guarantee that this operation throws a `System.NullReferenceException`.</span></span>

<span data-ttu-id="61be0-254">如果向指针分配了无效的值，则一元 `*` 运算符的行为是不确定的。</span><span class="sxs-lookup"><span data-stu-id="61be0-254">If an invalid value has been assigned to the pointer, the behavior of the unary `*` operator is undefined.</span></span> <span data-ttu-id="61be0-255">用于通过一元 `*` 运算符取消对指针的引用的无效值是为指向的类型（请参阅[指针转换](unsafe-code.md#pointer-conversions)中的示例）的不一致的地址，以及变量生存期结束后的变量地址。</span><span class="sxs-lookup"><span data-stu-id="61be0-255">Among the invalid values for dereferencing a pointer by the unary `*` operator are an address inappropriately aligned for the type pointed to (see example in [Pointer conversions](unsafe-code.md#pointer-conversions)), and the address of a variable after the end of its lifetime.</span></span>

<span data-ttu-id="61be0-256">出于明确的赋值分析目的，通过计算形式 `*P` 的表达式所生成的变量被视为初始赋值（[最初分配的变量](variables.md#initially-assigned-variables)）。</span><span class="sxs-lookup"><span data-stu-id="61be0-256">For purposes of definite assignment analysis, a variable produced by evaluating an expression of the form `*P` is considered initially assigned ([Initially assigned variables](variables.md#initially-assigned-variables)).</span></span>

### <a name="pointer-member-access"></a><span data-ttu-id="61be0-257">指针成员访问</span><span class="sxs-lookup"><span data-stu-id="61be0-257">Pointer member access</span></span>

<span data-ttu-id="61be0-258">*Pointer_member_access*包含一个*primary_expression*，后跟一个 "`->`" 标记，后跟一个*标识符*和一个可选的*type_argument_list*。</span><span class="sxs-lookup"><span data-stu-id="61be0-258">A *pointer_member_access* consists of a *primary_expression*, followed by a "`->`" token, followed by an *identifier* and an optional *type_argument_list*.</span></span>

```antlr
pointer_member_access
    : primary_expression '->' identifier
    ;
```

<span data-ttu-id="61be0-259">在形式 `P->I`的指针成员访问中，`P` 必须是除 `void*`之外的指针类型的表达式，并且 `I` 必须表示 `P` 点的类型的可访问成员。</span><span class="sxs-lookup"><span data-stu-id="61be0-259">In a pointer member access of the form `P->I`, `P` must be an expression of a pointer type other than `void*`, and `I` must denote an accessible member of the type to which `P` points.</span></span>

<span data-ttu-id="61be0-260">形式 `P->I` 的指针成员访问权限完全按 `(*P).I`进行计算。</span><span class="sxs-lookup"><span data-stu-id="61be0-260">A pointer member access of the form `P->I` is evaluated exactly as `(*P).I`.</span></span> <span data-ttu-id="61be0-261">有关指针间接寻址运算符（`*`）的说明，请参阅[指针间接寻址](unsafe-code.md#pointer-indirection)。</span><span class="sxs-lookup"><span data-stu-id="61be0-261">For a description of the pointer indirection operator (`*`), see [Pointer indirection](unsafe-code.md#pointer-indirection).</span></span> <span data-ttu-id="61be0-262">有关成员访问运算符（`.`）的说明，请参阅[成员访问](expressions.md#member-access)。</span><span class="sxs-lookup"><span data-stu-id="61be0-262">For a description of the member access operator (`.`), see [Member access](expressions.md#member-access).</span></span>

<span data-ttu-id="61be0-263">示例中</span><span class="sxs-lookup"><span data-stu-id="61be0-263">In the example</span></span>

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

<span data-ttu-id="61be0-264">`->` 运算符用于访问字段并通过指针调用结构的方法。</span><span class="sxs-lookup"><span data-stu-id="61be0-264">the `->` operator is used to access fields and invoke a method of a struct through a pointer.</span></span> <span data-ttu-id="61be0-265">由于 `P->I` 的操作完全等同于 `(*P).I`，因此，`Main` 方法的编写方式同样好：</span><span class="sxs-lookup"><span data-stu-id="61be0-265">Because the operation `P->I` is precisely equivalent to `(*P).I`, the `Main` method could equally well have been written:</span></span>

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

### <a name="pointer-element-access"></a><span data-ttu-id="61be0-266">指针元素访问</span><span class="sxs-lookup"><span data-stu-id="61be0-266">Pointer element access</span></span>

<span data-ttu-id="61be0-267">*Pointer_element_access*包含一个*primary_no_array_creation_expression*后跟一个括在 "`[`" 和 "`]`" 的表达式。</span><span class="sxs-lookup"><span data-stu-id="61be0-267">A *pointer_element_access* consists of a *primary_no_array_creation_expression* followed by an expression enclosed in "`[`" and "`]`".</span></span>

```antlr
pointer_element_access
    : primary_no_array_creation_expression '[' expression ']'
    ;
```

<span data-ttu-id="61be0-268">在 `P[E]`形式的指针元素访问中，`P` 必须是 `void*`的指针类型的表达式，而 `E` 必须是可以隐式转换为 `int`、`uint`、`long`或 `ulong`的表达式。</span><span class="sxs-lookup"><span data-stu-id="61be0-268">In a pointer element access of the form `P[E]`, `P` must be an expression of a pointer type other than `void*`, and `E` must be an expression that can be implicitly converted to `int`, `uint`, `long`, or `ulong`.</span></span>

<span data-ttu-id="61be0-269">`P[E]` 的窗体的指针元素访问都精确地按 `*(P + E)`进行计算。</span><span class="sxs-lookup"><span data-stu-id="61be0-269">A pointer element access of the form `P[E]` is evaluated exactly as `*(P + E)`.</span></span> <span data-ttu-id="61be0-270">有关指针间接寻址运算符（`*`）的说明，请参阅[指针间接寻址](unsafe-code.md#pointer-indirection)。</span><span class="sxs-lookup"><span data-stu-id="61be0-270">For a description of the pointer indirection operator (`*`), see [Pointer indirection](unsafe-code.md#pointer-indirection).</span></span> <span data-ttu-id="61be0-271">有关指针加法运算符（`+`）的说明，请参阅[指针算法](unsafe-code.md#pointer-arithmetic)。</span><span class="sxs-lookup"><span data-stu-id="61be0-271">For a description of the pointer addition operator (`+`), see [Pointer arithmetic](unsafe-code.md#pointer-arithmetic).</span></span>

<span data-ttu-id="61be0-272">示例中</span><span class="sxs-lookup"><span data-stu-id="61be0-272">In the example</span></span>

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

<span data-ttu-id="61be0-273">指针元素访问用于在 `for` 循环中初始化字符缓冲区。</span><span class="sxs-lookup"><span data-stu-id="61be0-273">a pointer element access is used to initialize the character buffer in a `for` loop.</span></span> <span data-ttu-id="61be0-274">由于 `P[E]` 的操作完全等同于 `*(P + E)`，因此，该示例可能同样已写入：</span><span class="sxs-lookup"><span data-stu-id="61be0-274">Because the operation `P[E]` is precisely equivalent to `*(P + E)`, the example could equally well have been written:</span></span>

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

<span data-ttu-id="61be0-275">指针元素访问运算符不会检查超出界限的错误，并且在访问超出界限的元素时的行为不确定。</span><span class="sxs-lookup"><span data-stu-id="61be0-275">The pointer element access operator does not check for out-of-bounds errors and the behavior when accessing an out-of-bounds element is undefined.</span></span> <span data-ttu-id="61be0-276">这与 C 和C++相同。</span><span class="sxs-lookup"><span data-stu-id="61be0-276">This is the same as C and C++.</span></span>

### <a name="the-address-of-operator"></a><span data-ttu-id="61be0-277">Address 运算符</span><span class="sxs-lookup"><span data-stu-id="61be0-277">The address-of operator</span></span>

<span data-ttu-id="61be0-278">*Addressof_expression*由 "and" 符（`&`） *unary_expression*开头。</span><span class="sxs-lookup"><span data-stu-id="61be0-278">An *addressof_expression* consists of an ampersand (`&`) followed by a *unary_expression*.</span></span>

```antlr
addressof_expression
    : '&' unary_expression
    ;
```

<span data-ttu-id="61be0-279">给定 `T` 类型为的表达式 `E`，并将其归类为固定变量（固定变量[和可移动变量](unsafe-code.md#fixed-and-moveable-variables)），构造 `&E` 计算 `E`指定的变量的地址。</span><span class="sxs-lookup"><span data-stu-id="61be0-279">Given an expression `E` which is of a type `T` and is classified as a fixed variable ([Fixed and moveable variables](unsafe-code.md#fixed-and-moveable-variables)), the construct `&E` computes the address of the variable given by `E`.</span></span> <span data-ttu-id="61be0-280">结果的类型为 `T*`，并归类为值。</span><span class="sxs-lookup"><span data-stu-id="61be0-280">The type of the result is `T*` and is classified as a value.</span></span> <span data-ttu-id="61be0-281">如果 `E` 未分类为变量，则会发生编译时错误，如果 `E` 归类为只读局部变量，则会发生编译时错误; 如果 `E` 表示可移动的变量，则会发生编译时错误。</span><span class="sxs-lookup"><span data-stu-id="61be0-281">A compile-time error occurs if `E` is not classified as a variable, if `E` is classified as a read-only local variable, or if `E` denotes a moveable variable.</span></span> <span data-ttu-id="61be0-282">在最后一种情况下，可以使用 fixed 语句（[fixed 语句](unsafe-code.md#the-fixed-statement)）来暂时 "修复" 该变量，然后获取其地址。</span><span class="sxs-lookup"><span data-stu-id="61be0-282">In the last case, a fixed statement ([The fixed statement](unsafe-code.md#the-fixed-statement)) can be used to temporarily "fix" the variable before obtaining its address.</span></span> <span data-ttu-id="61be0-283">如[成员访问](expressions.md#member-access)中所述，在定义 `readonly` 字段的结构或类的实例构造函数或静态构造函数之外，该字段被视为值而非变量。</span><span class="sxs-lookup"><span data-stu-id="61be0-283">As stated in [Member access](expressions.md#member-access), outside an instance constructor or static constructor for a struct or class that defines a `readonly` field, that field is considered a value, not a variable.</span></span> <span data-ttu-id="61be0-284">因此无法采用其地址。</span><span class="sxs-lookup"><span data-stu-id="61be0-284">As such, its address cannot be taken.</span></span> <span data-ttu-id="61be0-285">同样，不能采用常量的地址。</span><span class="sxs-lookup"><span data-stu-id="61be0-285">Similarly, the address of a constant cannot be taken.</span></span>

<span data-ttu-id="61be0-286">`&` 运算符不需要将其自变量指定为明确赋值，但在执行 `&` 操作后，该运算符应用于的变量在执行操作的执行路径中被视为明确赋值。</span><span class="sxs-lookup"><span data-stu-id="61be0-286">The `&` operator does not require its argument to be definitely assigned, but following an `&` operation, the variable to which the operator is applied is considered definitely assigned in the execution path in which the operation occurs.</span></span> <span data-ttu-id="61be0-287">编程人员负责确保在这种情况下正确地初始化变量。</span><span class="sxs-lookup"><span data-stu-id="61be0-287">It is the responsibility of the programmer to ensure that correct initialization of the variable actually does take place in this situation.</span></span>

<span data-ttu-id="61be0-288">示例中</span><span class="sxs-lookup"><span data-stu-id="61be0-288">In the example</span></span>

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

<span data-ttu-id="61be0-289">`i` 在用于初始化 `p`的 `&i` 操作后被视为已明确赋值。</span><span class="sxs-lookup"><span data-stu-id="61be0-289">`i` is considered definitely assigned following the `&i` operation used to initialize `p`.</span></span> <span data-ttu-id="61be0-290">对 `*p` 的赋值实际上会初始化 `i`，但包含此初始化是程序员的责任，并且如果删除分配，则不会发生编译时错误。</span><span class="sxs-lookup"><span data-stu-id="61be0-290">The assignment to `*p` in effect initializes `i`, but the inclusion of this initialization is the responsibility of the programmer, and no compile-time error would occur if the assignment was removed.</span></span>

<span data-ttu-id="61be0-291">为 `&` 运算符提供明确赋值的规则，以便可以避免本地变量的冗余初始化。</span><span class="sxs-lookup"><span data-stu-id="61be0-291">The rules of definite assignment for the `&` operator exist such that redundant initialization of local variables can be avoided.</span></span> <span data-ttu-id="61be0-292">例如，许多外部 Api 采用一个指向结构的指针，该结构由 API 填充。</span><span class="sxs-lookup"><span data-stu-id="61be0-292">For example, many external APIs take a pointer to a structure which is filled in by the API.</span></span> <span data-ttu-id="61be0-293">调用此类 Api 通常会传递本地结构变量的地址，并且如果没有规则，则需要对结构变量进行冗余初始化。</span><span class="sxs-lookup"><span data-stu-id="61be0-293">Calls to such APIs typically pass the address of a local struct variable, and without the rule, redundant initialization of the struct variable would be required.</span></span>

### <a name="pointer-increment-and-decrement"></a><span data-ttu-id="61be0-294">指针增量和减量</span><span class="sxs-lookup"><span data-stu-id="61be0-294">Pointer increment and decrement</span></span>

<span data-ttu-id="61be0-295">在不安全的上下文中，`++` 和 `--` 运算符（[后缀递增和递减运算符](expressions.md#postfix-increment-and-decrement-operators)以及[前缀增量和减量运算符](expressions.md#prefix-increment-and-decrement-operators)）可应用于除 `void*`之外的所有类型的指针变量。</span><span class="sxs-lookup"><span data-stu-id="61be0-295">In an unsafe context, the `++` and `--` operators ([Postfix increment and decrement operators](expressions.md#postfix-increment-and-decrement-operators) and [Prefix increment and decrement operators](expressions.md#prefix-increment-and-decrement-operators)) can be applied to pointer variables of all types except `void*`.</span></span> <span data-ttu-id="61be0-296">因此，对于每个指针类型 `T*`，将隐式定义以下运算符：</span><span class="sxs-lookup"><span data-stu-id="61be0-296">Thus, for every pointer type `T*`, the following operators are implicitly defined:</span></span>

```csharp
T* operator ++(T* x);
T* operator --(T* x);
```

<span data-ttu-id="61be0-297">运算符分别与 `x + 1` 和 `x - 1`生成相同的结果（[指针算法](unsafe-code.md#pointer-arithmetic)）。</span><span class="sxs-lookup"><span data-stu-id="61be0-297">The operators produce the same results as `x + 1` and `x - 1`, respectively ([Pointer arithmetic](unsafe-code.md#pointer-arithmetic)).</span></span> <span data-ttu-id="61be0-298">换言之，对于类型 `T*`的指针变量，`++` 运算符将 `sizeof(T)` 添加到变量中包含的地址，并且 `--` 运算符将从该变量中包含的地址减去 `sizeof(T)`。</span><span class="sxs-lookup"><span data-stu-id="61be0-298">In other words, for a pointer variable of type `T*`, the `++` operator adds `sizeof(T)` to the address contained in the variable, and the `--` operator subtracts `sizeof(T)` from the address contained in the variable.</span></span>

<span data-ttu-id="61be0-299">如果指针增量或减量运算溢出了指针类型的域，则结果是实现定义的，但不会产生异常。</span><span class="sxs-lookup"><span data-stu-id="61be0-299">If a pointer increment or decrement operation overflows the domain of the pointer type, the result is implementation-defined, but no exceptions are produced.</span></span>

### <a name="pointer-arithmetic"></a><span data-ttu-id="61be0-300">指针算术</span><span class="sxs-lookup"><span data-stu-id="61be0-300">Pointer arithmetic</span></span>

<span data-ttu-id="61be0-301">在不安全的上下文中，`+` 和 `-` 运算符（[加法运算符](expressions.md#addition-operator)和[减法运算符](expressions.md#subtraction-operator)）可应用于除 `void*`之外的所有指针类型的值。</span><span class="sxs-lookup"><span data-stu-id="61be0-301">In an unsafe context, the `+` and `-` operators ([Addition operator](expressions.md#addition-operator) and [Subtraction operator](expressions.md#subtraction-operator)) can be applied to values of all pointer types except `void*`.</span></span> <span data-ttu-id="61be0-302">因此，对于每个指针类型 `T*`，将隐式定义以下运算符：</span><span class="sxs-lookup"><span data-stu-id="61be0-302">Thus, for every pointer type `T*`, the following operators are implicitly defined:</span></span>

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

<span data-ttu-id="61be0-303">给定表达式 `P` 指针类型 `T*` 和类型 `int`、`uint`、`long`或 `ulong`的表达式 `N`，表达式 `P + N` 和 `N + P` 会计算类型 `T*` 的指针值，结果是将 `N * sizeof(T)` 添加到 `P`给定的地址。</span><span class="sxs-lookup"><span data-stu-id="61be0-303">Given an expression `P` of a pointer type `T*` and an expression `N` of type `int`, `uint`, `long`, or `ulong`, the expressions `P + N` and `N + P` compute the pointer value of type `T*` that results from adding `N * sizeof(T)` to the address given by `P`.</span></span> <span data-ttu-id="61be0-304">同样，表达式 `P - N` 会计算类型 `T*` 的指针值，因为从 `P`给定的地址中减去 `N * sizeof(T)` 产生的结果。</span><span class="sxs-lookup"><span data-stu-id="61be0-304">Likewise, the expression `P - N` computes the pointer value of type `T*` that results from subtracting `N * sizeof(T)` from the address given by `P`.</span></span>

<span data-ttu-id="61be0-305">给定指针类型 `T*`的两个表达式（`P` 和 `Q`），表达式 `P - Q` 计算 `P` 和 `Q` 给定的地址之间的差异，然后将该差异除以 `sizeof(T)`。</span><span class="sxs-lookup"><span data-stu-id="61be0-305">Given two expressions, `P` and `Q`, of a pointer type `T*`, the expression `P - Q` computes the difference between the addresses given by `P` and `Q` and then divides that difference by `sizeof(T)`.</span></span> <span data-ttu-id="61be0-306">结果的类型始终 `long`。</span><span class="sxs-lookup"><span data-stu-id="61be0-306">The type of the result is always `long`.</span></span> <span data-ttu-id="61be0-307">实际上，`P - Q` 是 `((long)(P) - (long)(Q)) / sizeof(T)`计算得出的。</span><span class="sxs-lookup"><span data-stu-id="61be0-307">In effect, `P - Q` is computed as `((long)(P) - (long)(Q)) / sizeof(T)`.</span></span>

<span data-ttu-id="61be0-308">例如：</span><span class="sxs-lookup"><span data-stu-id="61be0-308">For example:</span></span>

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

<span data-ttu-id="61be0-309">生成输出的：</span><span class="sxs-lookup"><span data-stu-id="61be0-309">which produces the output:</span></span>

```console
p - q = -14
q - p = 14
```

<span data-ttu-id="61be0-310">如果指针算术运算溢出了指针类型的域，则会以实现定义的方式截断结果，但不会产生异常。</span><span class="sxs-lookup"><span data-stu-id="61be0-310">If a pointer arithmetic operation overflows the domain of the pointer type, the result is truncated in an implementation-defined fashion, but no exceptions are produced.</span></span>

### <a name="pointer-comparison"></a><span data-ttu-id="61be0-311">指针比较</span><span class="sxs-lookup"><span data-stu-id="61be0-311">Pointer comparison</span></span>

<span data-ttu-id="61be0-312">在不安全的上下文中，`==`、`!=`、`<`、`>`、`<=`和 `=>` 运算符（[关系和类型测试运算符](expressions.md#relational-and-type-testing-operators)）可应用于所有指针类型的值。</span><span class="sxs-lookup"><span data-stu-id="61be0-312">In an unsafe context, the `==`, `!=`, `<`, `>`, `<=`, and `=>` operators ([Relational and type-testing operators](expressions.md#relational-and-type-testing-operators)) can be applied to values of all pointer types.</span></span> <span data-ttu-id="61be0-313">指针比较运算符是：</span><span class="sxs-lookup"><span data-stu-id="61be0-313">The pointer comparison operators are:</span></span>

```csharp
bool operator ==(void* x, void* y);
bool operator !=(void* x, void* y);
bool operator <(void* x, void* y);
bool operator >(void* x, void* y);
bool operator <=(void* x, void* y);
bool operator >=(void* x, void* y);
```

<span data-ttu-id="61be0-314">由于存在从任何指针类型到 `void*` 类型的隐式转换，因此可以使用这些运算符比较任何指针类型的操作数。</span><span class="sxs-lookup"><span data-stu-id="61be0-314">Because an implicit conversion exists from any pointer type to the `void*` type, operands of any pointer type can be compared using these operators.</span></span> <span data-ttu-id="61be0-315">比较运算符比较两个操作数给定的地址，就像它们是无符号整数。</span><span class="sxs-lookup"><span data-stu-id="61be0-315">The comparison operators compare the addresses given by the two operands as if they were unsigned integers.</span></span>

### <a name="the-sizeof-operator"></a><span data-ttu-id="61be0-316">Sizeof 运算符</span><span class="sxs-lookup"><span data-stu-id="61be0-316">The sizeof operator</span></span>

<span data-ttu-id="61be0-317">`sizeof` 运算符返回给定类型的变量所占用的字节数。</span><span class="sxs-lookup"><span data-stu-id="61be0-317">The `sizeof` operator returns the number of bytes occupied by a variable of a given type.</span></span> <span data-ttu-id="61be0-318">指定为 `sizeof` 的操作数的类型必须是*unmanaged_type* （[指针类型](unsafe-code.md#pointer-types)）。</span><span class="sxs-lookup"><span data-stu-id="61be0-318">The type specified as an operand to `sizeof` must be an *unmanaged_type* ([Pointer types](unsafe-code.md#pointer-types)).</span></span>

```antlr
sizeof_expression
    : 'sizeof' '(' unmanaged_type ')'
    ;
```

<span data-ttu-id="61be0-319">`sizeof` 运算符的结果是 `int`类型的值。</span><span class="sxs-lookup"><span data-stu-id="61be0-319">The result of the `sizeof` operator is a value of type `int`.</span></span> <span data-ttu-id="61be0-320">对于某些预定义类型，`sizeof` 运算符产生一个常数值，如下表所示。</span><span class="sxs-lookup"><span data-stu-id="61be0-320">For certain predefined types, the `sizeof` operator yields a constant value as shown in the table below.</span></span>


| <span data-ttu-id="61be0-321">__表达式__</span><span class="sxs-lookup"><span data-stu-id="61be0-321">__Expression__</span></span>   | <span data-ttu-id="61be0-322">__结果__</span><span class="sxs-lookup"><span data-stu-id="61be0-322">__Result__</span></span> |
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

<span data-ttu-id="61be0-323">对于所有其他类型，`sizeof` 运算符的结果是实现定义的，并归类为值而不是常量。</span><span class="sxs-lookup"><span data-stu-id="61be0-323">For all other types, the result of the `sizeof` operator is implementation-defined and is classified as a value, not a constant.</span></span>

<span data-ttu-id="61be0-324">成员打包到结构中的顺序是未指定的。</span><span class="sxs-lookup"><span data-stu-id="61be0-324">The order in which members are packed into a struct is unspecified.</span></span>

<span data-ttu-id="61be0-325">出于对齐目的，在结构的开头、结构和结构的末尾，可能有未命名的填充。</span><span class="sxs-lookup"><span data-stu-id="61be0-325">For alignment purposes, there may be unnamed padding at the beginning of a struct, within a struct, and at the end of the struct.</span></span> <span data-ttu-id="61be0-326">用作填充的位数不确定。</span><span class="sxs-lookup"><span data-stu-id="61be0-326">The contents of the bits used as padding are indeterminate.</span></span>

<span data-ttu-id="61be0-327">当应用于具有结构类型的操作数时，结果为该类型的变量中的总字节数，包括任何空白。</span><span class="sxs-lookup"><span data-stu-id="61be0-327">When applied to an operand that has struct type, the result is the total number of bytes in a variable of that type, including any padding.</span></span>

## <a name="the-fixed-statement"></a><span data-ttu-id="61be0-328">Fixed 语句</span><span class="sxs-lookup"><span data-stu-id="61be0-328">The fixed statement</span></span>

<span data-ttu-id="61be0-329">在不安全的上下文中， *embedded_statement* （[语句](statements.md)）生产允许使用其他构造，即 `fixed` 语句，该语句用于 "修复" 可移动变量，使其地址在语句的持续时间内保持不变。</span><span class="sxs-lookup"><span data-stu-id="61be0-329">In an unsafe context, the *embedded_statement* ([Statements](statements.md)) production permits an additional construct, the `fixed` statement, which is used to "fix" a moveable variable such that its address remains constant for the duration of the statement.</span></span>

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

<span data-ttu-id="61be0-330">每个*fixed_pointer_declarator*都声明给定*pointer_type*的局部变量，并使用相应*fixed_pointer_initializer*计算出的地址初始化该局部变量。</span><span class="sxs-lookup"><span data-stu-id="61be0-330">Each *fixed_pointer_declarator* declares a local variable of the given *pointer_type* and initializes that local variable with the address computed by the corresponding *fixed_pointer_initializer*.</span></span> <span data-ttu-id="61be0-331">在 `fixed` 语句中声明的局部变量可在该变量的声明右侧和 `fixed` 语句的*embedded_statement*中的任何*fixed_pointer_initializer*进行访问。</span><span class="sxs-lookup"><span data-stu-id="61be0-331">A local variable declared in a `fixed` statement is accessible in any *fixed_pointer_initializer*s occurring to the right of that variable's declaration, and in the *embedded_statement* of the `fixed` statement.</span></span> <span data-ttu-id="61be0-332">`fixed` 语句声明的局部变量被视为只读。</span><span class="sxs-lookup"><span data-stu-id="61be0-332">A local variable declared by a `fixed` statement is considered read-only.</span></span> <span data-ttu-id="61be0-333">如果嵌入的语句尝试修改此局部变量（通过赋值或 `++` 和 `--` 运算符），或将其作为 `ref` 或 `out` 参数传递，则会发生编译时错误。</span><span class="sxs-lookup"><span data-stu-id="61be0-333">A compile-time error occurs if the embedded statement attempts to modify this local variable (via assignment or the `++` and `--` operators) or pass it as a `ref` or `out` parameter.</span></span>

<span data-ttu-id="61be0-334">*Fixed_pointer_initializer*可以是以下项之一：</span><span class="sxs-lookup"><span data-stu-id="61be0-334">A *fixed_pointer_initializer* can be one of the following:</span></span>

*  <span data-ttu-id="61be0-335">标记 "`&`" 后跟*variable_reference* （[确定明确赋值的精确规则](variables.md#precise-rules-for-determining-definite-assignment)）到非托管类型 `T`的可移动变量（[固定和可移动变量](unsafe-code.md#fixed-and-moveable-variables)），前提是该类型 `T*` 可隐式转换为 `fixed` 语句中给定的指针类型。</span><span class="sxs-lookup"><span data-stu-id="61be0-335">The token "`&`" followed by a *variable_reference* ([Precise rules for determining definite assignment](variables.md#precise-rules-for-determining-definite-assignment)) to a moveable variable ([Fixed and moveable variables](unsafe-code.md#fixed-and-moveable-variables)) of an unmanaged type `T`, provided the type `T*` is implicitly convertible to the pointer type given in the `fixed` statement.</span></span> <span data-ttu-id="61be0-336">在这种情况下，初始值设定项将计算给定变量的地址，并且在 `fixed` 语句的持续时间内，该变量可保证保持为固定地址。</span><span class="sxs-lookup"><span data-stu-id="61be0-336">In this case, the initializer computes the address of the given variable, and the variable is guaranteed to remain at a fixed address for the duration of the `fixed` statement.</span></span>
*  <span data-ttu-id="61be0-337">带有非托管类型 `T`元素的*array_type*的表达式，前提是该类型 `T*` 可隐式转换为 `fixed` 语句中给定的指针类型。</span><span class="sxs-lookup"><span data-stu-id="61be0-337">An expression of an *array_type* with elements of an unmanaged type `T`, provided the type `T*` is implicitly convertible to the pointer type given in the `fixed` statement.</span></span> <span data-ttu-id="61be0-338">在这种情况下，初始值设定项计算数组中第一个元素的地址，并且在 `fixed` 语句的持续时间内保证整个数组保持为固定的地址。</span><span class="sxs-lookup"><span data-stu-id="61be0-338">In this case, the initializer computes the address of the first element in the array, and the entire array is guaranteed to remain at a fixed address for the duration of the `fixed` statement.</span></span> <span data-ttu-id="61be0-339">如果数组表达式为 null 或数组包含零个元素，则初始值设定项将计算等于零的地址。</span><span class="sxs-lookup"><span data-stu-id="61be0-339">If the array expression is null or if the array has zero elements, the initializer computes an address equal to zero.</span></span>
*  <span data-ttu-id="61be0-340">如果类型 `char*` 可隐式转换为 `fixed` 语句中给定的指针类型，则为 `string`类型的表达式。</span><span class="sxs-lookup"><span data-stu-id="61be0-340">An expression of type `string`, provided the type `char*` is implicitly convertible to the pointer type given in the `fixed` statement.</span></span> <span data-ttu-id="61be0-341">在这种情况下，初始值设定项将计算字符串中第一个字符的地址，并且在 `fixed` 语句的持续时间内保证整个字符串保持为固定地址。</span><span class="sxs-lookup"><span data-stu-id="61be0-341">In this case, the initializer computes the address of the first character in the string, and the entire string is guaranteed to remain at a fixed address for the duration of the `fixed` statement.</span></span> <span data-ttu-id="61be0-342">如果字符串表达式为 null，则 `fixed` 语句的行为是实现定义的。</span><span class="sxs-lookup"><span data-stu-id="61be0-342">The behavior of the `fixed` statement is implementation-defined if the string expression is null.</span></span>
*  <span data-ttu-id="61be0-343">如果固定大小缓冲区成员的类型可隐式转换为在 `fixed` 语句中给定的指针类型，则引用可移动变量的固定大小缓冲区成员的*simple_name*或*member_access* 。</span><span class="sxs-lookup"><span data-stu-id="61be0-343">A *simple_name* or *member_access* that references a fixed size buffer member of a moveable variable, provided the type of the fixed size buffer member is implicitly convertible to the pointer type given in the `fixed` statement.</span></span> <span data-ttu-id="61be0-344">在这种情况下，初始值设定项将计算一个指针，该指针指向固定大小缓冲区的第一个元素（[表达式中的固定大小缓冲区](unsafe-code.md#fixed-size-buffers-in-expressions)），并且在 `fixed` 语句的持续时间内保证固定大小缓冲区保持为固定地址。</span><span class="sxs-lookup"><span data-stu-id="61be0-344">In this case, the initializer computes a pointer to the first element of the fixed size buffer ([Fixed size buffers in expressions](unsafe-code.md#fixed-size-buffers-in-expressions)), and the fixed size buffer is guaranteed to remain at a fixed address for the duration of the `fixed` statement.</span></span>

<span data-ttu-id="61be0-345">对于由*fixed_pointer_initializer*计算得出的每个地址，`fixed` 语句可确保在 `fixed` 语句的持续时间内，垃圾回收器不会将该地址引用的变量重定位或释放。</span><span class="sxs-lookup"><span data-stu-id="61be0-345">For each address computed by a *fixed_pointer_initializer* the `fixed` statement ensures that the variable referenced by the address is not subject to relocation or disposal by the garbage collector for the duration of the `fixed` statement.</span></span> <span data-ttu-id="61be0-346">例如，如果由*fixed_pointer_initializer*计算的地址引用一个对象的字段或数组实例的一个元素，则 `fixed` 语句保证在该语句的生存期内不会重新定位或释放包含对象实例。</span><span class="sxs-lookup"><span data-stu-id="61be0-346">For example, if the address computed by a *fixed_pointer_initializer* references a field of an object or an element of an array instance, the `fixed` statement guarantees that the containing object instance is not relocated or disposed of during the lifetime of the statement.</span></span>

<span data-ttu-id="61be0-347">程序员应负责确保 `fixed` 语句创建的指针不会在执行这些语句的范围之外。</span><span class="sxs-lookup"><span data-stu-id="61be0-347">It is the programmer's responsibility to ensure that pointers created by `fixed` statements do not survive beyond execution of those statements.</span></span> <span data-ttu-id="61be0-348">例如，当 `fixed` 语句创建的指针传递到外部 Api 时，程序员应负责确保 Api 不保留这些指针的内存。</span><span class="sxs-lookup"><span data-stu-id="61be0-348">For example, when pointers created by `fixed` statements are passed to external APIs, it is the programmer's responsibility to ensure that the APIs retain no memory of these pointers.</span></span>

<span data-ttu-id="61be0-349">固定对象可能会导致堆碎片（因为它们无法移动）。</span><span class="sxs-lookup"><span data-stu-id="61be0-349">Fixed objects may cause fragmentation of the heap (because they can't be moved).</span></span> <span data-ttu-id="61be0-350">因此，仅当绝对必要时才应修复对象，而只应在尽可能最短的时间进行修复。</span><span class="sxs-lookup"><span data-stu-id="61be0-350">For that reason, objects should be fixed only when absolutely necessary and then only for the shortest amount of time possible.</span></span>

<span data-ttu-id="61be0-351">示例</span><span class="sxs-lookup"><span data-stu-id="61be0-351">The example</span></span>

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

<span data-ttu-id="61be0-352">演示 `fixed` 语句的几次使用。</span><span class="sxs-lookup"><span data-stu-id="61be0-352">demonstrates several uses of the `fixed` statement.</span></span> <span data-ttu-id="61be0-353">第一条语句修复并获取静态字段的地址，第二条语句修复并获取实例字段的地址，第三条语句修复并获取数组元素的地址。</span><span class="sxs-lookup"><span data-stu-id="61be0-353">The first statement fixes and obtains the address of a static field, the second statement fixes and obtains the address of an instance field, and the third statement fixes and obtains the address of an array element.</span></span> <span data-ttu-id="61be0-354">在每种情况下，使用 regular `&` 运算符都是错误的，因为变量全都归类为可移动变量。</span><span class="sxs-lookup"><span data-stu-id="61be0-354">In each case it would have been an error to use the regular `&` operator since the variables are all classified as moveable variables.</span></span>

<span data-ttu-id="61be0-355">上面示例中的第四个 `fixed` 语句将产生类似于第三个的结果。</span><span class="sxs-lookup"><span data-stu-id="61be0-355">The fourth `fixed` statement in the example above produces a similar result to the third.</span></span>

<span data-ttu-id="61be0-356">此 `fixed` 语句的示例使用 `string`：</span><span class="sxs-lookup"><span data-stu-id="61be0-356">This example of the `fixed` statement uses `string`:</span></span>

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

<span data-ttu-id="61be0-357">在不安全的上下文数组元素中，一维数组的元素按索引顺序存储，从索引 `0` 开始，以 index `Length - 1`结束。</span><span class="sxs-lookup"><span data-stu-id="61be0-357">In an unsafe context array elements of single-dimensional arrays are stored in increasing index order, starting with index `0` and ending with index `Length - 1`.</span></span> <span data-ttu-id="61be0-358">对于多维数组，将存储数组元素，这样最右边的维度的索引将首先增加，然后是下一个左侧维度，依此类推。</span><span class="sxs-lookup"><span data-stu-id="61be0-358">For multi-dimensional arrays, array elements are stored such that the indices of the rightmost dimension are increased first, then the next left dimension, and so on to the left.</span></span> <span data-ttu-id="61be0-359">在 `fixed` 语句中获取指向数组实例 `a`的指针 `p`，指针值范围从 `p` 到 `p + a.Length - 1` 表示数组中元素的地址。</span><span class="sxs-lookup"><span data-stu-id="61be0-359">Within a `fixed` statement that obtains a pointer `p` to an array instance `a`, the pointer values ranging from `p` to `p + a.Length - 1` represent addresses of the elements in the array.</span></span> <span data-ttu-id="61be0-360">同样，从 `p[0]` 到 `p[a.Length - 1]` 的变量表示实际数组元素。</span><span class="sxs-lookup"><span data-stu-id="61be0-360">Likewise, the variables ranging from `p[0]` to `p[a.Length - 1]` represent the actual array elements.</span></span> <span data-ttu-id="61be0-361">考虑到数组的存储方式，可以将任何维度的数组视为线性数组。</span><span class="sxs-lookup"><span data-stu-id="61be0-361">Given the way in which arrays are stored, we can treat an array of any dimension as though it were linear.</span></span>

<span data-ttu-id="61be0-362">例如：</span><span class="sxs-lookup"><span data-stu-id="61be0-362">For example:</span></span>

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

<span data-ttu-id="61be0-363">生成输出的：</span><span class="sxs-lookup"><span data-stu-id="61be0-363">which produces the output:</span></span>

```console
[0,0,0] =  0 [0,0,1] =  1 [0,0,2] =  2 [0,0,3] =  3
[0,1,0] =  4 [0,1,1] =  5 [0,1,2] =  6 [0,1,3] =  7
[0,2,0] =  8 [0,2,1] =  9 [0,2,2] = 10 [0,2,3] = 11
[1,0,0] = 12 [1,0,1] = 13 [1,0,2] = 14 [1,0,3] = 15
[1,1,0] = 16 [1,1,1] = 17 [1,1,2] = 18 [1,1,3] = 19
[1,2,0] = 20 [1,2,1] = 21 [1,2,2] = 22 [1,2,3] = 23
```

<span data-ttu-id="61be0-364">示例中</span><span class="sxs-lookup"><span data-stu-id="61be0-364">In the example</span></span>

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

<span data-ttu-id="61be0-365">使用 `fixed` 语句修复数组，以便可以将其地址传递给采用指针的方法。</span><span class="sxs-lookup"><span data-stu-id="61be0-365">a `fixed` statement is used to fix an array so its address can be passed to a method that takes a pointer.</span></span>

<span data-ttu-id="61be0-366">在下面的示例中：</span><span class="sxs-lookup"><span data-stu-id="61be0-366">In the example:</span></span>

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

<span data-ttu-id="61be0-367">fixed 语句用于修复结构的固定大小缓冲区，使其地址可用作指针。</span><span class="sxs-lookup"><span data-stu-id="61be0-367">a fixed statement is used to fix a fixed size buffer of a struct so its address can be used as a pointer.</span></span>

<span data-ttu-id="61be0-368">通过修复字符串实例产生 `char*` 值始终指向以 null 结尾的字符串。</span><span class="sxs-lookup"><span data-stu-id="61be0-368">A `char*` value produced by fixing a string instance always points to a null-terminated string.</span></span> <span data-ttu-id="61be0-369">在 `s``p` 获取指向字符串实例的指针的固定语句中，指针值从 `p` 到 `p + s.Length - 1` 表示字符串中的字符的地址，指针值 `p + s.Length` 始终指向 null 字符（值为 `'\0'`的字符）。</span><span class="sxs-lookup"><span data-stu-id="61be0-369">Within a fixed statement that obtains a pointer `p` to a string instance `s`, the pointer values ranging from `p` to `p + s.Length - 1` represent addresses of the characters in the string, and the pointer value `p + s.Length` always points to a null character (the character with value `'\0'`).</span></span>

<span data-ttu-id="61be0-370">通过固定指针修改托管类型的对象可能导致未定义的行为。</span><span class="sxs-lookup"><span data-stu-id="61be0-370">Modifying objects of managed type through fixed pointers can results in undefined behavior.</span></span> <span data-ttu-id="61be0-371">例如，因为字符串是不可变的，所以，程序员应负责确保不会修改指向固定字符串的指针所引用的字符。</span><span class="sxs-lookup"><span data-stu-id="61be0-371">For example, because strings are immutable, it is the programmer's responsibility to ensure that the characters referenced by a pointer to a fixed string are not modified.</span></span>

<span data-ttu-id="61be0-372">在调用需要 "C 样式" 字符串的外部 Api 时，字符串的自动 null 终止非常方便。</span><span class="sxs-lookup"><span data-stu-id="61be0-372">The automatic null-termination of strings is particularly convenient when calling external APIs that expect "C-style" strings.</span></span> <span data-ttu-id="61be0-373">但请注意，字符串实例允许包含 null 字符。</span><span class="sxs-lookup"><span data-stu-id="61be0-373">Note, however, that a string instance is permitted to contain null characters.</span></span> <span data-ttu-id="61be0-374">如果存在此类 null 字符，则该字符串将在被视为以 null 结尾的 `char*`时出现截断。</span><span class="sxs-lookup"><span data-stu-id="61be0-374">If such null characters are present, the string will appear truncated when treated as a null-terminated `char*`.</span></span>

## <a name="fixed-size-buffers"></a><span data-ttu-id="61be0-375">固定大小的缓冲区</span><span class="sxs-lookup"><span data-stu-id="61be0-375">Fixed size buffers</span></span>

<span data-ttu-id="61be0-376">固定大小的缓冲区用于将 "C 样式" 内联数组声明为结构的成员，主要用于与非托管 Api 建立交互。</span><span class="sxs-lookup"><span data-stu-id="61be0-376">Fixed size buffers are used to declare "C style" in-line arrays as members of structs, and are primarily useful for interfacing with unmanaged APIs.</span></span>

### <a name="fixed-size-buffer-declarations"></a><span data-ttu-id="61be0-377">固定大小的缓冲区声明</span><span class="sxs-lookup"><span data-stu-id="61be0-377">Fixed size buffer declarations</span></span>

<span data-ttu-id="61be0-378">***固定大小缓冲区***是表示给定类型的变量的固定长度缓冲区存储的成员。</span><span class="sxs-lookup"><span data-stu-id="61be0-378">A ***fixed size buffer*** is a member that represents storage for a fixed length buffer of variables of a given type.</span></span> <span data-ttu-id="61be0-379">固定大小的缓冲区声明引入给定元素类型的一个或多个固定大小的缓冲区。</span><span class="sxs-lookup"><span data-stu-id="61be0-379">A fixed size buffer declaration introduces one or more fixed size buffers of a given element type.</span></span> <span data-ttu-id="61be0-380">固定大小的缓冲区只允许在结构声明中使用，并且只能在不安全的上下文中出现（[不安全](unsafe-code.md#unsafe-contexts)上下文）。</span><span class="sxs-lookup"><span data-stu-id="61be0-380">Fixed size buffers are only permitted in struct declarations and can only occur in unsafe contexts ([Unsafe contexts](unsafe-code.md#unsafe-contexts)).</span></span>

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

<span data-ttu-id="61be0-381">固定大小的缓冲区声明可以包含一组属性（[特性](attributes.md)）、一个 `new` 修饰符（[修饰符](classes.md#modifiers)）、四个访问修饰符（[类型参数和约束](classes.md#type-parameters-and-constraints)）和 `unsafe` 修饰符（[不安全上下文](unsafe-code.md#unsafe-contexts)）的有效组合。</span><span class="sxs-lookup"><span data-stu-id="61be0-381">A fixed size buffer declaration may include a set of attributes ([Attributes](attributes.md)), a `new` modifier ([Modifiers](classes.md#modifiers)), a valid combination of the four access modifiers ([Type parameters and constraints](classes.md#type-parameters-and-constraints)) and an `unsafe` modifier ([Unsafe contexts](unsafe-code.md#unsafe-contexts)).</span></span> <span data-ttu-id="61be0-382">特性和修饰符适用于由固定大小缓冲区声明声明的所有成员。</span><span class="sxs-lookup"><span data-stu-id="61be0-382">The attributes and modifiers apply to all of the members declared by the fixed size buffer declaration.</span></span> <span data-ttu-id="61be0-383">同一修饰符在固定大小的缓冲区声明中多次出现是错误的。</span><span class="sxs-lookup"><span data-stu-id="61be0-383">It is an error for the same modifier to appear multiple times in a fixed size buffer declaration.</span></span>

<span data-ttu-id="61be0-384">固定大小的缓冲区声明不允许包含 `static` 修饰符。</span><span class="sxs-lookup"><span data-stu-id="61be0-384">A fixed size buffer declaration is not permitted to include the `static` modifier.</span></span>

<span data-ttu-id="61be0-385">固定大小缓冲区声明的 buffer 元素类型指定声明引入的缓冲区的元素类型。</span><span class="sxs-lookup"><span data-stu-id="61be0-385">The buffer element type of a fixed size buffer declaration specifies the element type of the buffer(s) introduced by the declaration.</span></span> <span data-ttu-id="61be0-386">Buffer 元素类型必须是预定义的类型之一 `sbyte`、`byte`、`short`、`ushort`、`int`、`uint`、`long`、`ulong`、`char`、`float`、`double`或 `bool`。</span><span class="sxs-lookup"><span data-stu-id="61be0-386">The buffer element type must be one of the predefined types `sbyte`, `byte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`, `char`, `float`, `double`, or `bool`.</span></span>

<span data-ttu-id="61be0-387">Buffer 元素类型后跟固定大小缓冲区声明符的列表，其中每个声明符都引入一个新成员。</span><span class="sxs-lookup"><span data-stu-id="61be0-387">The buffer element type is followed by a list of fixed size buffer declarators, each of which introduces a new member.</span></span> <span data-ttu-id="61be0-388">固定大小的缓冲区声明符包含一个命名成员的标识符，后跟一个括在 `[` 和 `]` 令牌中的常量表达式。</span><span class="sxs-lookup"><span data-stu-id="61be0-388">A fixed size buffer declarator consists of an identifier that names the member, followed by a constant expression enclosed in `[` and `]` tokens.</span></span> <span data-ttu-id="61be0-389">常数表达式表示该固定大小缓冲区声明符引入的成员中的元素数目。</span><span class="sxs-lookup"><span data-stu-id="61be0-389">The constant expression denotes the number of elements in the member introduced by that fixed size buffer declarator.</span></span> <span data-ttu-id="61be0-390">常数表达式的类型必须可隐式转换为类型 `int`，且值必须为非零正整数。</span><span class="sxs-lookup"><span data-stu-id="61be0-390">The type of the constant expression must be implicitly convertible to type `int`, and the value must be a non-zero positive integer.</span></span>

<span data-ttu-id="61be0-391">保证固定大小缓冲区的元素在内存中按顺序排列。</span><span class="sxs-lookup"><span data-stu-id="61be0-391">The elements of a fixed size buffer are guaranteed to be laid out sequentially in memory.</span></span>

<span data-ttu-id="61be0-392">声明多个固定大小缓冲区的固定大小缓冲区声明等效于具有相同属性和元素类型的单个固定大小缓冲区声明的多个声明。</span><span class="sxs-lookup"><span data-stu-id="61be0-392">A fixed size buffer declaration that declares multiple fixed size buffers is equivalent to multiple declarations of a single fixed size buffer declaration with the same attributes, and element types.</span></span> <span data-ttu-id="61be0-393">例如</span><span class="sxs-lookup"><span data-stu-id="61be0-393">For example</span></span>

```csharp
unsafe struct A
{
   public fixed int x[5], y[10], z[100];
}
```

<span data-ttu-id="61be0-394">等效于</span><span class="sxs-lookup"><span data-stu-id="61be0-394">is equivalent to</span></span>

```csharp
unsafe struct A
{
   public fixed int x[5];
   public fixed int y[10];
   public fixed int z[100];
}
```

### <a name="fixed-size-buffers-in-expressions"></a><span data-ttu-id="61be0-395">表达式中固定大小的缓冲区</span><span class="sxs-lookup"><span data-stu-id="61be0-395">Fixed size buffers in expressions</span></span>

<span data-ttu-id="61be0-396">固定大小缓冲区成员的成员查找（[运算符](expressions.md#operators)）与字段的成员查找完全相同。</span><span class="sxs-lookup"><span data-stu-id="61be0-396">Member lookup ([Operators](expressions.md#operators)) of a fixed size buffer member proceeds exactly like member lookup of a field.</span></span>

<span data-ttu-id="61be0-397">使用*simple_name* （[类型推理](expressions.md#type-inference)）或*member_access* （对[动态重载决策进行编译时检查](expressions.md#compile-time-checking-of-dynamic-overload-resolution)），可以在表达式中引用固定大小的缓冲区。</span><span class="sxs-lookup"><span data-stu-id="61be0-397">A fixed size buffer can be referenced in an expression using a *simple_name* ([Type inference](expressions.md#type-inference)) or a *member_access* ([Compile-time checking of dynamic overload resolution](expressions.md#compile-time-checking-of-dynamic-overload-resolution)).</span></span>

<span data-ttu-id="61be0-398">当将固定大小的缓冲区成员作为简单名称引用时，其效果与窗体 `this.I`的成员访问相同，其中 `I` 为固定大小缓冲区成员。</span><span class="sxs-lookup"><span data-stu-id="61be0-398">When a fixed size buffer member is referenced as a simple name, the effect is the same as a member access of the form `this.I`, where `I` is the fixed size buffer member.</span></span>

<span data-ttu-id="61be0-399">在窗体 `E.I`的成员访问中，如果 `E` 为结构类型，并且该结构类型中的 `I` 的成员查找标识固定大小成员，则按如下方式计算 `E.I`：</span><span class="sxs-lookup"><span data-stu-id="61be0-399">In a member access of the form `E.I`, if `E` is of a struct type and a member lookup of `I` in that struct type identifies a fixed size member, then `E.I` is evaluated an classified as follows:</span></span>

*  <span data-ttu-id="61be0-400">如果表达式 `E.I` 未出现在不安全的上下文中，则会发生编译时错误。</span><span class="sxs-lookup"><span data-stu-id="61be0-400">If the expression `E.I` does not occur in an unsafe context, a compile-time error occurs.</span></span>
*  <span data-ttu-id="61be0-401">如果 `E` 归类为值，则会发生编译时错误。</span><span class="sxs-lookup"><span data-stu-id="61be0-401">If `E` is classified as a value, a compile-time error occurs.</span></span>
*  <span data-ttu-id="61be0-402">否则，如果 `E` 是可移动变量（[固定和可移动变量](unsafe-code.md#fixed-and-moveable-variables)），并且表达式 `E.I` 不是*fixed_pointer_initializer* （[Fixed 语句](unsafe-code.md#the-fixed-statement)），则会发生编译时错误。</span><span class="sxs-lookup"><span data-stu-id="61be0-402">Otherwise, if `E` is a moveable variable ([Fixed and moveable variables](unsafe-code.md#fixed-and-moveable-variables)) and the expression `E.I` is not a *fixed_pointer_initializer* ([The fixed statement](unsafe-code.md#the-fixed-statement)), a compile-time error occurs.</span></span>
*  <span data-ttu-id="61be0-403">否则，`E` 引用了一个固定变量，并且该表达式的结果是一个指向 `E`中固定大小缓冲区成员 `I` 的第一个元素的指针。</span><span class="sxs-lookup"><span data-stu-id="61be0-403">Otherwise, `E` references a fixed variable and the result of the expression is a pointer to the first element of the fixed size buffer member `I` in `E`.</span></span> <span data-ttu-id="61be0-404">结果的类型为 `S*`，其中 `S` 是 `I`的元素类型，并归类为值。</span><span class="sxs-lookup"><span data-stu-id="61be0-404">The result is of type `S*`, where `S` is the element type of `I`, and is classified as a value.</span></span>

<span data-ttu-id="61be0-405">可以使用第一个元素的指针操作访问固定大小缓冲区的后续元素。</span><span class="sxs-lookup"><span data-stu-id="61be0-405">The subsequent elements of the fixed size buffer can be accessed using pointer operations from the first element.</span></span> <span data-ttu-id="61be0-406">与对数组的访问不同，对固定大小缓冲区的元素的访问是不安全的操作，并且不会进行范围检查。</span><span class="sxs-lookup"><span data-stu-id="61be0-406">Unlike access to arrays, access to the elements of a fixed size buffer is an unsafe operation and is not range checked.</span></span>

<span data-ttu-id="61be0-407">下面的示例声明并使用具有固定大小缓冲区成员的结构。</span><span class="sxs-lookup"><span data-stu-id="61be0-407">The following example declares and uses a struct with a fixed size buffer member.</span></span>

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

### <a name="definite-assignment-checking"></a><span data-ttu-id="61be0-408">明确赋值检查</span><span class="sxs-lookup"><span data-stu-id="61be0-408">Definite assignment checking</span></span>

<span data-ttu-id="61be0-409">固定大小的缓冲区不受明确的赋值检查（[明确赋值](variables.md#definite-assignment)）的限制，并且忽略固定大小的缓冲区成员，以便对结构类型变量进行明确的赋值检查。</span><span class="sxs-lookup"><span data-stu-id="61be0-409">Fixed size buffers are not subject to definite assignment checking ([Definite assignment](variables.md#definite-assignment)), and fixed size buffer members are ignored for purposes of definite assignment checking of struct type variables.</span></span>

<span data-ttu-id="61be0-410">当固定大小缓冲区成员的最外面的包含结构变量是静态变量、类实例的实例变量或数组元素时，固定大小缓冲区的元素会自动初始化为其默认值（[默认值](variables.md#default-values)）。</span><span class="sxs-lookup"><span data-stu-id="61be0-410">When the outermost containing struct variable of a fixed size buffer member is a static variable, an instance variable of a class instance, or an array element, the elements of the fixed size buffer are automatically initialized to their default values ([Default values](variables.md#default-values)).</span></span> <span data-ttu-id="61be0-411">在所有其他情况下，固定大小缓冲区的初始内容是不确定的。</span><span class="sxs-lookup"><span data-stu-id="61be0-411">In all other cases, the initial content of a fixed size buffer is undefined.</span></span>

## <a name="stack-allocation"></a><span data-ttu-id="61be0-412">堆栈分配</span><span class="sxs-lookup"><span data-stu-id="61be0-412">Stack allocation</span></span>

<span data-ttu-id="61be0-413">在不安全的上下文中，局部变量声明（[局部变量声明](statements.md#local-variable-declarations)）可能包括从调用堆栈中分配内存的堆栈分配初始值设定项。</span><span class="sxs-lookup"><span data-stu-id="61be0-413">In an unsafe context, a local variable declaration ([Local variable declarations](statements.md#local-variable-declarations)) may include a stack allocation initializer which allocates memory from the call stack.</span></span>

```antlr
local_variable_initializer_unsafe
    : stackalloc_initializer
    ;

stackalloc_initializer
    : 'stackalloc' unmanaged_type '[' expression ']'
    ;
```

<span data-ttu-id="61be0-414">*Unmanaged_type*指示将存储在新分配的位置的项的类型，并且*表达式*指示这些项的数目。</span><span class="sxs-lookup"><span data-stu-id="61be0-414">The *unmanaged_type* indicates the type of the items that will be stored in the newly allocated location, and the *expression* indicates the number of these items.</span></span> <span data-ttu-id="61be0-415">它们一起指定了所需的分配大小。</span><span class="sxs-lookup"><span data-stu-id="61be0-415">Taken together, these specify the required allocation size.</span></span> <span data-ttu-id="61be0-416">由于堆栈分配的大小不能为负，因此将项的数目指定为计算结果为负值的*constant_expression*是编译时错误。</span><span class="sxs-lookup"><span data-stu-id="61be0-416">Since the size of a stack allocation cannot be negative, it is a compile-time error to specify the number of items as a *constant_expression* that evaluates to a negative value.</span></span>

<span data-ttu-id="61be0-417">格式 `stackalloc T[E]` 的堆栈分配初始值设定项需要 `T` 为非托管类型（[指针类型](unsafe-code.md#pointer-types)），并且 `E` 是 `int`类型的表达式。</span><span class="sxs-lookup"><span data-stu-id="61be0-417">A stack allocation initializer of the form `stackalloc T[E]` requires `T` to be an unmanaged type ([Pointer types](unsafe-code.md#pointer-types)) and `E` to be an expression of type `int`.</span></span> <span data-ttu-id="61be0-418">构造从调用堆栈分配 `E * sizeof(T)` 字节，并将 `T*`类型的指针返回到新分配的块。</span><span class="sxs-lookup"><span data-stu-id="61be0-418">The construct allocates `E * sizeof(T)` bytes from the call stack and returns a pointer, of type `T*`, to the newly allocated block.</span></span> <span data-ttu-id="61be0-419">如果 `E` 为负值，则行为是不确定的。</span><span class="sxs-lookup"><span data-stu-id="61be0-419">If `E` is a negative value, then the behavior is undefined.</span></span> <span data-ttu-id="61be0-420">如果 `E` 为零，则不进行任何分配，并且返回的指针是实现定义的。</span><span class="sxs-lookup"><span data-stu-id="61be0-420">If `E` is zero, then no allocation is made, and the pointer returned is implementation-defined.</span></span> <span data-ttu-id="61be0-421">如果内存不足，无法分配指定大小的块，则会引发 `System.StackOverflowException`。</span><span class="sxs-lookup"><span data-stu-id="61be0-421">If there is not enough memory available to allocate a block of the given size, a `System.StackOverflowException` is thrown.</span></span>

<span data-ttu-id="61be0-422">新分配的内存的内容未定义。</span><span class="sxs-lookup"><span data-stu-id="61be0-422">The content of the newly allocated memory is undefined.</span></span>

<span data-ttu-id="61be0-423">`catch` 或 `finally` 块（[try 语句](statements.md#the-try-statement)）中不允许使用堆栈分配初始值设定项。</span><span class="sxs-lookup"><span data-stu-id="61be0-423">Stack allocation initializers are not permitted in `catch` or `finally` blocks ([The try statement](statements.md#the-try-statement)).</span></span>

<span data-ttu-id="61be0-424">无法显式释放使用 `stackalloc`分配的内存。</span><span class="sxs-lookup"><span data-stu-id="61be0-424">There is no way to explicitly free memory allocated using `stackalloc`.</span></span> <span data-ttu-id="61be0-425">在函数成员执行过程中创建的所有堆栈分配的内存块将在该函数成员返回时自动被丢弃。</span><span class="sxs-lookup"><span data-stu-id="61be0-425">All stack allocated memory blocks created during the execution of a function member are automatically discarded when that function member returns.</span></span> <span data-ttu-id="61be0-426">这对应于 `alloca` 函数，这是一个在 C 和C++实现中常见的扩展。</span><span class="sxs-lookup"><span data-stu-id="61be0-426">This corresponds to the `alloca` function, an extension commonly found in C and C++ implementations.</span></span>

<span data-ttu-id="61be0-427">示例中</span><span class="sxs-lookup"><span data-stu-id="61be0-427">In the example</span></span>

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

<span data-ttu-id="61be0-428">`stackalloc` 初始值设定项用在 `IntToString` 方法中，用于在堆栈上分配16个字符的缓冲区。</span><span class="sxs-lookup"><span data-stu-id="61be0-428">a `stackalloc` initializer is used in the `IntToString` method to allocate a buffer of 16 characters on the stack.</span></span> <span data-ttu-id="61be0-429">当该方法返回时，缓冲区会自动丢弃。</span><span class="sxs-lookup"><span data-stu-id="61be0-429">The buffer is automatically discarded when the method returns.</span></span>

## <a name="dynamic-memory-allocation"></a><span data-ttu-id="61be0-430">动态内存分配</span><span class="sxs-lookup"><span data-stu-id="61be0-430">Dynamic memory allocation</span></span>

<span data-ttu-id="61be0-431">除 `stackalloc` 运算符外， C#不提供用于管理非垃圾回收内存的预定义构造。</span><span class="sxs-lookup"><span data-stu-id="61be0-431">Except for the `stackalloc` operator, C# provides no predefined constructs for managing non-garbage collected memory.</span></span> <span data-ttu-id="61be0-432">此类服务通常通过支持类库来提供，也可以从基础操作系统直接导入。</span><span class="sxs-lookup"><span data-stu-id="61be0-432">Such services are typically provided by supporting class libraries or imported directly from the underlying operating system.</span></span> <span data-ttu-id="61be0-433">例如，下面的 `Memory` 类演示了如何从C#访问基础操作系统的堆函数：</span><span class="sxs-lookup"><span data-stu-id="61be0-433">For example, the `Memory` class below illustrates how the heap functions of an underlying operating system might be accessed from C#:</span></span>

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

<span data-ttu-id="61be0-434">下面给出了使用 `Memory` 类的示例：</span><span class="sxs-lookup"><span data-stu-id="61be0-434">An example that uses the `Memory` class is given below:</span></span>

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

<span data-ttu-id="61be0-435">该示例通过 `Memory.Alloc` 分配256字节的内存，并使用从0到255的值增加内存块。</span><span class="sxs-lookup"><span data-stu-id="61be0-435">The example allocates 256 bytes of memory through `Memory.Alloc` and initializes the memory block with values increasing from 0 to 255.</span></span> <span data-ttu-id="61be0-436">然后，它分配一个256元素字节数组，并使用 `Memory.Copy` 将内存块的内容复制到字节数组中。</span><span class="sxs-lookup"><span data-stu-id="61be0-436">It then allocates a 256 element byte array and uses `Memory.Copy` to copy the contents of the memory block into the byte array.</span></span> <span data-ttu-id="61be0-437">最后，使用 `Memory.Free` 释放内存块，并在控制台上输出字节数组的内容。</span><span class="sxs-lookup"><span data-stu-id="61be0-437">Finally, the memory block is freed using `Memory.Free` and the contents of the byte array are output on the console.</span></span>
