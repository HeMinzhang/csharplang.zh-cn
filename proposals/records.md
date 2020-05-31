---
ms.openlocfilehash: 6d8f0512d478d3d82cb262466e49a22a9e3cd0ee
ms.sourcegitcommit: ae114131069ca76e4a1ec8149b7bab81fce8965c
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/30/2020
ms.locfileid: "84227974"
---
# <a name="records-work-in-progress"></a><span data-ttu-id="33291-101">记录正在进行的工作</span><span class="sxs-lookup"><span data-stu-id="33291-101">Records Work-in-Progress</span></span>

<span data-ttu-id="33291-102">不同于其他记录建议，这不是一项提议，而是一项工作，用于记录有关记录功能的共识设计决策。</span><span class="sxs-lookup"><span data-stu-id="33291-102">Unlike the other records proposals, this is not a proposal in itself, but a work-in-progress designed to record consensus design decisions for the records feature.</span></span> <span data-ttu-id="33291-103">为了解决问题，将根据需要添加规范详细信息。</span><span class="sxs-lookup"><span data-stu-id="33291-103">Specification detail will be added as necessary to resolve questions.</span></span>

<span data-ttu-id="33291-104">建议添加记录的语法，如下所示：</span><span class="sxs-lookup"><span data-stu-id="33291-104">The syntax for a record is proposed to be added as follows:</span></span>

```antlr
class_declaration
    : attributes? class_modifiers? 'partial'? 'class' identifier type_parameter_list?
      parameter_list? type_parameter_constraints_clauses? class_body
    ;

struct_declaration
    : attributes? struct_modifiers? 'partial'? 'struct' identifier type_parameter_list?
      parameter_list? struct_interfaces? type_parameter_constraints_clauses? struct_body
    ;

class_body
    : '{' class_member_declarations? '}'
    | ';'
    ;

struct_body
    : '{' struct_members_declarations? '}'
    | ';'
    ;
```

<span data-ttu-id="33291-105">`attributes`非终端还将允许新的上下文属性 `data` 。</span><span class="sxs-lookup"><span data-stu-id="33291-105">The `attributes` non-terminal will also permit a new contextual attribute, `data`.</span></span>

<span data-ttu-id="33291-106">用参数列表或修饰符声明的类（结构） `data` 称为记录类（record struct），其中的任何一个是记录类型。</span><span class="sxs-lookup"><span data-stu-id="33291-106">A class (struct) declared with a parameter list or `data` modifier is called a record class (record struct), either of which is a record type.</span></span>

<span data-ttu-id="33291-107">声明不包含参数列表和修饰符的记录类型是错误的 `data` 。</span><span class="sxs-lookup"><span data-stu-id="33291-107">It is an error to declare a record type without both a parameter list and the `data` modifier.</span></span>

## <a name="members-of-a-record-type"></a><span data-ttu-id="33291-108">记录类型的成员</span><span class="sxs-lookup"><span data-stu-id="33291-108">Members of a record type</span></span>

<span data-ttu-id="33291-109">除了在类或结构体中声明的成员以外，记录类型还具有下列附加成员：</span><span class="sxs-lookup"><span data-stu-id="33291-109">In addition to the members declared in the class or struct body, a record type has the following additional members:</span></span>

### <a name="primary-constructor"></a><span data-ttu-id="33291-110">主构造函数</span><span class="sxs-lookup"><span data-stu-id="33291-110">Primary Constructor</span></span>

<span data-ttu-id="33291-111">记录类型具有公共构造函数，该构造函数的签名对应于类型声明的值参数。</span><span class="sxs-lookup"><span data-stu-id="33291-111">A record type has a public constructor whose signature corresponds to the value parameters of the type declaration.</span></span> <span data-ttu-id="33291-112">这称为类型的主构造函数，并导致禁止显示隐式声明的默认类构造函数（如果存在）。</span><span class="sxs-lookup"><span data-stu-id="33291-112">This is called the primary constructor for the type, and causes the implicitly declared default class constructor, if present, to be suppressed.</span></span> <span data-ttu-id="33291-113">类中已存在具有相同签名的主构造函数和构造函数是错误的。</span><span class="sxs-lookup"><span data-stu-id="33291-113">It is an error to have a primary constructor and a constructor with the same signature already present in the class.</span></span>

<span data-ttu-id="33291-114">在运行时，主构造函数</span><span class="sxs-lookup"><span data-stu-id="33291-114">At runtime the primary constructor</span></span>

1. <span data-ttu-id="33291-115">执行出现在类体中的实例字段初始值设定项;然后调用不带参数的基类构造函数。</span><span class="sxs-lookup"><span data-stu-id="33291-115">executes the instance field initializers appearing in the class-body; and then  invokes the base class constructor with no arguments.</span></span>

1. <span data-ttu-id="33291-116">为对应于值参数的属性初始化编译器生成的支持字段（如果这些属性是编译器提供的）</span><span class="sxs-lookup"><span data-stu-id="33291-116">initializes compiler-generated backing fields for the properties corresponding to the value parameters (if these properties are compiler-provided</span></span>

### <a name="properties"></a><span data-ttu-id="33291-117">属性</span><span class="sxs-lookup"><span data-stu-id="33291-117">Properties</span></span>

<span data-ttu-id="33291-118">对于记录类型声明的每个记录参数，都有一个对应的公共属性成员，其名称和类型取自值参数声明。</span><span class="sxs-lookup"><span data-stu-id="33291-118">For each record parameter of a record type declaration there is a corresponding public property member whose name and type are taken from the value parameter declaration.</span></span> <span data-ttu-id="33291-119">如果没有显式声明或继承具有 get 访问器和此名称和类型的具体（即非抽象）属性，则编译器将生成，如下所示：</span><span class="sxs-lookup"><span data-stu-id="33291-119">If no concrete (i.e. non-abstract) property with a get accessor and with this name and type is explicitly declared or inherited, it is produced by the compiler as follows:</span></span>

<span data-ttu-id="33291-120">对于记录结构或记录类：</span><span class="sxs-lookup"><span data-stu-id="33291-120">For a record struct or a record class:</span></span>

* <span data-ttu-id="33291-121">创建公共 `get` 和 `init` 自动属性（请参阅单独的 `init` 访问器规范）。</span><span class="sxs-lookup"><span data-stu-id="33291-121">A public `get` and `init` auto-property is created (see separate `init` accessor specification).</span></span> <span data-ttu-id="33291-122">在构造过程中，它的值将与相应的主构造函数参数的值一起初始化。</span><span class="sxs-lookup"><span data-stu-id="33291-122">Its value is initialized during construction with the value of the corresponding primary constructor parameter.</span></span> <span data-ttu-id="33291-123">将重写每个 "匹配" 继承抽象属性的 get 访问器。</span><span class="sxs-lookup"><span data-stu-id="33291-123">Each "matching" inherited abstract property's get accessor is overridden.</span></span>

### <a name="equality-members"></a><span data-ttu-id="33291-124">相等成员</span><span class="sxs-lookup"><span data-stu-id="33291-124">Equality members</span></span>

<span data-ttu-id="33291-125">记录类型为以下方法生成合成实现：</span><span class="sxs-lookup"><span data-stu-id="33291-125">Record types produce synthesized implementations for the following methods:</span></span>

* <span data-ttu-id="33291-126">`object.GetHashCode()`重写（除非它是密封的或用户提供的）</span><span class="sxs-lookup"><span data-stu-id="33291-126">`object.GetHashCode()` override, unless it is sealed or user provided</span></span>
* <span data-ttu-id="33291-127">`object.Equals(object)`重写（除非它是密封的或用户提供的）</span><span class="sxs-lookup"><span data-stu-id="33291-127">`object.Equals(object)` override, unless it is sealed or user provided</span></span>
* <span data-ttu-id="33291-128">`T Equals(T)`方法，其中 `T` 是当前类型</span><span class="sxs-lookup"><span data-stu-id="33291-128">`T Equals(T)` method, where `T` is the current type</span></span>

<span data-ttu-id="33291-129">`T Equals(T)`指定为执行值相等性，这样， `Equals` 当且仅当接收方类型中声明的所有实例字段都等于其他类型的字段时，才为 true。</span><span class="sxs-lookup"><span data-stu-id="33291-129">`T Equals(T)` is specified to perform value equality such that `Equals` is true if and only if all the instance fields declared in the receiver type are equal to the fields of the other type.</span></span>

<span data-ttu-id="33291-130">`object.Equals`执行等效于</span><span class="sxs-lookup"><span data-stu-id="33291-130">`object.Equals` performs the equivalent of</span></span>

```C#
override Equals(object o) => Equals(o as T);
```

### <a name="copy-and-clone-members"></a><span data-ttu-id="33291-131">复制和克隆成员</span><span class="sxs-lookup"><span data-stu-id="33291-131">Copy and Clone members</span></span>

<span data-ttu-id="33291-132">如果记录类型中尚未声明具有相同签名的方法，则记录类型包含两个合成复制成员：</span><span class="sxs-lookup"><span data-stu-id="33291-132">A record type contains two synthesized copying members if methods with the same signature are not already declared within the record type:</span></span>

* <span data-ttu-id="33291-133">一个受保护的构造函数，采用记录类型的单个自变量。</span><span class="sxs-lookup"><span data-stu-id="33291-133">A protected constructor taking a single argument of the record type.</span></span>
* <span data-ttu-id="33291-134">一个调用的公共无参数实例方法 `Clone` ，该方法返回记录类型。</span><span class="sxs-lookup"><span data-stu-id="33291-134">A public parameterless instance method called `Clone` which returns the record type.</span></span>

<span data-ttu-id="33291-135">受保护的构造函数称为 "复制构造函数"，合成体将输入类型中声明的所有实例字段的值复制到的相应字段 `this` 。</span><span class="sxs-lookup"><span data-stu-id="33291-135">The protected constructor is referred to as the "copy constructor" and the synthesized body copies the values of all instance fields declared in the input type to the corresponding fields of `this`.</span></span>

<span data-ttu-id="33291-136">`Clone`方法返回对构造函数的调用结果，该结果与复制构造函数具有相同的签名。</span><span class="sxs-lookup"><span data-stu-id="33291-136">The `Clone` method returns the result of a call to a constructor with the same signature as the copy constructor.</span></span>
## <a name="with-expression"></a><span data-ttu-id="33291-137">`with` 表达式</span><span class="sxs-lookup"><span data-stu-id="33291-137">`with` expression</span></span>

<span data-ttu-id="33291-138">`with`表达式是使用以下语法的新表达式。</span><span class="sxs-lookup"><span data-stu-id="33291-138">A `with` expression is a new expression using the following syntax.</span></span>

```antlr
with_expression
    : switch_expression
    | switch_expression 'with' '{' member_initializer_list? '}'
    ;
    
member_initializer_list
    : member_initializer (',' member_initializer)*
    ;

member_initializer
    : identifier '=' expression
    ;
```

<span data-ttu-id="33291-139">`with`表达式允许 "非破坏性转变"，旨在生成接收方表达式的副本，并在中对赋值进行修改 `member_initializer_list` 。</span><span class="sxs-lookup"><span data-stu-id="33291-139">A `with` expression allows for "non-destructive mutation", designed to produce a copy of the receiver expression with modifications in assignments in the `member_initializer_list`.</span></span>

<span data-ttu-id="33291-140">有效的 `with` 表达式包含具有非 void 类型的接收方。</span><span class="sxs-lookup"><span data-stu-id="33291-140">A valid `with` expression has a receiver with a non-void type.</span></span> <span data-ttu-id="33291-141">接收器类型必须包含一个名为的可访问参数实例方法，该方法 `Clone` 的返回类型必须是接收方类型或其基类型。</span><span class="sxs-lookup"><span data-stu-id="33291-141">The receiver type must contain an accessible parameterless instance method called `Clone` whose return type must be the receiver type, or a base type thereof.</span></span>

<span data-ttu-id="33291-142">表达式的右侧 `with` 是一个 `member_initializer_list` 带有*标识符*的赋值序列的，它必须是该方法的返回类型的可访问实例字段或属性 `Clone()` 。</span><span class="sxs-lookup"><span data-stu-id="33291-142">On the right hand side of the `with` expression is an `member_initializer_list` with a sequence of assignments to *identifier*, which must an accessible instance field or property of the return type of the `Clone()` method.</span></span>

<span data-ttu-id="33291-143">每个的 `member_initializer` 处理方式与方法的返回值上的字段或属性目标的赋值方式相同 `Clone()` 。</span><span class="sxs-lookup"><span data-stu-id="33291-143">Each `member_initializer` is processed the same way as an assignment to the field or property target on the return value of the `Clone()` method.</span></span> <span data-ttu-id="33291-144">`Clone()`方法只执行一次，并按词法顺序处理赋值。</span><span class="sxs-lookup"><span data-stu-id="33291-144">The `Clone()` method is executed only once and the assignments are processed in lexical order.</span></span>
