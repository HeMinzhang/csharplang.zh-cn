---
ms.openlocfilehash: 01fd4f1bc9e0a5b8ea1473e3c7c00fca80c5c8fb
ms.sourcegitcommit: 06ee75e6e3f1e0d9b4859ffed66024364d3d8f26
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/21/2020
ms.locfileid: "88720426"
---

# <a name="target-typed-new-expressions"></a><span data-ttu-id="3b7d7-101">目标类型的 `new` 表达式</span><span class="sxs-lookup"><span data-stu-id="3b7d7-101">Target-typed `new` expressions</span></span>

* <span data-ttu-id="3b7d7-102">[x] 建议</span><span class="sxs-lookup"><span data-stu-id="3b7d7-102">[x] Proposed</span></span>
* <span data-ttu-id="3b7d7-103">[x] 原型</span><span class="sxs-lookup"><span data-stu-id="3b7d7-103">[x] Prototype</span></span>
* <span data-ttu-id="3b7d7-104">[] 实现</span><span class="sxs-lookup"><span data-stu-id="3b7d7-104">[ ] Implementation</span></span>
* <span data-ttu-id="3b7d7-105">[] 规范</span><span class="sxs-lookup"><span data-stu-id="3b7d7-105">[ ] Specification</span></span>

## <a name="summary"></a><span data-ttu-id="3b7d7-106">摘要</span><span class="sxs-lookup"><span data-stu-id="3b7d7-106">Summary</span></span>
[summary]: #summary

<span data-ttu-id="3b7d7-107">当类型已知时，不需要构造函数的类型规范。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-107">Do not require type specification for constructors when the type is known.</span></span> 

## <a name="motivation"></a><span data-ttu-id="3b7d7-108">动机</span><span class="sxs-lookup"><span data-stu-id="3b7d7-108">Motivation</span></span>
[motivation]: #motivation

<span data-ttu-id="3b7d7-109">允许字段初始化，而不复制类型。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-109">Allow field initialization without duplicating the type.</span></span>
```cs
Dictionary<string, List<int>> field = new() {
    { "item1", new() { 1, 2, 3 } }
};
```

<span data-ttu-id="3b7d7-110">如果可从用法推断，则允许省略类型。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-110">Allow omitting the type when it can be inferred from usage.</span></span>
```cs
XmlReader.Create(reader, new() { IgnoreWhitespace = true });
```

<span data-ttu-id="3b7d7-111">实例化对象，而不会对类型进行拼写检查。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-111">Instantiate an object without spelling out the type.</span></span>
```cs
private readonly static object s_syncObj = new();
```

## <a name="specification"></a><span data-ttu-id="3b7d7-112">规格</span><span class="sxs-lookup"><span data-stu-id="3b7d7-112">Specification</span></span>
[design]: #detailed-design

<span data-ttu-id="3b7d7-113">接受新的句法窗体，其中的*类型*为可选项*target_typed_new* *object_creation_expression* 。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-113">A new syntactic form, *target_typed_new* of the *object_creation_expression* is accepted in which the *type* is optional.</span></span>

```antlr
object_creation_expression
    : 'new' type '(' argument_list? ')' object_or_collection_initializer?
    | 'new' type object_or_collection_initializer
    | target_typed_new
    ;
target_typed_new
    : 'new' '(' argument_list? ')' object_or_collection_initializer?
    ;
```

<span data-ttu-id="3b7d7-114">*Target_typed_new*表达式没有类型。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-114">A *target_typed_new* expression does not have a type.</span></span> <span data-ttu-id="3b7d7-115">但是，存在一个新的 *对象创建转换* ，它是从表达式到每个类型的 *target_typed_new* 的隐式转换。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-115">However, there is a new *object creation conversion* that is an implicit conversion from expression, that exists from a *target_typed_new* to every type.</span></span>

<span data-ttu-id="3b7d7-116">如果是的实例，则给定目标类型 `T` `T0` 为 `T` 的基础类型 `T` `System.Nullable` 。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-116">Given a target type `T`, the type `T0` is `T`'s underlying type if `T` is an instance of `System.Nullable`.</span></span> <span data-ttu-id="3b7d7-117">否则 `T0` 为 `T` 。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-117">Otherwise `T0` is `T`.</span></span> <span data-ttu-id="3b7d7-118">转换为类型的 *target_typed_new* 表达式的含义与 `T` 指定为类型的相应 *object_creation_expression* 的含义相同 `T0` 。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-118">The meaning of a *target_typed_new* expression that is converted to the type `T` is the same as the meaning of a corresponding *object_creation_expression* that specifies `T0` as the type.</span></span>

<span data-ttu-id="3b7d7-119">如果 *target_typed_new* 用作一元运算符或二元运算符的操作数，或者如果在不受 *对象创建转换*的情况下使用，则会发生编译时错误。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-119">It is a compile-time error if a *target_typed_new* is used as an operand of a unary or binary operator, or if it is used where it is not subject to an *object creation conversion*.</span></span>

> <span data-ttu-id="3b7d7-120">**打开问题：** 我们是否应允许委托和元组作为目标类型？</span><span class="sxs-lookup"><span data-stu-id="3b7d7-120">**Open Issue:** should we allow delegates and tuples as the target-type?</span></span>

<span data-ttu-id="3b7d7-121">上述规则包括 (引用类型) 和元组 (结构类型) 的委托。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-121">The above rules include delegates (a reference type) and tuples (a struct type).</span></span> <span data-ttu-id="3b7d7-122">尽管这两种类型都是可构造，但如果类型为 inferable，则可以使用匿名函数或元组文本。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-122">Although both types are constructible, if the type is inferable, an anonymous function or a tuple literal can already be used.</span></span>
```cs
(int a, int b) t = new(1, 2); // "new" is redundant
Action a = new(() => {}); // "new" is redundant

(int a, int b) t = new(); // OK; same as (0, 0)
Action a = new(); // no constructor found
```

### <a name="miscellaneous"></a><span data-ttu-id="3b7d7-123">其他</span><span class="sxs-lookup"><span data-stu-id="3b7d7-123">Miscellaneous</span></span>

<span data-ttu-id="3b7d7-124">下面是该规范的结果：</span><span class="sxs-lookup"><span data-stu-id="3b7d7-124">The following are consequences of the specification:</span></span>

- <span data-ttu-id="3b7d7-125">`throw new()` (目标类型，则允许 `System.Exception`) </span><span class="sxs-lookup"><span data-stu-id="3b7d7-125">`throw new()` is allowed (the target type is `System.Exception`)</span></span>
- <span data-ttu-id="3b7d7-126">`new`不允许将目标类型与二元运算符一起使用。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-126">Target-typed `new` is not allowed with binary operators.</span></span>
- <span data-ttu-id="3b7d7-127">没有目标类型时，不允许这样做：一元运算符，在中的 `foreach` `using` 析构，在表达式中，作为匿名类型属性 () ，在语句中，在语句中，在语句中，在语句中，在语句中的 `await` `new { Prop = new() }` `lock` `sizeof` `fixed` 成员访问 (`new().field`) 中 `someDynamic.Method(new())` `is` `??` ，作为运算符的操作数，在 LINQ (查询中，作为运算符的左操作数，.。。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-127">It is disallowed when there is no type to target: unary operators, collection of a `foreach`, in a `using`, in a deconstruction, in an `await` expression, as an anonymous type property (`new { Prop = new() }`), in a `lock` statement, in a `sizeof`, in a `fixed` statement, in a member access (`new().field`), in a dynamically dispatched operation (`someDynamic.Method(new())`), in a LINQ query, as the operand of the `is` operator, as the left operand of the `??` operator,  ...</span></span>
- <span data-ttu-id="3b7d7-128">它也不允许作为 `ref` 。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-128">It is also disallowed as a `ref`.</span></span>
- <span data-ttu-id="3b7d7-129">不允许将以下类型作为转换的目标</span><span class="sxs-lookup"><span data-stu-id="3b7d7-129">The following kinds of types are not permitted as targets of the conversion</span></span>
  - <span data-ttu-id="3b7d7-130">**枚举类型：** `new()` (为 `new Enum()`) 指定默认值，而 `new(1)` 不会起作用，因为枚举类型没有构造函数。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-130">**Enum types:** `new()` will work (as `new Enum()` works to give the default value), but `new(1)` will not work as enum types do not have a constructor.</span></span>
  - <span data-ttu-id="3b7d7-131">**接口类型：** 它的工作方式与 COM 类型对应的创建表达式相同。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-131">**Interface types:** This would work the same as the corresponding creation expression for COM types.</span></span>
  - <span data-ttu-id="3b7d7-132">**数组类型：** 数组需要一个特殊的语法来提供长度。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-132">**Array types:** arrays need a special syntax to provide the length.</span></span>    
  - <span data-ttu-id="3b7d7-133">**动态：** 我们不允许 `new dynamic()` ，因此不允许 `new()` `dynamic` 作为目标类型。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-133">**dynamic:** we don't allow `new dynamic()`, so we don't allow `new()` with `dynamic` as a target type.</span></span>
  - <span data-ttu-id="3b7d7-134">**元组：** 它们与使用基础类型创建对象的含义相同。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-134">**tuples:** These have the same meaning as an object creation using the underlying type.</span></span>
  - <span data-ttu-id="3b7d7-135">还会排除 *object_creation_expression* 中不允许的所有其他类型，例如，指针类型。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-135">All the other types that are not permitted in the *object_creation_expression* are excluded as well, for instance, pointer types.</span></span>   

## <a name="drawbacks"></a><span data-ttu-id="3b7d7-136">缺点</span><span class="sxs-lookup"><span data-stu-id="3b7d7-136">Drawbacks</span></span>
[drawbacks]: #drawbacks

<span data-ttu-id="3b7d7-137">对于目标类型化 `new` 创建新类别的重大更改，存在一些问题，但我们已使用 `null` 和，但 `default` 这并不是一个重要问题。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-137">There were some concerns with target-typed `new` creating new categories of breaking changes, but we already have that with `null` and `default`, and that has not been a significant problem.</span></span>

## <a name="alternatives"></a><span data-ttu-id="3b7d7-138">备选项</span><span class="sxs-lookup"><span data-stu-id="3b7d7-138">Alternatives</span></span>
[alternatives]: #alternatives

<span data-ttu-id="3b7d7-139">大多数关于字段初始化中太长时间的投诉都是关于类型 *参数* ，而不是类型本身，我们只能推断 `new Dictionary(...)` (或类似) 类型参数，并在本地从参数或集合初始值设定项推断类型参数。</span><span class="sxs-lookup"><span data-stu-id="3b7d7-139">Most of complaints about types being too long to duplicate in field initialization is about *type arguments* not the type itself, we could infer only type arguments like `new Dictionary(...)` (or similar) and infer type arguments locally from arguments or the collection initializer.</span></span>

## <a name="questions"></a><span data-ttu-id="3b7d7-140">问题</span><span class="sxs-lookup"><span data-stu-id="3b7d7-140">Questions</span></span>
[questions]: #questions

- <span data-ttu-id="3b7d7-141">是否应在表达式树中禁止使用情况？</span><span class="sxs-lookup"><span data-stu-id="3b7d7-141">Should we forbid usages in expression trees?</span></span> <span data-ttu-id="3b7d7-142"> (no) </span><span class="sxs-lookup"><span data-stu-id="3b7d7-142">(no)</span></span>
- <span data-ttu-id="3b7d7-143">此功能如何与 `dynamic` 参数交互？</span><span class="sxs-lookup"><span data-stu-id="3b7d7-143">How the feature interacts with `dynamic` arguments?</span></span> <span data-ttu-id="3b7d7-144"> (无特殊处理) </span><span class="sxs-lookup"><span data-stu-id="3b7d7-144">(no special treatment)</span></span>
- <span data-ttu-id="3b7d7-145">IntelliSense 应如何操作 `new()` ？</span><span class="sxs-lookup"><span data-stu-id="3b7d7-145">How IntelliSense should work with `new()`?</span></span> <span data-ttu-id="3b7d7-146">仅当存在单个目标类型时才 () </span><span class="sxs-lookup"><span data-stu-id="3b7d7-146">(only when there is a single target-type)</span></span>

## <a name="design-meetings"></a><span data-ttu-id="3b7d7-147">设计会议</span><span class="sxs-lookup"><span data-stu-id="3b7d7-147">Design meetings</span></span>

- [<span data-ttu-id="3b7d7-148">LDM-2017-10-18</span><span class="sxs-lookup"><span data-stu-id="3b7d7-148">LDM-2017-10-18</span></span>](https://github.com/dotnet/csharplang/blob/master/meetings/2017/LDM-2017-10-18.md#100)
- [<span data-ttu-id="3b7d7-149">LDM-2018-05-21</span><span class="sxs-lookup"><span data-stu-id="3b7d7-149">LDM-2018-05-21</span></span>](https://github.com/dotnet/csharplang/blob/master/meetings/2018/LDM-2018-05-21.md)
- [<span data-ttu-id="3b7d7-150">LDM-2018-06-25</span><span class="sxs-lookup"><span data-stu-id="3b7d7-150">LDM-2018-06-25</span></span>](https://github.com/dotnet/csharplang/blob/master/meetings/2018/LDM-2018-06-25.md)
- [<span data-ttu-id="3b7d7-151">LDM-2018-08-22</span><span class="sxs-lookup"><span data-stu-id="3b7d7-151">LDM-2018-08-22</span></span>](https://github.com/dotnet/csharplang/blob/master/meetings/2018/LDM-2018-08-22.md#target-typed-new)
- [<span data-ttu-id="3b7d7-152">LDM-2018-10-17</span><span class="sxs-lookup"><span data-stu-id="3b7d7-152">LDM-2018-10-17</span></span>](https://github.com/dotnet/csharplang/blob/master/meetings/2018/LDM-2018-10-17.md)
- [<span data-ttu-id="3b7d7-153">LDM-2020-03-25</span><span class="sxs-lookup"><span data-stu-id="3b7d7-153">LDM-2020-03-25</span></span>](https://github.com/dotnet/csharplang/blob/master/meetings/2020/LDM-2020-03-25.md)
