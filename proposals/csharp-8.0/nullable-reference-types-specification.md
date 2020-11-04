---
ms.openlocfilehash: 39291abb97f11662520467fbe0e3430ea2da621d
ms.sourcegitcommit: 29df547564c4ffc51b1dedf8369dc91e8f0ba854
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/04/2020
ms.locfileid: "93344341"
---
# <a name="nullable-reference-types-specification"></a><span data-ttu-id="48462-101">可以为 null 的引用类型规范</span><span class="sxs-lookup"><span data-stu-id="48462-101">Nullable Reference Types Specification</span></span>

<span data-ttu-id="48462-102">这是一个正在进行的工作-缺少几个部件或这些部件不完整。</span><span class="sxs-lookup"><span data-stu-id="48462-102">\*\*\*This is a work in progress - several parts are missing or incomplete.</span></span> <span data-ttu-id="48462-103">可在 c # 9 文件夹中找到此文档的更新版本。</span><span class="sxs-lookup"><span data-stu-id="48462-103">An updated version of this document can be found in the C# 9 folder.</span></span> <span data-ttu-id="48462-104">\*\*_</span><span class="sxs-lookup"><span data-stu-id="48462-104">\*\*_</span></span>

## <a name="syntax"></a><span data-ttu-id="48462-105">语法</span><span class="sxs-lookup"><span data-stu-id="48462-105">Syntax</span></span>

### <a name="nullable-reference-types"></a><span data-ttu-id="48462-106">可为空引用类型</span><span class="sxs-lookup"><span data-stu-id="48462-106">Nullable reference types</span></span>

<span data-ttu-id="48462-107">可以为 null 的引用类型与 "可以为 null 的值类型" 的缩写形式具有相同的语法 `T?` ，但没有相应的长格式。</span><span class="sxs-lookup"><span data-stu-id="48462-107">Nullable reference types have the same syntax `T?` as the short form of nullable value types, but do not have a corresponding long form.</span></span>

<span data-ttu-id="48462-108">出于规范的目的，当前 `nullable_type` 生产将重命名为 `nullable_value_type` ，并 `nullable_reference_type` 添加生产：</span><span class="sxs-lookup"><span data-stu-id="48462-108">For the purposes of the specification, the current `nullable_type` production is renamed to `nullable_value_type`, and a `nullable_reference_type` production is added:</span></span>

```antlr
reference_type
    : ...
    | nullable_reference_type
    ;
    
nullable_reference_type
    : non_nullable_reference_type '?'
    ;
    
non_nullable_reference_type
    : type
    ;
```

<span data-ttu-id="48462-109">`non_nullable_reference_type`在中， `nullable_reference_type` 必须是不可以为 null 的引用类型 (类、接口、委托或数组) ，或者是一个被约束为不可为 null 的引用类型的类型参数， (通过 `class` 约束或除) 以外的类 `object` 。</span><span class="sxs-lookup"><span data-stu-id="48462-109">The `non_nullable_reference_type` in a `nullable_reference_type` must be a non-nullable reference type (class, interface, delegate or array), or a type parameter that is constrained to be a non-nullable reference type (through the `class` constraint, or a class other than `object`).</span></span>

<span data-ttu-id="48462-110">可以为 null 的引用类型不能出现在以下位置：</span><span class="sxs-lookup"><span data-stu-id="48462-110">Nullable reference types cannot occur in the following positions:</span></span>

- <span data-ttu-id="48462-111">作为基类或接口</span><span class="sxs-lookup"><span data-stu-id="48462-111">as a base class or interface</span></span>
- <span data-ttu-id="48462-112">作为的接收方 `member_access`</span><span class="sxs-lookup"><span data-stu-id="48462-112">as the receiver of a `member_access`</span></span>
- <span data-ttu-id="48462-113">作为 `type` 中的 `object_creation_expression`</span><span class="sxs-lookup"><span data-stu-id="48462-113">as the `type` in an `object_creation_expression`</span></span>
- <span data-ttu-id="48462-114">作为 `delegate_type` 中的 `delegate_creation_expression`</span><span class="sxs-lookup"><span data-stu-id="48462-114">as the `delegate_type` in a `delegate_creation_expression`</span></span>
- <span data-ttu-id="48462-115">作为 `type` 中的 `is_expression` ， `catch_clause` 或 `type_pattern`</span><span class="sxs-lookup"><span data-stu-id="48462-115">as the `type` in an `is_expression`, a `catch_clause` or a `type_pattern`</span></span>
- <span data-ttu-id="48462-116">作为 `interface` 完全限定的接口成员名称中的</span><span class="sxs-lookup"><span data-stu-id="48462-116">as the `interface` in a fully qualified interface member name</span></span>

<span data-ttu-id="48462-117">在 `nullable_reference_type` 禁用了可为 null 的注释上下文的位置上，将发出警告。</span><span class="sxs-lookup"><span data-stu-id="48462-117">A warning is given on a `nullable_reference_type` where the nullable annotation context is disabled.</span></span>

### <a name="nullable-class-constraint"></a><span data-ttu-id="48462-118">可以为 null 的类约束</span><span class="sxs-lookup"><span data-stu-id="48462-118">Nullable class constraint</span></span>

<span data-ttu-id="48462-119">`class`约束具有可为 null 的对应项 `class?` ：</span><span class="sxs-lookup"><span data-stu-id="48462-119">The `class` constraint has a nullable counterpart `class?`:</span></span>

```antlr
primary_constraint
    : ...
    | 'class' '?'
    ;
```

### <a name="the-null-forgiving-operator"></a><span data-ttu-id="48462-120">包容性运算符</span><span class="sxs-lookup"><span data-stu-id="48462-120">The null-forgiving operator</span></span>

<span data-ttu-id="48462-121">后修复后的 `!` 运算符称为包容性运算符。</span><span class="sxs-lookup"><span data-stu-id="48462-121">The post-fix `!` operator is called the null-forgiving operator.</span></span>

```antlr
primary_expression
    : ...
    | null_forgiving_expression
    ;
    
null_forgiving_expression
    : primary_expression '!'
    ;
```

<span data-ttu-id="48462-122">`primary_expression`必须为引用类型。</span><span class="sxs-lookup"><span data-stu-id="48462-122">The `primary_expression` must be of a reference type.</span></span>  

<span data-ttu-id="48462-123">后缀 `!` 运算符没有运行时效果，它的计算结果为基础表达式的结果。</span><span class="sxs-lookup"><span data-stu-id="48462-123">The postfix `!` operator has no runtime effect - it evaluates to the result of the underlying expression.</span></span> <span data-ttu-id="48462-124">其唯一的作用是更改表达式的 null 状态，并限制其使用时给出的警告。</span><span class="sxs-lookup"><span data-stu-id="48462-124">Its only role is to change the null state of the expression, and to limit warnings given on its use.</span></span>

### <a name="nullable-implicitly-typed-local-variables"></a><span data-ttu-id="48462-125">可以为 null 的隐式类型的局部变量</span><span class="sxs-lookup"><span data-stu-id="48462-125">nullable implicitly typed local variables</span></span>

<span data-ttu-id="48462-126">`var` 推导引用类型的批注类型。</span><span class="sxs-lookup"><span data-stu-id="48462-126">`var` infers an annotated type for reference types.</span></span>
<span data-ttu-id="48462-127">例如，在中 `var s = "";` ， `var` 将推断为 `string?` 。</span><span class="sxs-lookup"><span data-stu-id="48462-127">For instance, in `var s = "";` the `var` is inferred as `string?`.</span></span>

### <a name="nullable-compiler-directives"></a><span data-ttu-id="48462-128">可以为 null 的编译器指令</span><span class="sxs-lookup"><span data-stu-id="48462-128">Nullable compiler directives</span></span>

<span data-ttu-id="48462-129">`#nullable` 指令控制可为 null 的批注和警告上下文。</span><span class="sxs-lookup"><span data-stu-id="48462-129">`#nullable` directives control the nullable annotation and warning contexts.</span></span>

```antlr
pp_directive
    : ...
    | pp_nullable
    ;
    
pp_nullable
    : whitespace? '#' whitespace? 'nullable' whitespace nullable_action pp_new_line
    ;
    
nullable_action
    : 'disable'
    | 'enable'
    | 'restore'
    ;
```

<span data-ttu-id="48462-130">`#pragma warning` 指令将展开以允许更改可为 null 的警告上下文，并允许在默认情况下启用单个警告（即使它们在默认情况下处于禁用状态）：</span><span class="sxs-lookup"><span data-stu-id="48462-130">`#pragma warning` directives are expanded to allow changing the nullable warning context, and to allow individual warnings to be enabled on even when they're disabled by default:</span></span>

```antlr
pragma_warning_body
    : ...
    | 'warning' whitespace nullable_action whitespace 'nullable'
    ;

warning_action
    : ...
    | 'enable'
    ;
```

<span data-ttu-id="48462-131">请注意，的新形式 `pragma_warning_body` 使用 `nullable_action` ，而不是 `warning_action` 。</span><span class="sxs-lookup"><span data-stu-id="48462-131">Note that the new form of `pragma_warning_body` uses `nullable_action`, not `warning_action`.</span></span>

## <a name="nullable-contexts"></a><span data-ttu-id="48462-132">可为空上下文</span><span class="sxs-lookup"><span data-stu-id="48462-132">Nullable contexts</span></span>

<span data-ttu-id="48462-133">源代码的每一行都有一个 _nullable 注释上下文 \* 和一个 *可以为 null 的警告上下文* 。</span><span class="sxs-lookup"><span data-stu-id="48462-133">Every line of source code has a _nullable annotation context\* and a *nullable warning context*.</span></span> <span data-ttu-id="48462-134">这些控件控制是否可为 null 的批注是否有效，以及是否给定了可为空性警告。</span><span class="sxs-lookup"><span data-stu-id="48462-134">These control whether nullable annotations have effect, and whether nullability warnings are given.</span></span> <span data-ttu-id="48462-135">给定行的批注上下文已 *禁用* 或 *启用* 。</span><span class="sxs-lookup"><span data-stu-id="48462-135">The annotation context of a given line is either *disabled* or *enabled*.</span></span> <span data-ttu-id="48462-136">给定行的警告上下文已 *禁用* 或 *启用* 。</span><span class="sxs-lookup"><span data-stu-id="48462-136">The warning context of a given line is either *disabled* or *enabled*.</span></span>

<span data-ttu-id="48462-137">既可在 c # 源代码) 之外的项目级别指定这两个上下文，也可通过预处理器指令在源文件中的任何位置指定 (`#nullable` 。</span><span class="sxs-lookup"><span data-stu-id="48462-137">Both contexts can be specified at the project level (outside of C# source code), or anywhere within a source file via `#nullable` pre-processor directives.</span></span> <span data-ttu-id="48462-138">如果未提供任何项目级别设置，则默认情况下，这两个上下文都是 *禁用* 的。</span><span class="sxs-lookup"><span data-stu-id="48462-138">If no project level settings are provided the default is for both contexts to be *disabled*.</span></span>

<span data-ttu-id="48462-139">`#nullable`指令控制源文本中的批注和警告上下文，并优先于项目级设置。</span><span class="sxs-lookup"><span data-stu-id="48462-139">The `#nullable` directive controls the annotation and warning contexts within the source text, and take precedence over the project-level settings.</span></span>

<span data-ttu-id="48462-140">指令设置上下文 () 它控制后续代码行，直到另一个指令重写它，或直到源文件的结尾。</span><span class="sxs-lookup"><span data-stu-id="48462-140">A directive sets the context(s) it controls for subsequent lines of code, until another directive overrides it, or until the end of the source file.</span></span>

<span data-ttu-id="48462-141">指令的效果如下所示：</span><span class="sxs-lookup"><span data-stu-id="48462-141">The effect of the directives is as follows:</span></span>

- <span data-ttu-id="48462-142">`#nullable disable`：将可为 null 的批注和警告上下文设置为 *已禁用*</span><span class="sxs-lookup"><span data-stu-id="48462-142">`#nullable disable`: Sets the nullable annotation and warning contexts to *disabled*</span></span>
- <span data-ttu-id="48462-143">`#nullable enable`：将可为 null 的批注和警告上下文设置为 *enabled*</span><span class="sxs-lookup"><span data-stu-id="48462-143">`#nullable enable`: Sets the nullable annotation and warning contexts to *enabled*</span></span>
- <span data-ttu-id="48462-144">`#nullable restore`：将可为 null 的批注和警告上下文还原到项目设置</span><span class="sxs-lookup"><span data-stu-id="48462-144">`#nullable restore`: Restores the nullable annotation and warning contexts to project settings</span></span>
- <span data-ttu-id="48462-145">`#nullable disable annotations`：将可为 null 的注释上下文设置为 *已禁用*</span><span class="sxs-lookup"><span data-stu-id="48462-145">`#nullable disable annotations`: Sets the nullable annotation context to *disabled*</span></span>
- <span data-ttu-id="48462-146">`#nullable enable annotations`：将可为 null 的注释上下文设置为 *enabled*</span><span class="sxs-lookup"><span data-stu-id="48462-146">`#nullable enable annotations`: Sets the nullable annotation context to *enabled*</span></span>
- <span data-ttu-id="48462-147">`#nullable restore annotations`：将可为 null 的注释上下文还原到项目设置</span><span class="sxs-lookup"><span data-stu-id="48462-147">`#nullable restore annotations`: Restores the nullable annotation context to project settings</span></span>
- <span data-ttu-id="48462-148">`#nullable disable warnings`：将可为 null 的警告上下文设置为 *已禁用*</span><span class="sxs-lookup"><span data-stu-id="48462-148">`#nullable disable warnings`: Sets the nullable warning context to *disabled*</span></span>
- <span data-ttu-id="48462-149">`#nullable enable warnings`：将可为 null 的警告上下文设置为 *已启用*</span><span class="sxs-lookup"><span data-stu-id="48462-149">`#nullable enable warnings`: Sets the nullable warning context to *enabled*</span></span>
- <span data-ttu-id="48462-150">`#nullable restore warnings`：将可为 null 的警告上下文还原到项目设置</span><span class="sxs-lookup"><span data-stu-id="48462-150">`#nullable restore warnings`: Restores the nullable warning context to project settings</span></span>

## <a name="nullability-of-types"></a><span data-ttu-id="48462-151">类型为 Null 性</span><span class="sxs-lookup"><span data-stu-id="48462-151">Nullability of types</span></span>

<span data-ttu-id="48462-152">给定的类型可以有以下四个 nullabilities 之一： *在意* 、 *不可 null* 、 *nullable* 和 *unknown* 。</span><span class="sxs-lookup"><span data-stu-id="48462-152">A given type can have one of four nullabilities: *Oblivious* , *nonnullable* , *nullable* and *unknown*.</span></span> 

<span data-ttu-id="48462-153">如果将可能的值分配给 *不可 null* 和 *未知* 类型，则可能会引发警告 `null` 。</span><span class="sxs-lookup"><span data-stu-id="48462-153">*Nonnullable* and *unknown* types may cause warnings if a potential `null` value is assigned to them.</span></span> <span data-ttu-id="48462-154">但是， *在意* 和可以 *为* null 的类型为 "可 *赋值* "，可以 `null` 为其分配值而不会出现警告。</span><span class="sxs-lookup"><span data-stu-id="48462-154">*Oblivious* and *nullable* types, however, are " *null-assignable* " and can have `null` values assigned to them without warnings.</span></span> 

<span data-ttu-id="48462-155">可以取消引用或分配 *在意* 和 *不可 null* 类型，而不会出现警告。</span><span class="sxs-lookup"><span data-stu-id="48462-155">*Oblivious* and *nonnullable* types can be dereferenced or assigned without warnings.</span></span> <span data-ttu-id="48462-156">然而， *可以为* null 的类型和 *未知* 类型的值都是 " *空* 值"，当取消引用或赋值时，如果没有正确的 null 检查，则可能会导致警告。</span><span class="sxs-lookup"><span data-stu-id="48462-156">Values of *nullable* and *unknown* types, however, are " *null-yielding* " and may cause warnings when dereferenced or assigned without proper null checking.</span></span> 

<span data-ttu-id="48462-157">Null 生成类型的 *默认 null 状态* 为 "可能为 null"。</span><span class="sxs-lookup"><span data-stu-id="48462-157">The *default null state* of a null-yielding type is "maybe null".</span></span> <span data-ttu-id="48462-158">非 null 生成类型的默认 null 状态为 "not null"。</span><span class="sxs-lookup"><span data-stu-id="48462-158">The default null state of a non-null-yielding type is "not null".</span></span>

<span data-ttu-id="48462-159">类型的类型和在确定其为 null 性时所发生的可为 null 的注释上下文：</span><span class="sxs-lookup"><span data-stu-id="48462-159">The kind of type and the nullable annotation context it occurs in determine its nullability:</span></span>

- <span data-ttu-id="48462-160">不可 null 值类型 `S` 始终为 *不可 null*</span><span class="sxs-lookup"><span data-stu-id="48462-160">A nonnullable value type `S` is always *nonnullable*</span></span>
- <span data-ttu-id="48462-161">可以为 null 的值类型 `S?` 始终 *可以为 null*</span><span class="sxs-lookup"><span data-stu-id="48462-161">A nullable value type `S?` is always *nullable*</span></span>
- <span data-ttu-id="48462-162">`C`*禁用* 的注释上下文中的批注引用类型为 *在意*</span><span class="sxs-lookup"><span data-stu-id="48462-162">An unannotated reference type `C` in a *disabled* annotation context is *oblivious*</span></span>
- <span data-ttu-id="48462-163">`C`*启用* 的批注上下文中的批注引用类型为 *不可 null*</span><span class="sxs-lookup"><span data-stu-id="48462-163">An unannotated reference type `C` in an *enabled* annotation context is *nonnullable*</span></span>
- <span data-ttu-id="48462-164">可以为 null 的引用类型可以 `C?` *为 null* (但在 *禁用* 的批注上下文中可能生成警告) </span><span class="sxs-lookup"><span data-stu-id="48462-164">A nullable reference type `C?` is *nullable* (but a warning may be yielded in a *disabled* annotation context)</span></span>

<span data-ttu-id="48462-165">类型参数还会考虑它们的约束：</span><span class="sxs-lookup"><span data-stu-id="48462-165">Type parameters additionally take their constraints into account:</span></span>

- <span data-ttu-id="48462-166">一个类型参数 `T` ，其中所有约束都 (如果任何) 为 null 生成类型 ( *可以为* null 且 *未知* ) 或 `class?` 约束 *未知*</span><span class="sxs-lookup"><span data-stu-id="48462-166">A type parameter `T` where all constraints (if any) are either null-yielding types ( *nullable* and *unknown* ) or the `class?` constraint is *unknown*</span></span>
- <span data-ttu-id="48462-167">一个类型参数 `T` ，其中至少有一个约束为 *在意* 或 *不可 null* ，或者其中一个 `struct` 或 `class` 约束为</span><span class="sxs-lookup"><span data-stu-id="48462-167">A type parameter `T` where at least one constraint is either *oblivious* or *nonnullable* or one of the `struct` or `class` constraints is</span></span>
    - <span data-ttu-id="48462-168">*禁用* 的注释上下文中的 *在意*</span><span class="sxs-lookup"><span data-stu-id="48462-168">*oblivious* in a *disabled* annotation context</span></span>
    - <span data-ttu-id="48462-169">*已启用* 的批注上下文中的 *不可 null*</span><span class="sxs-lookup"><span data-stu-id="48462-169">*nonnullable* in an *enabled* annotation context</span></span>
- <span data-ttu-id="48462-170">一个可以为 null 的类型参数， `T?` 其中至少有一个 `T` 约束是 *在意* 或 *不可 null* ，或者是 `struct` 或 `class` 约束之一，是</span><span class="sxs-lookup"><span data-stu-id="48462-170">A nullable type parameter `T?` where at least one of `T`'s constraints is *oblivious* or *nonnullable* or one of the `struct` or `class` constraints, is</span></span>
    - <span data-ttu-id="48462-171">在 *禁用* 的批注上下文中 *可以为 null* (但会生成警告) </span><span class="sxs-lookup"><span data-stu-id="48462-171">*nullable* in a *disabled* annotation context (but a warning is yielded)</span></span>
    - <span data-ttu-id="48462-172">在 *启用* 的批注上下文中 *为 null*</span><span class="sxs-lookup"><span data-stu-id="48462-172">*nullable* in an *enabled* annotation context</span></span>

<span data-ttu-id="48462-173">对于类型参数 `T` ， `T?` 仅当 `T` 已知为值类型或已知为引用类型时，才允许。</span><span class="sxs-lookup"><span data-stu-id="48462-173">For a type parameter `T`, `T?` is only allowed if `T` is known to be a value type or known to be a reference type.</span></span>

### <a name="nested-functions"></a><span data-ttu-id="48462-174">嵌套函数</span><span class="sxs-lookup"><span data-stu-id="48462-174">Nested functions</span></span>

<span data-ttu-id="48462-175">嵌套函数 (lambda 和局部函数) 被视为方法，但其捕获变量除外。</span><span class="sxs-lookup"><span data-stu-id="48462-175">Nested functions (lambdas and local functions) are treated like methods, except in regards to their captured variables.</span></span>
<span data-ttu-id="48462-176">Lambda 或本地函数中捕获的变量的默认状态是该嵌套函数的所有 "使用" 中的变量的可以为 null 的状态的交集。</span><span class="sxs-lookup"><span data-stu-id="48462-176">The default state of a captured variable inside a lambda or local function is the intersection of the nullable state of the variable at all the "uses" of that nested function.</span></span> <span data-ttu-id="48462-177">函数的使用是对该函数的调用或将其转换为委托的位置。</span><span class="sxs-lookup"><span data-stu-id="48462-177">A use of a function is either a call to that function, or where it is converted to a delegate.</span></span>

### <a name="oblivious-vs-nonnullable"></a><span data-ttu-id="48462-178">在意 vs 不可 null</span><span class="sxs-lookup"><span data-stu-id="48462-178">Oblivious vs nonnullable</span></span>

<span data-ttu-id="48462-179">`type`当类型的最后一个标记在该上下文中时，将被视为在给定的批注上下文中发生。</span><span class="sxs-lookup"><span data-stu-id="48462-179">A `type` is deemed to occur in a given annotation context when the last token of the type is within that context.</span></span>

<span data-ttu-id="48462-180">源代码中的给定引用类型 `C` 是解释为在意还是不可 null 依赖于源代码的注释上下文。</span><span class="sxs-lookup"><span data-stu-id="48462-180">Whether a given reference type `C` in source code is interpreted as oblivious or nonnullable depends on the annotation context of that source code.</span></span> <span data-ttu-id="48462-181">但一旦建立，就会将其视为该类型的一部分，并在替换泛型类型参数的过程中将其视为 "随之一起移动"。</span><span class="sxs-lookup"><span data-stu-id="48462-181">But once established, it is considered part of that type, and "travels with it" e.g. during substitution of generic type arguments.</span></span> <span data-ttu-id="48462-182">这就像在类型上有一个批注 `?` ，但不可见。</span><span class="sxs-lookup"><span data-stu-id="48462-182">It is as if there is an annotation like `?` on the type, but invisible.</span></span>

## <a name="constraints"></a><span data-ttu-id="48462-183">约束</span><span class="sxs-lookup"><span data-stu-id="48462-183">Constraints</span></span>

<span data-ttu-id="48462-184">可以为 null 的引用类型可用作泛型约束。</span><span class="sxs-lookup"><span data-stu-id="48462-184">Nullable reference types can be used as generic constraints.</span></span> <span data-ttu-id="48462-185">而且 `object` 现在作为显式约束有效。</span><span class="sxs-lookup"><span data-stu-id="48462-185">Furthermore `object` is now valid as an explicit constraint.</span></span> <span data-ttu-id="48462-186">缺少约束现在等效于 `object?` 约束 (而不是 `object`) ，但 (与之前不同， `object`) `object?` 不会被视为显式约束。</span><span class="sxs-lookup"><span data-stu-id="48462-186">Absence of a constraint is now equivalent to an `object?` constraint (instead of `object`), but (unlike `object` before) `object?` is not prohibited as an explicit constraint.</span></span>

<span data-ttu-id="48462-187">`class?` 表示 "可能可以为 null 的引用类型" 的新约束，而 `class` 表示 "不可 null 引用类型"。</span><span class="sxs-lookup"><span data-stu-id="48462-187">`class?` is a new constraint denoting "possibly nullable reference type", whereas `class` denotes "nonnullable reference type".</span></span>

<span data-ttu-id="48462-188">类型参数或约束的为空性并不会影响类型是否满足约束，但目前已 (可以为 null 的值类型不满足约束) 的情况除外 `struct` 。</span><span class="sxs-lookup"><span data-stu-id="48462-188">The nullability of a type argument or of a constraint does not impact whether the type satisfies the constraint, except where that is already the case today (nullable value types do not satisfy the `struct` constraint).</span></span> <span data-ttu-id="48462-189">但是，如果类型参数不满足约束的为 null 性要求，则可以提供警告。</span><span class="sxs-lookup"><span data-stu-id="48462-189">However, if the type argument does not satisfy the nullability requirements of the constraint, a warning may be given.</span></span>

## <a name="null-state-and-null-tracking"></a><span data-ttu-id="48462-190">Null 状态和 null 跟踪</span><span class="sxs-lookup"><span data-stu-id="48462-190">Null state and null tracking</span></span>

<span data-ttu-id="48462-191">给定源位置中的每个表达式都具有 *null 状态* ，指示其是否被认为可能计算为 null。</span><span class="sxs-lookup"><span data-stu-id="48462-191">Every expression in a given source location has a *null state* , which indicated whether it is believed to potentially evaluate to null.</span></span> <span data-ttu-id="48462-192">Null 状态为 "not null" 或 "可能为 null"。</span><span class="sxs-lookup"><span data-stu-id="48462-192">The null state is either "not null" or "maybe null".</span></span> <span data-ttu-id="48462-193">Null 状态用于确定是否应为不安全的转换和取消引用提供警告。</span><span class="sxs-lookup"><span data-stu-id="48462-193">The null state is used to determine whether a warning should be given about null-unsafe conversions and dereferences.</span></span>

### <a name="null-tracking-for-variables"></a><span data-ttu-id="48462-194">变量的 Null 跟踪</span><span class="sxs-lookup"><span data-stu-id="48462-194">Null tracking for variables</span></span>

<span data-ttu-id="48462-195">对于指定变量或属性的某些表达式，将根据对它们的赋值、对它们执行的测试以及它们之间的控制流，在发生之间跟踪 null 状态。</span><span class="sxs-lookup"><span data-stu-id="48462-195">For certain expressions denoting variables or properties, the null state is tracked between occurrences, based on assignments to them, tests performed on them and the control flow between them.</span></span> <span data-ttu-id="48462-196">这类似于为变量跟踪明确赋值的方式。</span><span class="sxs-lookup"><span data-stu-id="48462-196">This is similar to how definite assignment is tracked for variables.</span></span> <span data-ttu-id="48462-197">跟踪的表达式如下所示：</span><span class="sxs-lookup"><span data-stu-id="48462-197">The tracked expressions are the ones of the following form:</span></span>

```antlr
tracked_expression
    : simple_name
    | this
    | base
    | tracked_expression '.' identifier
    ;
```

<span data-ttu-id="48462-198">其中，标识符表示字段或属性。</span><span class="sxs-lookup"><span data-stu-id="48462-198">Where the identifiers denote fields or properties.</span></span>

<span data-ttu-id="48462-199">\***描述类似于明确赋值的空状态转换** _</span><span class="sxs-lookup"><span data-stu-id="48462-199">\***Describe null state transitions similar to definite assignment** _</span></span>

### <a name="null-state-for-expressions"></a><span data-ttu-id="48462-200">表达式为 Null</span><span class="sxs-lookup"><span data-stu-id="48462-200">Null state for expressions</span></span>

<span data-ttu-id="48462-201">表达式的 null 状态派生自其形式和类型以及它所涉及的变量的 null 状态。</span><span class="sxs-lookup"><span data-stu-id="48462-201">The null state of an expression is derived from its form and type, and from the null state of variables involved in it.</span></span>

### <a name="literals"></a><span data-ttu-id="48462-202">文本</span><span class="sxs-lookup"><span data-stu-id="48462-202">Literals</span></span>

<span data-ttu-id="48462-203">文本的 null 状态 `null` 为 "可能为 null"。</span><span class="sxs-lookup"><span data-stu-id="48462-203">The null state of a `null` literal is "maybe null".</span></span> <span data-ttu-id="48462-204">`default`正在转换为已知不是不可 null 值类型的类型的文本的 null 状态为 "可能为 null"。</span><span class="sxs-lookup"><span data-stu-id="48462-204">The null state of a `default` literal that is being converted to a type that is known not to be a nonnullable value type is "maybe null".</span></span> <span data-ttu-id="48462-205">任何其他文本的 null 状态为 "not null"。</span><span class="sxs-lookup"><span data-stu-id="48462-205">The null state of any other literal is "not null".</span></span>

### <a name="simple-names"></a><span data-ttu-id="48462-206">简单名称</span><span class="sxs-lookup"><span data-stu-id="48462-206">Simple names</span></span>

<span data-ttu-id="48462-207">如果 `simple_name` 未归类为值，则其 null 状态为 "not null"。</span><span class="sxs-lookup"><span data-stu-id="48462-207">If a `simple_name` is not classified as a value, its null state is "not null".</span></span> <span data-ttu-id="48462-208">否则为跟踪的表达式，其 null 状态将为其在此源位置跟踪的 null 状态。</span><span class="sxs-lookup"><span data-stu-id="48462-208">Otherwise it is a tracked expression, and its null state is its tracked null state at this source location.</span></span>

### <a name="member-access"></a><span data-ttu-id="48462-209">成员访问</span><span class="sxs-lookup"><span data-stu-id="48462-209">Member access</span></span>

<span data-ttu-id="48462-210">如果 `member_access` 未归类为值，则其 null 状态为 "not null"。</span><span class="sxs-lookup"><span data-stu-id="48462-210">If a `member_access` is not classified as a value, its null state is "not null".</span></span> <span data-ttu-id="48462-211">否则，如果是跟踪的表达式，则其 null 状态将为其在此源位置跟踪的 null 状态。</span><span class="sxs-lookup"><span data-stu-id="48462-211">Otherwise, if it is a tracked expression, its null state is its tracked null state at this source location.</span></span> <span data-ttu-id="48462-212">否则，其 null 状态为其类型的默认 null 状态。</span><span class="sxs-lookup"><span data-stu-id="48462-212">Otherwise its null state is the default null state of its type.</span></span>

### <a name="invocation-expressions"></a><span data-ttu-id="48462-213">调用表达式</span><span class="sxs-lookup"><span data-stu-id="48462-213">Invocation expressions</span></span>

<span data-ttu-id="48462-214">如果 `invocation_expression` 调用使用一个或多个特性声明的成员以实现特殊的 null 行为，则 null 状态由这些特性决定。</span><span class="sxs-lookup"><span data-stu-id="48462-214">If an `invocation_expression` invokes a member that is declared with one or more attributes for special null behavior, the null state is determined by those attributes.</span></span> <span data-ttu-id="48462-215">否则，表达式的 null 状态为其类型的默认 null 状态。</span><span class="sxs-lookup"><span data-stu-id="48462-215">Otherwise the null state of the expression is the default null state of its type.</span></span>

### <a name="element-access"></a><span data-ttu-id="48462-216">元素访问</span><span class="sxs-lookup"><span data-stu-id="48462-216">Element access</span></span>

<span data-ttu-id="48462-217">如果 `element_access` 调用使用一个或多个特性声明的索引器来实现特殊的 null 行为，则 null 状态由这些特性决定。</span><span class="sxs-lookup"><span data-stu-id="48462-217">If an `element_access` invokes an indexer that is declared with one or more attributes for special null behavior, the null state is determined by those attributes.</span></span> <span data-ttu-id="48462-218">否则，表达式的 null 状态为其类型的默认 null 状态。</span><span class="sxs-lookup"><span data-stu-id="48462-218">Otherwise the null state of the expression is the default null state of its type.</span></span>

### <a name="base-access"></a><span data-ttu-id="48462-219">基本访问权限</span><span class="sxs-lookup"><span data-stu-id="48462-219">Base access</span></span>

<span data-ttu-id="48462-220">如果 `B` 表示封闭类型的基类型， `base.I` 则具有与相同的 null 状态， `((B)this).I` 并且与 `base[E]` 具有相同的 null 状态 `((B)this)[E]` 。</span><span class="sxs-lookup"><span data-stu-id="48462-220">If `B` denotes the base type of the enclosing type, `base.I` has the same null state as `((B)this).I` and `base[E]` has the same null state as `((B)this)[E]`.</span></span>

### <a name="default-expressions"></a><span data-ttu-id="48462-221">默认表达式</span><span class="sxs-lookup"><span data-stu-id="48462-221">Default expressions</span></span>

<span data-ttu-id="48462-222">`default(T)` 如果 `T` 已知为不可 null 值类型，则具有 null 状态 "非 null"。</span><span class="sxs-lookup"><span data-stu-id="48462-222">`default(T)` has the null state "non-null" if `T` is known to be a nonnullable value type.</span></span> <span data-ttu-id="48462-223">否则，它的状态为 "可能为 null"。</span><span class="sxs-lookup"><span data-stu-id="48462-223">Otherwise it has the null state "maybe null".</span></span>

### <a name="null-conditional-expressions"></a><span data-ttu-id="48462-224">Null 条件表达式</span><span class="sxs-lookup"><span data-stu-id="48462-224">Null-conditional expressions</span></span>

<span data-ttu-id="48462-225">`null_conditional_expression`具有 null 状态 "可能为 null"。</span><span class="sxs-lookup"><span data-stu-id="48462-225">A `null_conditional_expression` has the null state "maybe null".</span></span>

### <a name="cast-expressions"></a><span data-ttu-id="48462-226">强制转换表达式</span><span class="sxs-lookup"><span data-stu-id="48462-226">Cast expressions</span></span>

<span data-ttu-id="48462-227">如果强制转换表达式 `(T)E` 调用用户定义的转换，则表达式的 null 状态为其类型的默认 null 状态。</span><span class="sxs-lookup"><span data-stu-id="48462-227">If a cast expression `(T)E` invokes a user-defined conversion, then the null state of the expression is the default null state for its type.</span></span> <span data-ttu-id="48462-228">否则，如果 `T` 为 null 生成 (_nullable \* 或 *未知* ) ，则 null 状态为 "可能为 null"。</span><span class="sxs-lookup"><span data-stu-id="48462-228">Otherwise, if `T` is null-yielding (_nullable\* or *unknown* ) then the null state is "maybe null".</span></span> <span data-ttu-id="48462-229">否则，null 状态与的 null 状态相同 `E` 。</span><span class="sxs-lookup"><span data-stu-id="48462-229">Otherwise the null state is the same as the null state of `E`.</span></span>

### <a name="await-expressions"></a><span data-ttu-id="48462-230">Await 表达式</span><span class="sxs-lookup"><span data-stu-id="48462-230">Await expressions</span></span>

<span data-ttu-id="48462-231">的 null 状态 `await E` 为其类型的默认 null 状态。</span><span class="sxs-lookup"><span data-stu-id="48462-231">The null state of `await E` is the default null state of its type.</span></span>

### <a name="the-as-operator"></a><span data-ttu-id="48462-232">`as` 运算符</span><span class="sxs-lookup"><span data-stu-id="48462-232">The `as` operator</span></span>

<span data-ttu-id="48462-233">`as`表达式的状态为 "可能为 null"。</span><span class="sxs-lookup"><span data-stu-id="48462-233">An `as` expression has the null state "maybe null".</span></span>

### <a name="the-null-coalescing-operator"></a><span data-ttu-id="48462-234">Null 合并运算符</span><span class="sxs-lookup"><span data-stu-id="48462-234">The null-coalescing operator</span></span>

<span data-ttu-id="48462-235">`E1 ?? E2` 具有与相同的空状态 `E2`</span><span class="sxs-lookup"><span data-stu-id="48462-235">`E1 ?? E2` has the same null state as `E2`</span></span>

### <a name="the-conditional-operator"></a><span data-ttu-id="48462-236">条件运算符</span><span class="sxs-lookup"><span data-stu-id="48462-236">The conditional operator</span></span>

<span data-ttu-id="48462-237">`E1 ? E2 : E3`如果和的 null 状态为 `E2` `E3` "not null"，则的 null 状态为 "not null"。</span><span class="sxs-lookup"><span data-stu-id="48462-237">The null state of `E1 ? E2 : E3` is "not null" if the null state of both `E2` and `E3` are "not null".</span></span> <span data-ttu-id="48462-238">否则为 "可能为 null"。</span><span class="sxs-lookup"><span data-stu-id="48462-238">Otherwise it is "maybe null".</span></span>

### <a name="query-expressions"></a><span data-ttu-id="48462-239">查询表达式</span><span class="sxs-lookup"><span data-stu-id="48462-239">Query expressions</span></span>

<span data-ttu-id="48462-240">查询表达式的 null 状态为其类型的默认 null 状态。</span><span class="sxs-lookup"><span data-stu-id="48462-240">The null state of a query expression is the default null state of its type.</span></span>

### <a name="assignment-operators"></a><span data-ttu-id="48462-241">赋值运算符</span><span class="sxs-lookup"><span data-stu-id="48462-241">Assignment operators</span></span>

<span data-ttu-id="48462-242">`E1 = E2` 和 `E1 op= E2` 都具有与 `E2` 应用任何隐式转换相同的空状态。</span><span class="sxs-lookup"><span data-stu-id="48462-242">`E1 = E2` and `E1 op= E2` have the same null state as `E2` after any implicit conversions have been applied.</span></span>

### <a name="unary-and-binary-operators"></a><span data-ttu-id="48462-243">一元运算符和二元运算符</span><span class="sxs-lookup"><span data-stu-id="48462-243">Unary and binary operators</span></span>

<span data-ttu-id="48462-244">如果一元或二元运算符调用了用一个或多个特性声明的用户定义的运算符来实现特殊的 null 行为，则 null 状态由这些特性决定。</span><span class="sxs-lookup"><span data-stu-id="48462-244">If a unary or binary operator invokes an user-defined operator that is declared with one or more attributes for special null behavior, the null state is determined by those attributes.</span></span> <span data-ttu-id="48462-245">否则，表达式的 null 状态为其类型的默认 null 状态。</span><span class="sxs-lookup"><span data-stu-id="48462-245">Otherwise the null state of the expression is the default null state of its type.</span></span>

<span data-ttu-id="48462-246">\***对于 `+` 字符串和委托，二进制文件的特殊** 操作是什么？_</span><span class="sxs-lookup"><span data-stu-id="48462-246">\***Something special to do for binary `+` over strings and delegates?** _</span></span>

### <a name="expressions-that-propagate-null-state"></a><span data-ttu-id="48462-247">传播 null 状态的表达式</span><span class="sxs-lookup"><span data-stu-id="48462-247">Expressions that propagate null state</span></span>

<span data-ttu-id="48462-248">`(E)``checked(E)`和 `unchecked(E)` 都具有与相同的 null 状态 `E` 。</span><span class="sxs-lookup"><span data-stu-id="48462-248">`(E)`, `checked(E)` and `unchecked(E)` all have the same null state as `E`.</span></span>

### <a name="expressions-that-are-never-null"></a><span data-ttu-id="48462-249">从不为 null 的表达式</span><span class="sxs-lookup"><span data-stu-id="48462-249">Expressions that are never null</span></span>

<span data-ttu-id="48462-250">以下表达式格式的 null 状态始终为 "not null"：</span><span class="sxs-lookup"><span data-stu-id="48462-250">The null state of the following expression forms is always "not null":</span></span>

- <span data-ttu-id="48462-251">`this` access</span><span class="sxs-lookup"><span data-stu-id="48462-251">`this` access</span></span>
- <span data-ttu-id="48462-252">内插字符串</span><span class="sxs-lookup"><span data-stu-id="48462-252">interpolated strings</span></span>
- <span data-ttu-id="48462-253">`new` 表达式 (对象、委托、匿名对象和数组创建表达式) </span><span class="sxs-lookup"><span data-stu-id="48462-253">`new` expressions (object, delegate, anonymous object and array creation expressions)</span></span>
- <span data-ttu-id="48462-254">`typeof` 表达式</span><span class="sxs-lookup"><span data-stu-id="48462-254">`typeof` expressions</span></span>
- <span data-ttu-id="48462-255">`nameof` 表达式</span><span class="sxs-lookup"><span data-stu-id="48462-255">`nameof` expressions</span></span>
- <span data-ttu-id="48462-256">匿名函数 (匿名方法和 lambda 表达式) </span><span class="sxs-lookup"><span data-stu-id="48462-256">anonymous functions (anonymous methods and lambda expressions)</span></span>
- <span data-ttu-id="48462-257">包容性表达式</span><span class="sxs-lookup"><span data-stu-id="48462-257">null-forgiving expressions</span></span>
- <span data-ttu-id="48462-258">`is` 表达式</span><span class="sxs-lookup"><span data-stu-id="48462-258">`is` expressions</span></span>

## <a name="type-inference"></a><span data-ttu-id="48462-259">类型推理</span><span class="sxs-lookup"><span data-stu-id="48462-259">Type inference</span></span>

### <a name="type-inference-for-var"></a><span data-ttu-id="48462-260">的类型推理 `var`</span><span class="sxs-lookup"><span data-stu-id="48462-260">Type inference for `var`</span></span>

<span data-ttu-id="48462-261">为声明的局部变量声明的类型 `var` 由初始化表达式的 null 状态通知。</span><span class="sxs-lookup"><span data-stu-id="48462-261">The type inferred for local variables declared with `var` is informed by the null state of the initializing expression.</span></span>

```csharp
var x = E;
```

<span data-ttu-id="48462-262">如果的类型 `E` 是可以为 null 的引用类型 `C?` ，并且的 null 状态 `E` 为 "not null"，则为推断的类型 `x` 为 `C` 。</span><span class="sxs-lookup"><span data-stu-id="48462-262">If the type of `E` is a nullable reference type `C?` and the null state of `E` is "not null" then the type inferred for `x` is `C`.</span></span> <span data-ttu-id="48462-263">否则，推理出的类型是的类型 `E` 。</span><span class="sxs-lookup"><span data-stu-id="48462-263">Otherwise, the inferred type is the type of `E`.</span></span>

<span data-ttu-id="48462-264">为推断的类型的可为 null 性根据 `x` 的注释上下文确定，这一点与该 `var` 位置中显式给定的类型相同。</span><span class="sxs-lookup"><span data-stu-id="48462-264">The nullability of the type inferred for `x` is determined as described above, based on the annotation context of the `var`, just as if the type had been given explicitly in that position.</span></span>

### <a name="type-inference-for-var"></a><span data-ttu-id="48462-265">的类型推理 `var?`</span><span class="sxs-lookup"><span data-stu-id="48462-265">Type inference for `var?`</span></span>

<span data-ttu-id="48462-266">为声明的局部变量的类型与 `var?` 初始化表达式的 null 状态无关。</span><span class="sxs-lookup"><span data-stu-id="48462-266">The type inferred for local variables declared with `var?` is independent of the null state of the initializing expression.</span></span>

```csharp
var? x = E;
```

<span data-ttu-id="48462-267">如果的类型 `T` `E` 是可以为 null 的值类型或可以为 null 的引用类型，则为推断的类型 `x` 为 `T` 。</span><span class="sxs-lookup"><span data-stu-id="48462-267">If the type `T` of `E` is a nullable value type or a nullable reference type then the type inferred for `x` is `T`.</span></span> <span data-ttu-id="48462-268">否则，如果 `T` 是不可 null 值类型，则 `S` 推断出的类型为 `S?` 。</span><span class="sxs-lookup"><span data-stu-id="48462-268">Otherwise, if `T` is a nonnullable value type `S` the type inferred is `S?`.</span></span> <span data-ttu-id="48462-269">否则，如果 `T` 是不可 null 引用类型，则 `C` 推断出的类型为 `C?` 。</span><span class="sxs-lookup"><span data-stu-id="48462-269">Otherwise, if `T` is a nonnullable reference type `C` the type inferred is `C?`.</span></span> <span data-ttu-id="48462-270">否则，声明是非法的。</span><span class="sxs-lookup"><span data-stu-id="48462-270">Otherwise, the declaration is illegal.</span></span>

<span data-ttu-id="48462-271">为推断的类型的为空性 `x` 始终 _nullable \*。</span><span class="sxs-lookup"><span data-stu-id="48462-271">The nullability of the type inferred for `x` is always _nullable\*.</span></span>

### <a name="generic-type-inference"></a><span data-ttu-id="48462-272">泛型类型推理</span><span class="sxs-lookup"><span data-stu-id="48462-272">Generic type inference</span></span>

<span data-ttu-id="48462-273">泛型类型推理经过了增强，可帮助确定推断的引用类型是否可以为 null。</span><span class="sxs-lookup"><span data-stu-id="48462-273">Generic type inference is enhanced to help decide whether inferred reference types should be nullable or not.</span></span> <span data-ttu-id="48462-274">这是一项最大的工作，并且本身不会生成警告，但在将所选重载的推断类型应用于参数时，可能会导致可以为 null 的警告。</span><span class="sxs-lookup"><span data-stu-id="48462-274">This is a best effort, and does not in and of itself yield warnings, but may lead to nullable warnings when the inferred types of the selected overload are applied to the arguments.</span></span>

<span data-ttu-id="48462-275">类型推断不依赖于传入类型的注释上下文。</span><span class="sxs-lookup"><span data-stu-id="48462-275">The type inference does not rely on the annotation context of incoming types.</span></span> <span data-ttu-id="48462-276">相反 `type` ，将从其 "可能已在" 中获取其自己的注释上下文（如果已显式表示）。</span><span class="sxs-lookup"><span data-stu-id="48462-276">Instead a `type` is inferred which acquires its own annotation context from where it "would have been" if it had been expressed explicitly.</span></span> <span data-ttu-id="48462-277">这会将类型推理的角色作为一种简便方式，方便您自己编写的内容。</span><span class="sxs-lookup"><span data-stu-id="48462-277">This underscores the role of type inference as a convenience for what you could have written yourself.</span></span>

<span data-ttu-id="48462-278">更准确地说，推理出的类型实参的注释上下文是标记的上下文，该标记后面应跟 `<...>` 有一个类型形参列表，其中有一个，即要调用的泛型方法的名称。</span><span class="sxs-lookup"><span data-stu-id="48462-278">More precisely, the annotation context for an inferred type argument is the context of the token that would have been followed by the `<...>` type parameter list, had there been one; i.e. the name of the generic method being called.</span></span> <span data-ttu-id="48462-279">对于转换为此类调用的查询表达式，上下文取自从中生成调用的查询子句的初始上下文关键字。</span><span class="sxs-lookup"><span data-stu-id="48462-279">For query expressions that translate to such calls, the context is taken from the initial contextual keyword of the query clause from which the call is generated.</span></span>

### <a name="the-first-phase"></a><span data-ttu-id="48462-280">第一阶段</span><span class="sxs-lookup"><span data-stu-id="48462-280">The first phase</span></span>

<span data-ttu-id="48462-281">可以为 null 的引用类型从初始表达式流入边界，如下所述。</span><span class="sxs-lookup"><span data-stu-id="48462-281">Nullable reference types flow into the bounds from the initial expressions, as described below.</span></span> <span data-ttu-id="48462-282">此外，引入了两种新的界限，即 `null` 和 `default` 。</span><span class="sxs-lookup"><span data-stu-id="48462-282">In addition, two new kinds of bounds, namely `null` and `default` are introduced.</span></span> <span data-ttu-id="48462-283">其目的是在输入表达式中执行 `null` 或 `default` ，这可能会导致推断出的类型可以为 null，即使在其他情况下也是如此。</span><span class="sxs-lookup"><span data-stu-id="48462-283">Their purpose is to carry through occurrences of `null` or `default` in the input expressions, which may cause an inferred type to be nullable, even when it otherwise wouldn't.</span></span> <span data-ttu-id="48462-284">这甚至适用于可为 null 的值类型，这些 *值* 类型得到了增强，可在推断过程中选取 "非 null"。</span><span class="sxs-lookup"><span data-stu-id="48462-284">This works even for nullable *value* types, which are enhanced to pick up "nullness" in the inference process.</span></span>

<span data-ttu-id="48462-285">在第一阶段中，确定要添加的界限如下：</span><span class="sxs-lookup"><span data-stu-id="48462-285">The determination of what bounds to add in the first phase are enhanced as follows:</span></span>

<span data-ttu-id="48462-286">如果参数 `Ei` 具有引用类型，则 `U` 用于推理的类型取决于的 null 状态及其 `Ei` 声明的类型：</span><span class="sxs-lookup"><span data-stu-id="48462-286">If an argument `Ei` has a reference type, the type `U` used for inference depends on the null state of `Ei` as well as its declared type:</span></span>
- <span data-ttu-id="48462-287">如果声明的类型是不可 null 引用类型 `U0` 或可以为 null 的引用类型， `U0?` 则</span><span class="sxs-lookup"><span data-stu-id="48462-287">If the declared type is a nonnullable reference type `U0` or a nullable reference type `U0?` then</span></span>
    - <span data-ttu-id="48462-288">如果的 null 状态 `Ei` 为 "not null"，则 `U` 为 `U0`</span><span class="sxs-lookup"><span data-stu-id="48462-288">if the null state of `Ei` is "not null" then `U` is `U0`</span></span>
    - <span data-ttu-id="48462-289">如果的 null 状态 `Ei` 为 "可能为 null"，则 `U` 为 `U0?`</span><span class="sxs-lookup"><span data-stu-id="48462-289">if the null state of `Ei` is "maybe null" then `U` is `U0?`</span></span>
- <span data-ttu-id="48462-290">否则 `Ei` ，如果具有已声明的类型， `U` 则为该类型</span><span class="sxs-lookup"><span data-stu-id="48462-290">Otherwise if `Ei` has a declared type, `U` is that type</span></span>
- <span data-ttu-id="48462-291">否则，如果为，则 `Ei` `null` `U` 为特殊界限 `null`</span><span class="sxs-lookup"><span data-stu-id="48462-291">Otherwise if `Ei` is `null` then `U` is the special bound `null`</span></span>
- <span data-ttu-id="48462-292">否则，如果为，则 `Ei` `default` `U` 为特殊界限 `default`</span><span class="sxs-lookup"><span data-stu-id="48462-292">Otherwise if `Ei` is `default` then `U` is the special bound `default`</span></span>
- <span data-ttu-id="48462-293">否则，不进行推理。</span><span class="sxs-lookup"><span data-stu-id="48462-293">Otherwise no inference is made.</span></span>

### <a name="exact-upper-bound-and-lower-bound-inferences"></a><span data-ttu-id="48462-294">精确、上限和下限推理</span><span class="sxs-lookup"><span data-stu-id="48462-294">Exact, upper-bound and lower-bound inferences</span></span>

<span data-ttu-id="48462-295">在 *从* 类型推断 `U` *为* 类型时 `V` ，如果 `V` 是可以为 null 的引用类型，则将在 `V0?` `V0` `V` 以下子句中使用而不是。</span><span class="sxs-lookup"><span data-stu-id="48462-295">In inferences *from* the type `U` *to* the type `V`, if `V` is a nullable reference type `V0?`, then `V0` is used instead of `V` in the following clauses.</span></span>
- <span data-ttu-id="48462-296">如果 `V` 是未固定的类型变量中的一个， `U` 则会像以前一样，添加一个精确、上下限或下限</span><span class="sxs-lookup"><span data-stu-id="48462-296">If `V` is one of the unfixed type variables, `U` is added as an exact, upper or lower bound as before</span></span>
- <span data-ttu-id="48462-297">否则，如果 `U` 为 `null` 或 `default` ，则不进行推理</span><span class="sxs-lookup"><span data-stu-id="48462-297">Otherwise, if `U` is `null` or `default`, no inference is made</span></span>
- <span data-ttu-id="48462-298">否则，如果 `U` 是可以为 null 的引用类型 `U0?` ，则 `U0` 将在 `U` 后续子句中使用而不是。</span><span class="sxs-lookup"><span data-stu-id="48462-298">Otherwise, if `U` is a nullable reference type `U0?`, then `U0` is used instead of `U` in the subsequent clauses.</span></span>

<span data-ttu-id="48462-299">实质上，与某个未固定的类型变量直接相关的可为 null 将保留在其边界内。</span><span class="sxs-lookup"><span data-stu-id="48462-299">The essence is that nullability that pertains directly to one of the unfixed type variables is preserved into its bounds.</span></span> <span data-ttu-id="48462-300">另一方面，如果推断将进一步递归到源和目标类型，则会忽略为空性。</span><span class="sxs-lookup"><span data-stu-id="48462-300">For the inferences that recurse further into the source and target types, on the other hand, nullability is ignored.</span></span> <span data-ttu-id="48462-301">它可以或不匹配，但如果不匹配，则会在以后选择并应用重载时发出警告。</span><span class="sxs-lookup"><span data-stu-id="48462-301">It may or may not match, but if it doesn't, a warning will be issued later if the overload is chosen and applied.</span></span>

### <a name="fixing"></a><span data-ttu-id="48462-302">修正 </span><span class="sxs-lookup"><span data-stu-id="48462-302">Fixing</span></span>

<span data-ttu-id="48462-303">如果多个边界彼此之间相互转换，但不同，则该规范并不是一种很好的描述。</span><span class="sxs-lookup"><span data-stu-id="48462-303">The spec currently does not do a good job of describing what happens when multiple bounds are identity convertible to each other, but are different.</span></span> <span data-ttu-id="48462-304">这种情况可能发生 `object` 在与之间， `dynamic` 在不同于元素名称的元组类型之间、构造它们的类型之间以及 `C` `C?` 引用类型中的类型之间。</span><span class="sxs-lookup"><span data-stu-id="48462-304">This may happen between `object` and `dynamic`, between tuple types that differ only in element names, between types constructed thereof and now also between `C` and `C?` for reference types.</span></span>

<span data-ttu-id="48462-305">此外，我们还需要将 "非 null" 从输入表达式传播到结果类型。</span><span class="sxs-lookup"><span data-stu-id="48462-305">In addition we need to propagate "nullness" from the input expressions to the result type.</span></span> 

<span data-ttu-id="48462-306">为了应对这些情况，我们添加了更多的修复阶段，这现在是：</span><span class="sxs-lookup"><span data-stu-id="48462-306">To handle these we add more phases to fixing, which is now:</span></span>

1. <span data-ttu-id="48462-307">将所有边界中的所有类型都作为候选项收集， `?` 从所有可为 null 的引用类型中移除</span><span class="sxs-lookup"><span data-stu-id="48462-307">Gather all the types in all the bounds as candidates, removing `?` from all that are nullable reference types</span></span>
2. <span data-ttu-id="48462-308">根据 (保持和限制) 的精确、下限和上限的要求，消除候选项 `null` `default`</span><span class="sxs-lookup"><span data-stu-id="48462-308">Eliminate candidates based on requirements of exact, lower and upper bounds (keeping `null` and `default` bounds)</span></span>
3. <span data-ttu-id="48462-309">消除没有隐式转换为所有其他候选项的候选项</span><span class="sxs-lookup"><span data-stu-id="48462-309">Eliminate candidates that do not have an implicit conversion to all the other candidates</span></span>
4. <span data-ttu-id="48462-310">如果剩余的候选项彼此之间没有标识转换，则类型推理将失败</span><span class="sxs-lookup"><span data-stu-id="48462-310">If the remaining candidates do not all have identity conversions to one another, then type inference fails</span></span>
5. <span data-ttu-id="48462-311">按如下所述 *合并* 剩余候选项</span><span class="sxs-lookup"><span data-stu-id="48462-311">*Merge* the remaining candidates as described below</span></span>
6. <span data-ttu-id="48462-312">如果生成的候选项是引用类型或不可 null 值类型，并且 *所有* 完全 *限定或下限* 都是可以为 null 的值类型、可以为 null 的引用类型 `null` 或 `default` ，则 `?` 将其添加到生成的候选项，使其成为可以为 null 的值类型或引用类型。</span><span class="sxs-lookup"><span data-stu-id="48462-312">If the resulting candidate is a reference type or a nonnullable value type and *all* of the exact bounds or *any* of the lower bounds are nullable value types, nullable reference types, `null` or `default`, then `?` is added to the resulting candidate, making it a nullable value type or reference type.</span></span>

<span data-ttu-id="48462-313">两个候选类型之间介绍了 *合并* 。</span><span class="sxs-lookup"><span data-stu-id="48462-313">*Merging* is described between two candidate types.</span></span> <span data-ttu-id="48462-314">它是可传递和可交换的，因此，可按任何顺序将候选项合并，并获得相同的最终结果。</span><span class="sxs-lookup"><span data-stu-id="48462-314">It is transitive and commutative, so the candidates can be merged in any order with the same ultimate result.</span></span> <span data-ttu-id="48462-315">如果两个候选类型不能相互转换，则它是不确定的。</span><span class="sxs-lookup"><span data-stu-id="48462-315">It is undefined if the two candidate types are not identity convertible to each other.</span></span>

<span data-ttu-id="48462-316">*Merge* 函数采用两种候选类型和方向 ( *+* 或 *-* ) ：</span><span class="sxs-lookup"><span data-stu-id="48462-316">The *Merge* function takes two candidate types and a direction ( *+* or *-* ):</span></span>

- <span data-ttu-id="48462-317">*Merge* (`T` ， `T` ， *d* ) = T</span><span class="sxs-lookup"><span data-stu-id="48462-317">*Merge* (`T`, `T`, *d* ) = T</span></span>
- <span data-ttu-id="48462-318">*Merge* (`S` ， `T?` ， *+* ) = *merge* (`S?` ， `T` ， *+* ) = *merge* (`S` ， `T` ， *+* ) `?`</span><span class="sxs-lookup"><span data-stu-id="48462-318">*Merge* (`S`, `T?`, *+* ) = *Merge* (`S?`, `T`, *+* ) = *Merge* (`S`, `T`, *+* )`?`</span></span>
- <span data-ttu-id="48462-319">*Merge* (`S` ， `T?` ， *-* ) = *merge* (`S?` ， `T` ， *-* ) = *merge* (`S` ， `T` ， *-* ) </span><span class="sxs-lookup"><span data-stu-id="48462-319">*Merge* (`S`, `T?`, *-* ) = *Merge* (`S?`, `T`, *-* ) = *Merge* (`S`, `T`, *-* )</span></span>
- <span data-ttu-id="48462-320">*Merge* (`C<S1,...,Sn>` ， `C<T1,...,Tn>` ， *+* ) = `C<` *merge* (`S1` ， `T1` ， *d1* ) `,...,` *merge* (`Sn` ， `Tn` ， *dn* ) `>` ， *其中*</span><span class="sxs-lookup"><span data-stu-id="48462-320">*Merge* (`C<S1,...,Sn>`, `C<T1,...,Tn>`, *+* ) = `C<`*Merge* (`S1`, `T1`, *d1* )`,...,`*Merge* (`Sn`, `Tn`, *dn* )`>`, *where*</span></span>
    - <span data-ttu-id="48462-321">`di` = *+* 如果 `i` 的第一个类型参数 `C<...>` 是协变的</span><span class="sxs-lookup"><span data-stu-id="48462-321">`di` = *+* if the `i`'th type parameter of `C<...>` is covariant</span></span>
    - <span data-ttu-id="48462-322">`di` = *-* 如果的 `i` 第一个类型参数 `C<...>` 为逆变或固定</span><span class="sxs-lookup"><span data-stu-id="48462-322">`di` = *-* if the `i`'th type parameter of `C<...>` is contra- or invariant</span></span>
- <span data-ttu-id="48462-323">*Merge* (`C<S1,...,Sn>` ， `C<T1,...,Tn>` ， *-* ) = `C<` *merge* (`S1` ， `T1` ， *d1* ) `,...,` *merge* (`Sn` ， `Tn` ， *dn* ) `>` ， *其中*</span><span class="sxs-lookup"><span data-stu-id="48462-323">*Merge* (`C<S1,...,Sn>`, `C<T1,...,Tn>`, *-* ) = `C<`*Merge* (`S1`, `T1`, *d1* )`,...,`*Merge* (`Sn`, `Tn`, *dn* )`>`, *where*</span></span>
    - <span data-ttu-id="48462-324">`di` = *-* 如果 `i` 的第一个类型参数 `C<...>` 是协变的</span><span class="sxs-lookup"><span data-stu-id="48462-324">`di` = *-* if the `i`'th type parameter of `C<...>` is covariant</span></span>
    - <span data-ttu-id="48462-325">`di` = *+* 如果的 `i` 第一个类型参数 `C<...>` 为逆变或固定</span><span class="sxs-lookup"><span data-stu-id="48462-325">`di` = *+* if the `i`'th type parameter of `C<...>` is contra- or invariant</span></span>
- <span data-ttu-id="48462-326">*Merge* (`(S1 s1,..., Sn sn)` ， `(T1 t1,..., Tn tn)` ， *d* ) = `(` *merge* (`S1` ， `T1` ， *d* ) `n1,...,` *合并* (`Sn` ， `Tn` ， *d* ) `nn)` ， *其中*</span><span class="sxs-lookup"><span data-stu-id="48462-326">*Merge* (`(S1 s1,..., Sn sn)`, `(T1 t1,..., Tn tn)`, *d* ) = `(`*Merge* (`S1`, `T1`, *d* )`n1,...,`*Merge* (`Sn`, `Tn`, *d* ) `nn)`, *where*</span></span>
    - <span data-ttu-id="48462-327">`ni` 如果 `si` 和 `ti` 不同，或者两者都不存在，则不存在</span><span class="sxs-lookup"><span data-stu-id="48462-327">`ni` is absent if `si` and `ti` differ, or if both are absent</span></span>
    - <span data-ttu-id="48462-328">`ni` 是 `si` `si` 和 `ti` 是否相同</span><span class="sxs-lookup"><span data-stu-id="48462-328">`ni` is `si` if `si` and `ti` are the same</span></span>
- <span data-ttu-id="48462-329">*Merge* (`object` ， `dynamic`) = *merge* (`dynamic` ， `object`) = `dynamic`</span><span class="sxs-lookup"><span data-stu-id="48462-329">*Merge* (`object`, `dynamic`) = *Merge* (`dynamic`, `object`) = `dynamic`</span></span>

## <a name="warnings"></a><span data-ttu-id="48462-330">警告</span><span class="sxs-lookup"><span data-stu-id="48462-330">Warnings</span></span>

### <a name="potential-null-assignment"></a><span data-ttu-id="48462-331">潜在 null 赋值</span><span class="sxs-lookup"><span data-stu-id="48462-331">Potential null assignment</span></span>

### <a name="potential-null-dereference"></a><span data-ttu-id="48462-332">潜在的空取消引用</span><span class="sxs-lookup"><span data-stu-id="48462-332">Potential null dereference</span></span>

### <a name="constraint-nullability-mismatch"></a><span data-ttu-id="48462-333">约束为空性不匹配</span><span class="sxs-lookup"><span data-stu-id="48462-333">Constraint nullability mismatch</span></span>

### <a name="nullable-types-in-disabled-annotation-context"></a><span data-ttu-id="48462-334">禁用的注释上下文中可以为 null 的类型</span><span class="sxs-lookup"><span data-stu-id="48462-334">Nullable types in disabled annotation context</span></span>

## <a name="attributes-for-special-null-behavior"></a><span data-ttu-id="48462-335">特殊 null 行为的特性</span><span class="sxs-lookup"><span data-stu-id="48462-335">Attributes for special null behavior</span></span>


