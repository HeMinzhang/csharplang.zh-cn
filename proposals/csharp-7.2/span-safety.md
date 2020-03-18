---
ms.openlocfilehash: 6088a5cd41926d828013f1b8e5736fd2b7939e44
ms.sourcegitcommit: da452002c3f472165a0e1fa7759f494cc703ae31
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2019
ms.locfileid: "79483960"
---
# <a name="compile-time-enforcement-of-safety-for-ref-like-types"></a><span data-ttu-id="a1135-101">对类似引用类型执行安全编译的时间</span><span class="sxs-lookup"><span data-stu-id="a1135-101">Compile time enforcement of safety for ref-like types</span></span>

## <a name="introduction"></a><span data-ttu-id="a1135-102">介绍</span><span class="sxs-lookup"><span data-stu-id="a1135-102">Introduction</span></span>

<span data-ttu-id="a1135-103">处理类型（如 `Span<T>` 和 `ReadOnlySpan<T>`）时其他安全规则的主要原因是，此类类型必须局限于执行堆栈。</span><span class="sxs-lookup"><span data-stu-id="a1135-103">The main reason for the additional safety rules when dealing with types like `Span<T>` and `ReadOnlySpan<T>` is that such types must be confined to the execution stack.</span></span>
 
<span data-ttu-id="a1135-104">`Span<T>` 和类似类型必须为仅堆栈类型有两个原因。</span><span class="sxs-lookup"><span data-stu-id="a1135-104">There are two reasons why `Span<T>` and similar types must be a stack-only types.</span></span>

1. <span data-ttu-id="a1135-105">`Span<T>` 在语义上是包含引用和范围 `(ref T data, int length)`的结构。</span><span class="sxs-lookup"><span data-stu-id="a1135-105">`Span<T>` is semantically a struct containing a reference and a range - `(ref T data, int length)`.</span></span> <span data-ttu-id="a1135-106">不考虑实际的实现，因此，对此类结构的写入将不是原子的。</span><span class="sxs-lookup"><span data-stu-id="a1135-106">Regardless of actual implementation, writes to such struct would not be atomic.</span></span> <span data-ttu-id="a1135-107">此类结构的并发 "撕裂" 将导致 `length` 与 `data`不匹配，导致超出范围的访问和类型安全冲突，最终可能会导致垃圾回收堆在看似 "safe" 代码中损坏。</span><span class="sxs-lookup"><span data-stu-id="a1135-107">Concurrent "tearing" of such struct would lead to the possibility of `length` not matching the `data`, causing out-of-range accesses and type-safety violations, which ultimately could result in GC heap corruption in seemingly "safe" code.</span></span>
2. <span data-ttu-id="a1135-108">`Span<T>` 的某些实现在字面上包含一个托管指针。</span><span class="sxs-lookup"><span data-stu-id="a1135-108">Some implementations of `Span<T>` literally contain a managed pointer in one of its fields.</span></span> <span data-ttu-id="a1135-109">不支持将托管指针作为堆对象的字段，管理用于将托管指针放置在 GC 堆上的代码通常会在 JIT 时崩溃。</span><span class="sxs-lookup"><span data-stu-id="a1135-109">Managed pointers are not supported as fields of heap objects and code that manages to put a managed pointer on the GC heap typically crashes at JIT time.</span></span>

<span data-ttu-id="a1135-110">如果 `Span<T>` 的实例被约束为仅存在于执行堆栈中，则上述所有问题都将会缓解。</span><span class="sxs-lookup"><span data-stu-id="a1135-110">All the above problems would be alleviated if instances of `Span<T>` are constrained to exist only on the execution stack.</span></span> 

<span data-ttu-id="a1135-111">由于撰写导致的其他问题。</span><span class="sxs-lookup"><span data-stu-id="a1135-111">An additional problem arises due to composition.</span></span> <span data-ttu-id="a1135-112">通常需要生成更复杂的数据类型，以便嵌入 `Span<T>` 和 `ReadOnlySpan<T>` 实例。</span><span class="sxs-lookup"><span data-stu-id="a1135-112">It would be generally desirable to build more complex data types that would embed `Span<T>` and `ReadOnlySpan<T>` instances.</span></span> <span data-ttu-id="a1135-113">此类复合类型必须是结构，并将分担 `Span<T>`的所有危险和要求。</span><span class="sxs-lookup"><span data-stu-id="a1135-113">Such composite types would have to be structs and would share all the hazards and requirements of `Span<T>`.</span></span> <span data-ttu-id="a1135-114">因此，应查看此处所述的安全规则，使其适用于 **_类似于引用类型_** 的整个范围。</span><span class="sxs-lookup"><span data-stu-id="a1135-114">As a result the safety rules described here should be viewed as applicable to the whole range of **_ref-like types_**.</span></span>

<span data-ttu-id="a1135-115">[草稿语言规范](#draft-language-specification)旨在确保仅在堆栈上出现类似于引用类型的值。</span><span class="sxs-lookup"><span data-stu-id="a1135-115">The [draft language specification](#draft-language-specification) is intended to ensure that values of a ref-like type occurs only on the stack.</span></span>

## <a name="generalized-ref-like-types-in-source-code"></a><span data-ttu-id="a1135-116">源代码中的一般化 `ref-like` 类型</span><span class="sxs-lookup"><span data-stu-id="a1135-116">Generalized `ref-like` types in source code</span></span>

<span data-ttu-id="a1135-117">使用 `ref` 修饰符在源代码中显式标记 `ref-like` 结构：</span><span class="sxs-lookup"><span data-stu-id="a1135-117">`ref-like` structs are explicitly marked in the source code using `ref` modifier:</span></span>

```csharp
ref struct TwoSpans<T>
{
    // can have ref-like instance fields
    public Span<T> first;
    public Span<T> second;
} 

// error: arrays of ref-like types are not allowed. 
TwoSpans<T>[] arr = null;

```

<span data-ttu-id="a1135-118">将结构指定为同类引用后，该结构将允许该结构具有类似于引用的实例字段，并且还会使与该结构相关的 ref 类型的所有要求。</span><span class="sxs-lookup"><span data-stu-id="a1135-118">Designating a struct as ref-like will allow the struct to have ref-like instance fields and will also make all the requirements of ref-like types applicable to the struct.</span></span> 

## <a name="metadata-representation-or-ref-like-structs"></a><span data-ttu-id="a1135-119">元数据表示形式或类似于引用的结构</span><span class="sxs-lookup"><span data-stu-id="a1135-119">Metadata representation or ref-like structs</span></span>

<span data-ttu-id="a1135-120">类似于引用的结构将用**system.runtime.compilerservices. IsRefLikeAttribute**属性进行标记。</span><span class="sxs-lookup"><span data-stu-id="a1135-120">Ref-like structs will be marked with **System.Runtime.CompilerServices.IsRefLikeAttribute** attribute.</span></span>

<span data-ttu-id="a1135-121">该属性将添加到公共基库，如 `mscorlib`。</span><span class="sxs-lookup"><span data-stu-id="a1135-121">The attribute will be added to common base libraries such as `mscorlib`.</span></span> <span data-ttu-id="a1135-122">在这种情况下，如果该属性不可用，编译器将生成一个与其他嵌入的按需属性（如 `IsReadOnlyAttribute`）相似的内部属性。</span><span class="sxs-lookup"><span data-stu-id="a1135-122">In a case if the attribute is not available, compiler will generate an internal one similarly to other embedded-on-demand attributes such as `IsReadOnlyAttribute`.</span></span>

<span data-ttu-id="a1135-123">将采取额外的措施来防止在编译器中不熟悉安全规则（这包括C#在其上实现此功能的编译器之前）使用类似于引用的结构。</span><span class="sxs-lookup"><span data-stu-id="a1135-123">An additional measure will be taken to prevent the use of ref-like structs in compilers not familiar with the safety rules (this includes C# compilers prior to the one in which this feature is implemented).</span></span> 

<span data-ttu-id="a1135-124">如果没有其他适用于旧编译器的替代方法，不提供服务，则会将具有已知字符串的 `Obsolete` 属性添加到所有类似于引用的结构。</span><span class="sxs-lookup"><span data-stu-id="a1135-124">Having no other good alternatives that work in old compilers without servicing, an `Obsolete` attribute with a known string will be added to all ref-like structs.</span></span> <span data-ttu-id="a1135-125">知道如何使用类似引用的类型的编译器将忽略此特定形式的 `Obsolete`。</span><span class="sxs-lookup"><span data-stu-id="a1135-125">Compilers that know how to use ref-like types will ignore this particular form of `Obsolete`.</span></span>

<span data-ttu-id="a1135-126">典型的元数据表示形式：</span><span class="sxs-lookup"><span data-stu-id="a1135-126">A typical metadata representation:</span></span>

```csharp
    [IsRefLike]
    [Obsolete("Types with embedded references are not supported in this version of your compiler.")]
    public struct TwoSpans<T>
    {
       // . . . .
    }
```

<span data-ttu-id="a1135-127">注意：不是这样做的目标，因此在旧编译器上使用类似于引用的类型会导致100%。</span><span class="sxs-lookup"><span data-stu-id="a1135-127">NOTE: it is not the goal to make it so that any use of ref-like types on old compilers fails 100%.</span></span> <span data-ttu-id="a1135-128">这并不是必需的。</span><span class="sxs-lookup"><span data-stu-id="a1135-128">That is hard to achieve and is not strictly necessary.</span></span> <span data-ttu-id="a1135-129">例如，总会有一种方法可以使用动态代码来解决 `Obsolete`，例如通过反射创建类似于引用类型的数组。</span><span class="sxs-lookup"><span data-stu-id="a1135-129">For example there would always be a way to get around the `Obsolete` using dynamic code or, for example, creating an array of ref-like types through reflection.</span></span>

<span data-ttu-id="a1135-130">特别是，如果用户想要在类似于引用的类型上实际放置 `Obsolete` 或 `Deprecated` 属性，则除了不发出预定义的类型之外，我们还没有选择，因为 `Obsolete` 属性不能应用多次。</span><span class="sxs-lookup"><span data-stu-id="a1135-130">In particular, if user wants to actually put an `Obsolete` or `Deprecated` attribute on a ref-like type, we will have no choice other than not emitting the predefined one since `Obsolete` attribute cannot be applied more than once..</span></span>  

## <a name="examples"></a><span data-ttu-id="a1135-131">示例：</span><span class="sxs-lookup"><span data-stu-id="a1135-131">Examples:</span></span>

```csharp
SpanLikeType M1(ref SpanLikeType x, Span<byte> y)
{
    // this is all valid, unconcerned with stack-referring stuff
    var local = new SpanLikeType(y);
    x = local;
    return x;
}

void Test1(ref SpanLikeType param1, Span<byte> param2)
{
    Span<byte> stackReferring1 = stackalloc byte[10];
    var stackReferring2 = new SpanLikeType(stackReferring1);

    // this is allowed
    stackReferring2 = M1(ref stackReferring2, stackReferring1);

    // this is NOT allowed
    stackReferring2 = M1(ref param1, stackReferring1);

    // this is NOT allowed
    param1 = M1(ref stackReferring2, stackReferring1);

    // this is NOT allowed
    param2 = stackReferring1.Slice(10);

    // this is allowed
    param1 = new SpanLikeType(param2);

    // this is allowed
    stackReferring2 = param1;
}

ref SpanLikeType M2(ref SpanLikeType x)
{
    return ref x;
}

ref SpanLikeType Test2(ref SpanLikeType param1, Span<byte> param2)
{
    Span<byte> stackReferring1 = stackalloc byte[10];
    var stackReferring2 = new SpanLikeType(stackReferring1);

    ref var stackReferring3 = M2(ref stackReferring2);

    // this is allowed
    stackReferring3 = M1(ref stackReferring2, stackReferring1);

    // this is allowed
    M2(ref stackReferring3) = stackReferring2;

    // this is NOT allowed
    M1(ref param1) = stackReferring2;

    // this is NOT allowed
    param1 = stackReferring3;

    // this is NOT allowed
    return ref stackReferring3;

    // this is allowed
    return ref param1;
}

```

----------------

## <a name="draft-language-specification"></a><span data-ttu-id="a1135-132">草稿语言规范</span><span class="sxs-lookup"><span data-stu-id="a1135-132">Draft language specification</span></span>

<span data-ttu-id="a1135-133">下面我们介绍一组类似于引用类型（`ref struct`）的安全规则，以确保这些类型的值仅出现在堆栈上。</span><span class="sxs-lookup"><span data-stu-id="a1135-133">Below we describe a set of safety rules for ref-like types (`ref struct`s) to ensure that values of these types occur only on the stack.</span></span> <span data-ttu-id="a1135-134">如果无法通过引用传递局部变量，则可以使用一组不同的更简单的安全规则。</span><span class="sxs-lookup"><span data-stu-id="a1135-134">A different, simpler set of safety rules would be possible if locals cannot be passed by reference.</span></span> <span data-ttu-id="a1135-135">此规范还允许重新指定 ref 局部变量。</span><span class="sxs-lookup"><span data-stu-id="a1135-135">This specification would also permit the safe reassignment of ref locals.</span></span>

### <a name="overview"></a><span data-ttu-id="a1135-136">概述</span><span class="sxs-lookup"><span data-stu-id="a1135-136">Overview</span></span>

<span data-ttu-id="a1135-137">在编译时，与每个表达式相关联的是允许该表达式转义的范围的概念，即 "安全到转义"。</span><span class="sxs-lookup"><span data-stu-id="a1135-137">We associate with each expression at compile-time the concept of what scope that expression is permitted to escape to, "safe-to-escape".</span></span> <span data-ttu-id="a1135-138">同样，对于每个左值，我们都将保留对其引用范围的概念，以允许对其进行转义，即 "引用安全到转义"。</span><span class="sxs-lookup"><span data-stu-id="a1135-138">Similarly, for each lvalue we maintain a concept of what scope a reference to it is permitted to escape to, "ref-safe-to-escape".</span></span> <span data-ttu-id="a1135-139">对于给定的左值表达式，这些表达式可能不同。</span><span class="sxs-lookup"><span data-stu-id="a1135-139">For a given lvalue expression, these may be different.</span></span>

<span data-ttu-id="a1135-140">它们类似于引用局部变量功能的 "安全返回"，但更细化。</span><span class="sxs-lookup"><span data-stu-id="a1135-140">These are analogous to the "safe to return" of the ref locals feature, but it is more fine-grained.</span></span> <span data-ttu-id="a1135-141">如果表达式的 "安全到返回" 只记录了（或不是）它是否可以将封闭方法作为一个整体进行转义，则安全到转义的记录可能会被转义的范围（它可能不会被转义的范围）。</span><span class="sxs-lookup"><span data-stu-id="a1135-141">Where the "safe-to-return" of an expression records only whether (or not) it may escape the enclosing method as a whole, the safe-to-escape records which scope it may escape to (which scope it may not escape beyond).</span></span> <span data-ttu-id="a1135-142">基本安全机制按如下方式强制执行。</span><span class="sxs-lookup"><span data-stu-id="a1135-142">The basic safety mechanism is enforced as follows.</span></span> <span data-ttu-id="a1135-143">给定从具有安全到转义范围 S1 的 expression E1 到（左值）表达式 E2 的赋值，使用安全到转义范围 S2，如果 S2 的范围比 S1 更宽，则是错误的。</span><span class="sxs-lookup"><span data-stu-id="a1135-143">Given an assignment from an expression E1 with a safe-to-escape scope S1, to an (lvalue) expression E2 with safe-to-escape scope S2, it is an error if S2 is a wider scope than S1.</span></span> <span data-ttu-id="a1135-144">按照构造，两个作用域 S1 和 S2 在一个嵌套关系中，因为合法表达式始终是从包含表达式的某些范围中恢复为安全的。</span><span class="sxs-lookup"><span data-stu-id="a1135-144">By construction, the two scopes S1 and S2 are in a nesting relationship, because a legal expression is always safe-to-return from some scope enclosing the expression.</span></span>

<span data-ttu-id="a1135-145">在这种情况下，为了进行分析，只支持两个范围：外部到方法和方法的顶级范围。</span><span class="sxs-lookup"><span data-stu-id="a1135-145">For the time being it is sufficient, for the purpose of the analysis, to support just two scopes - external to the method, and top-level scope of the method.</span></span> <span data-ttu-id="a1135-146">这是因为无法创建具有内部范围的类似引用的值，并且引用局部变量不支持重新赋值。</span><span class="sxs-lookup"><span data-stu-id="a1135-146">That is because ref-like values with inner scopes cannot be created and ref locals do not support re-assignment.</span></span> <span data-ttu-id="a1135-147">不过，这些规则可支持两个以上的作用域级别。</span><span class="sxs-lookup"><span data-stu-id="a1135-147">The rules, however, can support more than two scope levels.</span></span>

<span data-ttu-id="a1135-148">若要计算表达式的*安全恢复返回*状态和管理表达式合法性的规则，请遵循。</span><span class="sxs-lookup"><span data-stu-id="a1135-148">The precise rules for computing the *safe-to-return* status of an expression, and the rules governing the legality of expressions, follow.</span></span>

### <a name="ref-safe-to-escape"></a><span data-ttu-id="a1135-149">引用安全-转义</span><span class="sxs-lookup"><span data-stu-id="a1135-149">ref-safe-to-escape</span></span>

<span data-ttu-id="a1135-150">*Ref safe 到 escape*是一个范围，其中包含左值表达式，对 lvalue 的引用可以安全地转义给它。</span><span class="sxs-lookup"><span data-stu-id="a1135-150">The *ref-safe-to-escape* is a scope, enclosing an lvalue expression, to which it is safe for a ref to the lvalue to escape to.</span></span> <span data-ttu-id="a1135-151">如果该作用域是整个方法，我们说到左值的引用可以*安全地*从方法返回。</span><span class="sxs-lookup"><span data-stu-id="a1135-151">If that scope is the entire method, we say that a ref to the lvalue is *safe to return* from the method.</span></span>

### <a name="safe-to-escape"></a><span data-ttu-id="a1135-152">安全到转义</span><span class="sxs-lookup"><span data-stu-id="a1135-152">safe-to-escape</span></span>

<span data-ttu-id="a1135-153">*安全 to escape*是一个包含表达式的范围，其中的值可以安全地转义。</span><span class="sxs-lookup"><span data-stu-id="a1135-153">The *safe-to-escape* is a scope, enclosing an expression, to which it is safe for the value to escape to.</span></span> <span data-ttu-id="a1135-154">如果该作用域是整个方法，我们就会说，*可以安全地*从方法返回值。</span><span class="sxs-lookup"><span data-stu-id="a1135-154">If that scope is the entire method, we say that a the value is *safe to return* from the method.</span></span>

<span data-ttu-id="a1135-155">类型不是 `ref struct` 类型的表达式是从整个封闭方法中*安全返回*的。</span><span class="sxs-lookup"><span data-stu-id="a1135-155">An expression whose type is not a `ref struct` type is *safe-to-return* from the entire enclosing method.</span></span> <span data-ttu-id="a1135-156">否则，请参阅下面的规则。</span><span class="sxs-lookup"><span data-stu-id="a1135-156">Otherwise we refer to the rules below.</span></span>

#### <a name="parameters"></a><span data-ttu-id="a1135-157">parameters</span><span class="sxs-lookup"><span data-stu-id="a1135-157">Parameters</span></span>

<span data-ttu-id="a1135-158">指定形参的左值是引用*安全转义*（按引用），如下所示：</span><span class="sxs-lookup"><span data-stu-id="a1135-158">An lvalue designating a formal parameter is *ref-safe-to-escape* (by reference) as follows:</span></span>
- <span data-ttu-id="a1135-159">如果参数是一个 `ref`、`out`或 `in` 参数，则为整个方法（例如通过 `return ref` 语句）进行*引用安全的转义*;本来</span><span class="sxs-lookup"><span data-stu-id="a1135-159">If the parameter is a `ref`, `out`, or `in` parameter, it is *ref-safe-to-escape* from the entire method (e.g. by a `return ref` statement); otherwise</span></span>
- <span data-ttu-id="a1135-160">如果参数是结构类型的 `this` 参数，则它是对方法的顶级范围（而不是从整个方法本身）进行*引用安全的引用*;[示例](#struct-this-escape)</span><span class="sxs-lookup"><span data-stu-id="a1135-160">If the parameter is the `this` parameter of a struct type, it is *ref-safe-to-escape* to the top-level scope of the method (but not from the entire method itself); [Sample](#struct-this-escape)</span></span>
- <span data-ttu-id="a1135-161">否则，该参数是一个值参数，它是对方法的顶级范围（而不是从方法本身）的*引用安全的转义*。</span><span class="sxs-lookup"><span data-stu-id="a1135-161">Otherwise the parameter is a value parameter, and it is *ref-safe-to-escape* to the top-level scope of the method (but not from the method itself).</span></span>

<span data-ttu-id="a1135-162">指定形参使用的右值的表达式（通过值）从整个方法（例如，通过 `return` 语句）进行*安全的转义*。</span><span class="sxs-lookup"><span data-stu-id="a1135-162">An expression that is an rvalue designating the use of a formal parameter is *safe-to-escape* (by value) from the entire method (e.g. by a `return` statement).</span></span> <span data-ttu-id="a1135-163">这也适用于 `this` 参数。</span><span class="sxs-lookup"><span data-stu-id="a1135-163">This applies to the `this` parameter as well.</span></span>

#### <a name="locals"></a><span data-ttu-id="a1135-164">局部变量</span><span class="sxs-lookup"><span data-stu-id="a1135-164">Locals</span></span>

<span data-ttu-id="a1135-165">指定局部变量的左值是引用*安全*的（按引用），如下所示：</span><span class="sxs-lookup"><span data-stu-id="a1135-165">An lvalue designating a local variable is *ref-safe-to-escape* (by reference) as follows:</span></span>
- <span data-ttu-id="a1135-166">如果变量是 `ref` 变量，则它的*ref 安全到转义*将从其初始化表达式的*ref 安全对转义*;本来</span><span class="sxs-lookup"><span data-stu-id="a1135-166">If the variable is a `ref` variable, then its *ref-safe-to-escape* is taken from the *ref-safe-to-escape* of its initializing expression; otherwise</span></span>
- <span data-ttu-id="a1135-167">变量是*引用安全*的，可对它的声明范围进行转义。</span><span class="sxs-lookup"><span data-stu-id="a1135-167">The variable is *ref-safe-to-escape* the scope in which it was declared.</span></span>

<span data-ttu-id="a1135-168">指定局部变量使用的右值表达式是*安全转义*（通过值），如下所示：</span><span class="sxs-lookup"><span data-stu-id="a1135-168">An expression that is an rvalue designating the use of a local variable is *safe-to-escape* (by value) as follows:</span></span>
- <span data-ttu-id="a1135-169">但上面的一般规则是，其类型不是 `ref struct` 类型的本地是*安全地*从整个封闭方法返回的。</span><span class="sxs-lookup"><span data-stu-id="a1135-169">But the general rule above, a local whose type is not a `ref struct` type is *safe-to-return* from the entire enclosing method.</span></span>
- <span data-ttu-id="a1135-170">如果该变量是 `foreach` 循环的迭代变量，则该变量的*safe 到 escape*范围与 `foreach` 循环的表达式的*安全转义*等效。</span><span class="sxs-lookup"><span data-stu-id="a1135-170">If the variable is an iteration variable of a `foreach` loop, then the variable's *safe-to-escape* scope is the same as the *safe-to-escape* of the `foreach` loop's expression.</span></span>
- <span data-ttu-id="a1135-171">在声明点上，不能从整个封闭方法中*安全地返回*`ref struct` 类型的本地和在声明点未初始化。</span><span class="sxs-lookup"><span data-stu-id="a1135-171">A local of `ref struct` type and uninitialized at the point of declaration is *safe-to-return* from the entire enclosing method.</span></span>
- <span data-ttu-id="a1135-172">否则，该变量的类型为 `ref struct` 类型，该变量的声明需要初始值设定项。</span><span class="sxs-lookup"><span data-stu-id="a1135-172">Otherwise the variable's type is a `ref struct` type, and the variable's declaration requires an initializer.</span></span> <span data-ttu-id="a1135-173">变量的*安全到转义*的范围与它的初始值设定项的*安全转义*。</span><span class="sxs-lookup"><span data-stu-id="a1135-173">The variable's *safe-to-escape* scope is the same as the *safe-to-escape* of its initializer.</span></span>

#### <a name="field-reference"></a><span data-ttu-id="a1135-174">字段引用</span><span class="sxs-lookup"><span data-stu-id="a1135-174">Field reference</span></span>

<span data-ttu-id="a1135-175">指定对字段的引用的左值 `e.F`，它是引用*安全引用转义*（按引用），如下所示：</span><span class="sxs-lookup"><span data-stu-id="a1135-175">An lvalue designating a reference to a field, `e.F`, is *ref-safe-to-escape* (by reference) as follows:</span></span>
- <span data-ttu-id="a1135-176">如果 `e` 的类型为引用类型，则它是对整个方法的引用*安全*的引用;本来</span><span class="sxs-lookup"><span data-stu-id="a1135-176">If `e` is of a reference type, it is *ref-safe-to-escape* from the entire method; otherwise</span></span>
- <span data-ttu-id="a1135-177">如果 `e` 是值类型，则将从 `e`的*ref 安全到转义*获取它的*ref 安全到转义*。</span><span class="sxs-lookup"><span data-stu-id="a1135-177">If `e` is of a value type, its *ref-safe-to-escape* is taken from the *ref-safe-to-escape* of `e`.</span></span>

<span data-ttu-id="a1135-178">指定对字段 `e.F`的引用具有*安全到转义*的范围，该范围等同于 `e`的*安全转义*。</span><span class="sxs-lookup"><span data-stu-id="a1135-178">An rvalue designating a reference to a field, `e.F`, has a *safe-to-escape* scope that is the same as the *safe-to-escape* of `e`.</span></span>

#### <a name="operators-including-"></a><span data-ttu-id="a1135-179">包括 `?:` 的运算符</span><span class="sxs-lookup"><span data-stu-id="a1135-179">Operators including `?:`</span></span>

<span data-ttu-id="a1135-180">用户定义的运算符的应用程序被视为方法调用。</span><span class="sxs-lookup"><span data-stu-id="a1135-180">The application of a user-defined operator is treated as a method invocation.</span></span>

<span data-ttu-id="a1135-181">对于生成右值的运算符（如 `e1 + e2` 或 `c ? e1 : e2`），结果的*安全对转义*是运算符的操作数的*安全转义*的最小范围。</span><span class="sxs-lookup"><span data-stu-id="a1135-181">For an operator that yields an rvalue, such as `e1 + e2` or `c ? e1 : e2`, the *safe-to-escape* of the result is the narrowest scope among the *safe-to-escape* of the operands of the operator.</span></span>  <span data-ttu-id="a1135-182">因此，对于产生右值的一元运算符（如 `+e`），该结果的*安全对转义*是操作数的*安全转义*。</span><span class="sxs-lookup"><span data-stu-id="a1135-182">As a consequence, for a unary operator that yields an rvalue, such as `+e`, the *safe-to-escape* of the result is the *safe-to-escape* of the operand.</span></span>

<span data-ttu-id="a1135-183">对于产生左值的运算符，例如 `c ? ref e1 : ref e2`</span><span class="sxs-lookup"><span data-stu-id="a1135-183">For an operator that yields an lvalue, such as `c ? ref e1 : ref e2`</span></span>
- <span data-ttu-id="a1135-184">结果的*ref 安全对转义*是运算符的操作数的*ref 安全到转义*中的最小范围。</span><span class="sxs-lookup"><span data-stu-id="a1135-184">the *ref-safe-to-escape* of the result is the narrowest scope among the *ref-safe-to-escape* of the operands of the operator.</span></span>
- <span data-ttu-id="a1135-185">操作数的*安全对转义*必须一致，这是生成的左值的*安全转义*。</span><span class="sxs-lookup"><span data-stu-id="a1135-185">the *safe-to-escape* of the operands must agree, and that is the *safe-to-escape* of the resulting lvalue.</span></span>

#### <a name="method-invocation"></a><span data-ttu-id="a1135-186">方法调用</span><span class="sxs-lookup"><span data-stu-id="a1135-186">Method invocation</span></span>

<span data-ttu-id="a1135-187">由引用返回的方法调用生成的左值 `e1.M(e2, ...)` 是*引用安全*的最小的范围：</span><span class="sxs-lookup"><span data-stu-id="a1135-187">An lvalue resulting from a ref-returning method invocation `e1.M(e2, ...)` is *ref-safe-to-escape* the smallest of the following scopes:</span></span>
- <span data-ttu-id="a1135-188">整个封闭方法</span><span class="sxs-lookup"><span data-stu-id="a1135-188">The entire enclosing method</span></span>
- <span data-ttu-id="a1135-189">所有 `ref` 和 `out` 参数表达式（接收方除外）的*ref 安全转义*</span><span class="sxs-lookup"><span data-stu-id="a1135-189">the *ref-safe-to-escape* of all `ref` and `out` argument expressions (excluding the receiver)</span></span>
- <span data-ttu-id="a1135-190">对于方法的每个 `in` 参数，如果有一个对应的表达式是左值，则为它的*ref 安全到转义*，否则为最近的封闭范围</span><span class="sxs-lookup"><span data-stu-id="a1135-190">For each `in` parameter of the method, if there is a corresponding expression that is an lvalue, its *ref-safe-to-escape*, otherwise the nearest enclosing scope</span></span>
- <span data-ttu-id="a1135-191">所有自变量表达式（包括接收方）的*安全转义*</span><span class="sxs-lookup"><span data-stu-id="a1135-191">the *safe-to-escape* of all argument expressions (including the receiver)</span></span>

> <span data-ttu-id="a1135-192">注意：必须具有最后一个项目符号才能处理代码，如</span><span class="sxs-lookup"><span data-stu-id="a1135-192">Note: the last bullet is necessary to handle code such as</span></span>
> ```csharp
> var sp = new Span(...)
> return ref sp[0];
> ```
> <span data-ttu-id="a1135-193">或</span><span class="sxs-lookup"><span data-stu-id="a1135-193">or</span></span>
> ```csharp
> return ref M(sp, 0);
> ```

<span data-ttu-id="a1135-194">方法调用生成的右值 `e1.M(e2, ...)` 是从以下范围的最小值中*安全到转义*：</span><span class="sxs-lookup"><span data-stu-id="a1135-194">An rvalue resulting from a method invocation `e1.M(e2, ...)` is *safe-to-escape* from the smallest of the following scopes:</span></span>
- <span data-ttu-id="a1135-195">整个封闭方法</span><span class="sxs-lookup"><span data-stu-id="a1135-195">The entire enclosing method</span></span>
- <span data-ttu-id="a1135-196">所有自变量表达式（包括接收方）的*安全转义*</span><span class="sxs-lookup"><span data-stu-id="a1135-196">the *safe-to-escape* of all argument expressions (including the receiver)</span></span>

#### <a name="an-rvalue"></a><span data-ttu-id="a1135-197">右值</span><span class="sxs-lookup"><span data-stu-id="a1135-197">An Rvalue</span></span>
<span data-ttu-id="a1135-198">右值是从最近的封闭范围中*引用安全的引用*。</span><span class="sxs-lookup"><span data-stu-id="a1135-198">An rvalue is *ref-safe-to-escape* from the nearest enclosing scope.</span></span> <span data-ttu-id="a1135-199">例如，在 `M(ref d.Length)` 的调用（其中 `d` 类型 `dynamic`）中，会发生这种情况。</span><span class="sxs-lookup"><span data-stu-id="a1135-199">This occurs for example in an invocation such as `M(ref d.Length)` where `d` is of type `dynamic`.</span></span> <span data-ttu-id="a1135-200">它还与（可能是 subsumes）处理与 `in` 参数相对应的参数的处理方式一致。 \*</span><span class="sxs-lookup"><span data-stu-id="a1135-200">It is also consistent with (and perhaps subsumes) our handling of arguments corresponding to `in` parameters.\*</span></span>

#### <a name="property-invocations"></a><span data-ttu-id="a1135-201">属性调用</span><span class="sxs-lookup"><span data-stu-id="a1135-201">Property invocations</span></span>

<span data-ttu-id="a1135-202">通过上述规则将属性调用（`get` 或 `set`）视为基础方法的方法调用。</span><span class="sxs-lookup"><span data-stu-id="a1135-202">A property invocation (either `get` or `set`) it treated as a method invocation of the underlying method by the above rules.</span></span>

#### `stackalloc`

<span data-ttu-id="a1135-203">Stackalloc 表达式是对方法的顶级范围（而不是整个方法本身）进行*安全转义*的右值。</span><span class="sxs-lookup"><span data-stu-id="a1135-203">A stackalloc expression is an rvalue that is *safe-to-escape* to the top-level scope of the method (but not from the entire method itself).</span></span>

#### <a name="constructor-invocations"></a><span data-ttu-id="a1135-204">构造函数调用</span><span class="sxs-lookup"><span data-stu-id="a1135-204">Constructor invocations</span></span>

<span data-ttu-id="a1135-205">调用构造函数的 `new` 表达式将与被视为返回正在构造的类型的方法调用的规则服从。</span><span class="sxs-lookup"><span data-stu-id="a1135-205">A `new` expression that invokes a constructor obeys the same rules as a method invocation that is considered to return the type being constructed.</span></span>

<span data-ttu-id="a1135-206">此外，如果存在初始值设定项，则不会以递归方式在对象初始值设定项表达式的所有参数/操作数的最小*转义*范围内进行*安全到*转义。</span><span class="sxs-lookup"><span data-stu-id="a1135-206">In addition *safe-to-escape* is no wider than the smallest of the *safe-to-escape* of all arguments/operands of the object initializer expressions, recursively, if initializer is present.</span></span> 

#### <a name="span-constructor"></a><span data-ttu-id="a1135-207">Span 构造函数</span><span class="sxs-lookup"><span data-stu-id="a1135-207">Span constructor</span></span>
<span data-ttu-id="a1135-208">语言依赖于 `Span<T>` 不具有以下形式的构造函数：</span><span class="sxs-lookup"><span data-stu-id="a1135-208">The language relies on `Span<T>` not having a constructor of the following form:</span></span>

```csharp
void Example(ref int x)
{
    // Create a span of length one
    var span = new Span<int>(ref x); 
}
```

<span data-ttu-id="a1135-209">此类构造函数使 `Span<T>` 用作与 `ref` 字段不区分的字段。</span><span class="sxs-lookup"><span data-stu-id="a1135-209">Such a constructor makes `Span<T>` which are used as fields indistinguishable from a `ref` field.</span></span> <span data-ttu-id="a1135-210">本文档中所述的安全规则依赖于不是或 .NET 中C#的有效构造 `ref` 字段。</span><span class="sxs-lookup"><span data-stu-id="a1135-210">The safety rules described in this document depend on `ref` fields not being a valid construct in C#, or .NET.</span></span>

#### <a name="default-expressions"></a><span data-ttu-id="a1135-211">`default` 表达式</span><span class="sxs-lookup"><span data-stu-id="a1135-211">`default` expressions</span></span>

<span data-ttu-id="a1135-212">`default` 表达式是从整个封闭方法中*安全地转义*的。</span><span class="sxs-lookup"><span data-stu-id="a1135-212">A `default` expression is *safe-to-escape* from the entire enclosing method.</span></span>

## <a name="language-constraints"></a><span data-ttu-id="a1135-213">语言约束</span><span class="sxs-lookup"><span data-stu-id="a1135-213">Language Constraints</span></span>

<span data-ttu-id="a1135-214">我们希望确保没有 `ref` 本地变量，并且没有 `ref struct` 类型的变量引用堆栈内存或不再处于活动状态的变量。</span><span class="sxs-lookup"><span data-stu-id="a1135-214">We wish to ensure that no `ref` local variable, and no variable of `ref struct` type, refers to stack memory or variables that are no longer alive.</span></span> <span data-ttu-id="a1135-215">因此，我们具有以下语言约束：</span><span class="sxs-lookup"><span data-stu-id="a1135-215">We therefore have the following language constraints:</span></span>

- <span data-ttu-id="a1135-216">Ref 参数既不是引用本地，也不能将 `ref struct` 类型的参数或局部变量提升为 lambda 或局部函数。</span><span class="sxs-lookup"><span data-stu-id="a1135-216">Neither a ref parameter, nor a ref local, nor a parameter or local of a `ref struct` type can be lifted into a lambda or local function.</span></span>

- <span data-ttu-id="a1135-217">`ref struct` 类型的 ref 参数和参数都不能是迭代器方法或 `async` 方法的参数。</span><span class="sxs-lookup"><span data-stu-id="a1135-217">Neither a ref parameter nor a parameter of a `ref struct` type may be an argument on an iterator method or an `async` method.</span></span>

- <span data-ttu-id="a1135-218">在 `yield return` 语句或 `await` 表达式的点，引用本地和 `ref struct` 类型都不能在范围内。</span><span class="sxs-lookup"><span data-stu-id="a1135-218">Neither a ref local, nor a local of a `ref struct` type may be in scope at the point of a `yield return` statement or an `await` expression.</span></span>

- <span data-ttu-id="a1135-219">`ref struct` 类型不能用作类型参数或元组类型中的元素类型。</span><span class="sxs-lookup"><span data-stu-id="a1135-219">A `ref struct` type may not be used as a type argument, or as an element type in a tuple type.</span></span>

- <span data-ttu-id="a1135-220">`ref struct` 类型可能不是字段的声明类型，只是它可能是另一个 `ref struct`的实例字段的声明类型。</span><span class="sxs-lookup"><span data-stu-id="a1135-220">A `ref struct` type may not be the declared type of a field, except that it may be the declared type of an instance field of another `ref struct`.</span></span>

- <span data-ttu-id="a1135-221">`ref struct` 类型不能是数组的元素类型。</span><span class="sxs-lookup"><span data-stu-id="a1135-221">A `ref struct` type may not be the element type of an array.</span></span>

- <span data-ttu-id="a1135-222">`ref struct` 类型的值不能装箱：</span><span class="sxs-lookup"><span data-stu-id="a1135-222">A value of a `ref struct` type may not be boxed:</span></span>
  - <span data-ttu-id="a1135-223">没有从 `ref struct` 类型到类型 `object` 或类型 `System.ValueType`的转换。</span><span class="sxs-lookup"><span data-stu-id="a1135-223">There is no conversion from a `ref struct` type to the type `object` or the type `System.ValueType`.</span></span>
  - <span data-ttu-id="a1135-224">不能声明一个 `ref struct` 类型来实现任何接口</span><span class="sxs-lookup"><span data-stu-id="a1135-224">A `ref struct` type may not be declared to implement any interface</span></span>
  - <span data-ttu-id="a1135-225">在 `object` 或未在 `System.ValueType` 中声明但在 `ref struct` 类型中未重写的实例方法可以使用该 `ref struct` 类型的接收方调用。</span><span class="sxs-lookup"><span data-stu-id="a1135-225">No instance method declared in `object` or in `System.ValueType` but not overridden in a `ref struct` type may be called with a receiver of that `ref struct` type.</span></span>
  - <span data-ttu-id="a1135-226">不能通过方法转换为委托类型来捕获 `ref struct` 类型的实例方法。</span><span class="sxs-lookup"><span data-stu-id="a1135-226">No instance method of a `ref struct` type may be captured by method conversion to a delegate type.</span></span>

- <span data-ttu-id="a1135-227">对于引用重新分配 `ref e1 = ref e2`，`e2` 的*ref 安全到转义符*的范围必须至少与 `e1`的*ref 安全对转义*。</span><span class="sxs-lookup"><span data-stu-id="a1135-227">For a ref reassignment `ref e1 = ref e2`, the *ref-safe-to-escape* of `e2` must be at least as wide a scope as the *ref-safe-to-escape* of `e1`.</span></span>

- <span data-ttu-id="a1135-228">对于 ref return 语句 `return ref e1`，`e1` 的*ref 安全到转义*必须是整个方法的*引用*安全转义。</span><span class="sxs-lookup"><span data-stu-id="a1135-228">For a ref return statement `return ref e1`, the *ref-safe-to-escape* of `e1` must be *ref-safe-to-escape* from the entire method.</span></span> <span data-ttu-id="a1135-229">（TODO：我们还需要一个规则，该规则 `e1` 必须对整个方法是*安全*的，还是冗余的？）</span><span class="sxs-lookup"><span data-stu-id="a1135-229">(TODO: Do we also need a rule that `e1` must be *safe-to-escape* from the entire method, or is that redundant?)</span></span>

- <span data-ttu-id="a1135-230">对于 `return e1`的 return 语句，`e1` 的*安全对转义*必须是整个方法的*安全转义*。</span><span class="sxs-lookup"><span data-stu-id="a1135-230">For a return statement `return e1`, the *safe-to-escape* of `e1` must be *safe-to-escape* from the entire method.</span></span>

- <span data-ttu-id="a1135-231">对于分配 `e1 = e2`，如果 `e1` 类型是 `ref struct` 类型，则 `e2` 的*安全对转义*必须至少为范围，作为 `e1`的*安全到转义*的范围。</span><span class="sxs-lookup"><span data-stu-id="a1135-231">For an assignment `e1 = e2`, if the type of `e1` is a `ref struct` type, then the *safe-to-escape* of `e2` must be at least as wide a scope as the *safe-to-escape* of `e1`.</span></span>

- <span data-ttu-id="a1135-232">对于方法调用（如果存在 `ref struct` 类型（包括接收方）的 `ref` 或 `out` 参数（包括接收方）和*安全到转义*e1，则不会有任何参数（包括接收方）的*安全转义转义*比 E1 更小。</span><span class="sxs-lookup"><span data-stu-id="a1135-232">For a method invocation if there is a `ref` or `out` argument of a `ref struct` type (including the receiver), with *safe-to-escape* E1, then no argument (including the receiver) may have a narrower *safe-to-escape* than E1.</span></span> [<span data-ttu-id="a1135-233">示例</span><span class="sxs-lookup"><span data-stu-id="a1135-233">Sample</span></span>](#method-arguments-must-match)

- <span data-ttu-id="a1135-234">本地函数或匿名函数不能引用在封闭范围内声明的 `ref struct` 类型的本地或参数。</span><span class="sxs-lookup"><span data-stu-id="a1135-234">A local function or anonymous function may not refer to a local or parameter of `ref struct` type declared in an enclosing scope.</span></span>

> <span data-ttu-id="a1135-235">***打开问题：*** 我们需要一些规则，以便在需要溢出 await 表达式的 `ref struct` 类型的堆栈值时（例如，在代码中）生成错误</span><span class="sxs-lookup"><span data-stu-id="a1135-235">***Open Issue:*** We need some rule that permits us to produce an error when needing to spill a stack value of a `ref struct` type at an await expression, for example in the code</span></span>
> ```csharp
> Foo(new Span<int>(...), await e2);
> ```

## <a name="explanations"></a><span data-ttu-id="a1135-236">方便</span><span class="sxs-lookup"><span data-stu-id="a1135-236">Explanations</span></span>
<span data-ttu-id="a1135-237">这些说明和示例有助于解释上述许多安全规则存在的原因</span><span class="sxs-lookup"><span data-stu-id="a1135-237">These explanations and samples help explain why many of the safety rules above exist</span></span>

### <a name="method-arguments-must-match"></a><span data-ttu-id="a1135-238">方法参数必须匹配</span><span class="sxs-lookup"><span data-stu-id="a1135-238">Method Arguments Must Match</span></span>
<span data-ttu-id="a1135-239">如果调用的方法中有 `out`，`ref` 参数是 `ref struct` 包括接收方，则所有 `ref struct` 都需要具有相同的生存期。</span><span class="sxs-lookup"><span data-stu-id="a1135-239">When invoking a method where there is an `out`, `ref` parameter that is a `ref struct` including the receiver then all of the `ref struct` need to have the same lifetime.</span></span> <span data-ttu-id="a1135-240">这是必需的C# ，因为必须根据方法签名中提供的信息和调用站点上值的生存期，使其所有的决策都围绕生存期安全。</span><span class="sxs-lookup"><span data-stu-id="a1135-240">This is necessary because C# must make all of it's decisions around lifetime safety based on the information available in the signature of the method and the lifetime of the values at the call site.</span></span> 

<span data-ttu-id="a1135-241">如果有 `ref` 的参数 `ref struct`，则它们可以在其内容周围交换。</span><span class="sxs-lookup"><span data-stu-id="a1135-241">When there are `ref` parameters that are `ref struct` then there is the possiblity they could swap around their contents.</span></span> <span data-ttu-id="a1135-242">因此，在调用站点，我们必须确保所有这些**可能**的交换都是兼容的。</span><span class="sxs-lookup"><span data-stu-id="a1135-242">Hence at the call site we must ensure all of these **potential** swaps are compatible.</span></span> <span data-ttu-id="a1135-243">如果该语言未强制执行该语言，则会允许错误的代码，如下所示。</span><span class="sxs-lookup"><span data-stu-id="a1135-243">If the language didn't enforce that then it will allow for bad code like the following.</span></span>

```csharp
void M1(ref Span<int> s1)
{
    Span<int> s2 = stackalloc int[1];
    Swap(ref s1, ref s2);
}

void Swap(ref Span<int> x, ref int Span<int> y)
{
    // This will effectively assign the stackalloc to the s1 parameter and allow it
    // to escape to the caller of M1
    ref x = ref y; 
}
```

<span data-ttu-id="a1135-244">对接收器的限制是必要的，因为它的任何内容都不是引用安全的，它可以存储提供的值。</span><span class="sxs-lookup"><span data-stu-id="a1135-244">The restriction on the receiver is necessary because while none of its contents are ref-safe-to-escape it can store the provided values.</span></span> <span data-ttu-id="a1135-245">这意味着，使用不匹配的生存期，可以通过以下方式创建类型安全漏洞：</span><span class="sxs-lookup"><span data-stu-id="a1135-245">This means with mismatched lifetimes you could create a type safety hole in the following way:</span></span>

```csharp
ref struct S
{
    public Span<int> Span;

    public void Set(Span<int> span)
    {
        Span = span;
    }
}

void Broken(ref S s)
{
    Span<int> span = stackalloc int[1];

    // The result of a stackalloc is now stored in s.Span and escaped to the caller
    // of Broken
    s.Set(span); 
}
```

### <a name="struct-this-escape"></a><span data-ttu-id="a1135-246">Struct This Escape</span><span class="sxs-lookup"><span data-stu-id="a1135-246">Struct This Escape</span></span>
<span data-ttu-id="a1135-247">当涉及到跨安全规则时，实例成员中的 `this` 值将建模为成员的参数。</span><span class="sxs-lookup"><span data-stu-id="a1135-247">When it comes to span safety rules the `this` value in an instance member is modeled as a parameter to the member.</span></span> <span data-ttu-id="a1135-248">现在，对于 `struct` `this` 的类型实际上是 `ref S`，其中 `class` 只是 `S` （对于名为的 `class / struct` 的成员）。</span><span class="sxs-lookup"><span data-stu-id="a1135-248">Now for a `struct` the type of `this` is actually `ref S` where in a `class` it's simply `S` (for members of a `class / struct` named S).</span></span> 

<span data-ttu-id="a1135-249">但 `this` 具有不同于其他 `ref` 参数的转义规则。</span><span class="sxs-lookup"><span data-stu-id="a1135-249">Yet `this` has different escaping rules than other `ref` parameters.</span></span> <span data-ttu-id="a1135-250">具体而言，它不是引用安全的，而是其他参数：</span><span class="sxs-lookup"><span data-stu-id="a1135-250">Specifically it is not ref-safe-to-escape while other parameters are:</span></span>

```csharp
ref struct S
{ 
    int Field;

    // Illegal because this isn't safe to escape as ref
    ref int Get() => ref Field;

    // Legal
    ref int GetParam(ref int p) => ref p;
}
```

<span data-ttu-id="a1135-251">与 `struct` 成员调用相比，此限制的原因实际上很少。</span><span class="sxs-lookup"><span data-stu-id="a1135-251">The reason for this restriction actually has little to do with `struct` member invocation.</span></span> <span data-ttu-id="a1135-252">对于接收方为右值 `struct` 成员的成员调用，需要遵循一些规则。</span><span class="sxs-lookup"><span data-stu-id="a1135-252">There are some rules that need to be worked out with respect to member invocation on `struct` members where the receiver is an rvalue.</span></span> <span data-ttu-id="a1135-253">但这非常易学。</span><span class="sxs-lookup"><span data-stu-id="a1135-253">But that is very approachable.</span></span> 

<span data-ttu-id="a1135-254">此限制的原因实际上就是接口调用。</span><span class="sxs-lookup"><span data-stu-id="a1135-254">The reason for this restriction is actually about interface invocation.</span></span> <span data-ttu-id="a1135-255">具体而言，它会出现在下面的示例中是否应编译;</span><span class="sxs-lookup"><span data-stu-id="a1135-255">Specifically it comes down to whether or not the following sample should or should not compile;</span></span>

```csharp
interface I1
{
    ref int Get();
}

ref int Use<T>(T p)
    where T : I1
{
    return ref p.Get();
}
```

<span data-ttu-id="a1135-256">请考虑将 `T` 实例化为 `struct`的情况。</span><span class="sxs-lookup"><span data-stu-id="a1135-256">Consider the case where `T` is instantiated as a `struct`.</span></span> <span data-ttu-id="a1135-257">如果 `this` 参数为 ref-safe to escape，则返回 `p.Get` 可能指向堆栈（具体而言，它可以是 `T`的实例化类型内的字段）。</span><span class="sxs-lookup"><span data-stu-id="a1135-257">If the `this` parameter is ref-safe-to-escape then the return of `p.Get` could point to the stack (specifically it could be a field inside of the instantiated type of `T`).</span></span> <span data-ttu-id="a1135-258">这意味着，语言不允许此示例编译，因为它可以将 `ref` 返回到堆栈位置。</span><span class="sxs-lookup"><span data-stu-id="a1135-258">That means the language could not allow this sample to compile as it could be returning a `ref` to a stack location.</span></span> <span data-ttu-id="a1135-259">另一方面，如果 `this` 不是引用安全的 to escape，则 `p.Get` 无法引用堆栈，因此可以安全返回。</span><span class="sxs-lookup"><span data-stu-id="a1135-259">On the other hand if `this` is not ref-safe-to-escape then `p.Get` cannot refer to the stack and hence it's safe to return.</span></span> 

<span data-ttu-id="a1135-260">这就是 escapability 在 `struct` 中 `this` 的根本就是与接口有关的原因。</span><span class="sxs-lookup"><span data-stu-id="a1135-260">This is why the escapability of `this` in a `struct` is really all about interfaces.</span></span> <span data-ttu-id="a1135-261">绝对可以正常工作，但它已经有了折衷。</span><span class="sxs-lookup"><span data-stu-id="a1135-261">It can absolutely be made to work but it has a trade off.</span></span> <span data-ttu-id="a1135-262">设计最终会使接口更灵活。</span><span class="sxs-lookup"><span data-stu-id="a1135-262">The design eventually came down in favor of making interfaces more flexible.</span></span> 

<span data-ttu-id="a1135-263">但在今后，我们可能会放松这一点。</span><span class="sxs-lookup"><span data-stu-id="a1135-263">There is potential for us to relax this in the future though.</span></span> 

## <a name="future-considerations"></a><span data-ttu-id="a1135-264">未来的注意事项</span><span class="sxs-lookup"><span data-stu-id="a1135-264">Future Considerations</span></span>

### <a name="length-one-spant-over-ref-values"></a><span data-ttu-id="a1135-265">长度为一个跨度\<T > 超过 ref 值</span><span class="sxs-lookup"><span data-stu-id="a1135-265">Length one Span\<T> over ref values</span></span>
<span data-ttu-id="a1135-266">虽然目前不合法，但在某些情况下，在一个值上创建一个 `Span<T>` 实例的长度将是有益的：</span><span class="sxs-lookup"><span data-stu-id="a1135-266">Though not legal today there are cases where creating a length one `Span<T>` instance over a value would be beneficial:</span></span>

```csharp
void RefExample()
{
    int x = ...;

    // Today creating a length one Span<int> requires a stackalloc and a new 
    // local
    Span<int> span1 = stackalloc [] { x };
    Use(span1);
    x = span1[0]; 

    // Simpler to just allow length one span
    var span2 = new Span<int>(ref x);
    Use(span2);
}
```

<span data-ttu-id="a1135-267">如果我们对[固定大小的缓冲区](https://github.com/dotnet/csharplang/blob/master/proposals/fixed-sized-buffers.md)提出限制，因为它允许 `Span<T>` 的实例甚至更长，则此功能将变得更加引人注目。</span><span class="sxs-lookup"><span data-stu-id="a1135-267">This feature gets more compelling if we lift the restrictions on [fixed sized buffers](https://github.com/dotnet/csharplang/blob/master/proposals/fixed-sized-buffers.md) as it would allow for `Span<T>` instances of even greater length.</span></span> 

<span data-ttu-id="a1135-268">如果需要关闭此路径，则该语言可通过确保此类 `Span<T>` 实例仅向下来满足此要求。</span><span class="sxs-lookup"><span data-stu-id="a1135-268">If there is ever a need to go down this path then the language could accommodate this by ensuring such `Span<T>` instances were downward facing only.</span></span> <span data-ttu-id="a1135-269">这就是，它们只是对创建它们的范围而言是*安全*的。</span><span class="sxs-lookup"><span data-stu-id="a1135-269">That is they were only ever *safe-to-escape* to the scope in which they were created.</span></span> <span data-ttu-id="a1135-270">这确保了语言决不需要考虑 `ref` 值：通过 `ref struct`的 `ref struct` 返回或字段来转义方法。</span><span class="sxs-lookup"><span data-stu-id="a1135-270">This ensure the language never had to consider a `ref` value escaping a method via a `ref struct` return or field of `ref struct`.</span></span> <span data-ttu-id="a1135-271">这可能还需要进一步的更改以通过这种方式捕获 `ref` 参数。</span><span class="sxs-lookup"><span data-stu-id="a1135-271">This would likely also require further changes to recognize such constructors as capturing a `ref` parameter in this way though.</span></span>
