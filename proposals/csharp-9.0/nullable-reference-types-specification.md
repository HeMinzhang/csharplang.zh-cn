---
ms.openlocfilehash: e6a784ef90308a5395c6b1db454a67d40c10e3e4
ms.sourcegitcommit: 00d9d791b6f1d9538a979a111e93cf935d9b6cfe
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/10/2020
ms.locfileid: "94423364"
---
# <a name="nullable-reference-types-specification"></a>可以为 null 的引用类型规范

***这是一个正在进行的工作-缺少几个部件或这些部件不完整。** _

此功能添加了两种新类型的可为 null 的类型 (可以为 null 的引用类型和可以为 null 的现有值类型) 为 null 的泛型类型，并引入了静态流分析以实现 null 安全。

## <a name="syntax"></a>语法

### <a name="nullable-reference-types-and-nullable-type-parameters"></a>可以为 null 的引用类型和可以为 null 的类型参数

可以为 null 的引用类型和可以为 null 的类型参数的语法 `T?` 与可以为 null 的值类型的短格式相同，但没有相应的长格式。

出于规范的目的，当前 `nullable_type` 生产被重命名为 `nullable_value_type` ，并 `nullable_reference_type` `nullable_type_parameter` 添加了生产：

```antlr
type
    : value_type
    | reference_type
    | nullable_type_parameter
    | type_parameter
    | type_unsafe
    ;

reference_type
    : ...
    | nullable_reference_type
    ;

nullable_reference_type
    : non_nullable_reference_type '?'
    ;

non_nullable_reference_type
    : reference_type
    ;

nullable_type_parameter
    : non_nullable_non_value_type_parameter '?'
    ;

non_nullable_non_value_type_parameter
    : type_parameter
    ;
```

`non_nullable_reference_type` `nullable_reference_type` 必须是不可 null 引用类型 (类、接口、委托或数组) 。

`non_nullable_non_value_type_parameter`In `nullable_type_parameter` 必须是不被约束为值类型的类型参数。

可以为 null 的引用类型和可以为 null 的类型参数不能出现在以下位置：

- 作为基类或接口
- 作为的接收方 `member_access`
- 作为 `type` 中的 `object_creation_expression`
- 作为 `delegate_type` 中的 `delegate_creation_expression`
- 作为 `type` 中的 `is_expression` ， `catch_clause` 或 `type_pattern`
- 作为 `interface` 完全限定的接口成员名称中的

在 `nullable_reference_type` 和 `nullable_type_parameter` _disabled * 可为 null 的注释上下文中提供了警告。

### <a name="class-and-class-constraint"></a>`class` and `class?` 约束

`class`约束具有可为 null 的对应项 `class?` ：

```antlr
primary_constraint
    : ...
    | 'class' '?'
    ;
```

`class`*已启用* 的批注上下文) 中 (约束的类型参数必须使用不可 null 引用类型进行实例化。

`class?` (或 `class` *已禁用* 的注释上下文中的类型形参) 可以使用可以为 null 的或不可 null 的引用类型进行实例化。

`class?`*已禁用* 批注上下文中的约束提供了警告。

### <a name="notnull-constraint"></a>`notnull` constraint

使用约束的类型形参 `notnull` 不能是可以为 null 的类型 (可以为 null 的值类型、可以为 null 的引用类型或可以为 null 的类型参数) 

```antlr
primary_constraint
    : ...
    | 'notnull'
    ;
```

### <a name="default-constraint"></a>`default` constraint

`default`约束可用于方法重写或显式实现，以消除 "可以为 null `T?` 的类型参数" 与 "可以为 null 的值类型" (`Nullable<T>`) 。 如果缺少 `default` 约束，则 `T?` 重写或显式实现中的语法将被解释为 `Nullable<T>`

请参见https://github.com/dotnet/csharplang/blob/master/proposals/csharp-9.0/unconstrained-type-parameter-annotations.md#default-constraint

### <a name="the-null-forgiving-operator"></a>包容性运算符

后修复后的 `!` 运算符称为包容性运算符。 它可以应用于 *primary_expression* 或 *null_conditional_expression* 内：

```antlr
primary_expression
    : ...
    | null_forgiving_expression
    ;

null_forgiving_expression
    : primary_expression '!'
    ;

null_conditional_expression
    : primary_expression null_conditional_operations_no_suppression suppression?
    ;

null_conditional_operations_no_suppression
    : null_conditional_operations? '?' '.' identifier type_argument_list?
    | null_conditional_operations? '?' '[' argument_list ']'
    | null_conditional_operations '.' identifier type_argument_list?
    | null_conditional_operations '[' argument_list ']'
    | null_conditional_operations '(' argument_list? ')'
    ;

null_conditional_operations
    : null_conditional_operations_no_suppression suppression?
    ;

suppression
    : '!'
    ;
```

例如：

```csharp
var v = expr!;
expr!.M();
_ = a?.b!.c;
```

`primary_expression`和 `null_conditional_operations_no_suppression` 必须是可以为 null 的类型。

后缀 `!` 运算符没有运行时效果，它的计算结果为基础表达式的结果。 其唯一的作用是将表达式的 null 状态更改为 "not null"，并限制在使用时给出的警告。

### <a name="nullable-compiler-directives"></a>可以为 null 的编译器指令

`#nullable` 指令控制可为 null 的批注和警告上下文。

```antlr
pp_directive
    : ...
    | pp_nullable
    ;

pp_nullable
    : whitespace? '#' whitespace? 'nullable' whitespace nullable_action (whitespace nullable_target)? pp_new_line
    ;

nullable_action
    : 'disable'
    | 'enable'
    | 'restore'
    ;

nullable_target
    : 'warnings'
    | 'annotations'
    ;
```

`#pragma warning` 展开指令以允许更改可为 null 的警告上下文：

```antlr
pragma_warning_body
    : ...
    | 'warning' whitespace warning_action whitespace 'nullable'
    ;
```

例如：

```csharp
#pragma warning disable nullable
```

## <a name="nullable-contexts"></a>可为空上下文

源代码的每个行都有 *可为 null 的注释上下文* 和 *可以为 null 的警告上下文* 。 这些控件控制是否可为 null 的批注是否有效，以及是否给定了可为空性警告。 给定行的批注上下文已 *禁用* 或 *启用* 。 给定行的警告上下文已 *禁用* 或 *启用* 。

既可在 c # 源代码) 之外的项目级别指定这两个上下文，也可通过预处理器指令在源文件中的任何位置指定 (`#nullable` 。 如果未提供任何项目级别设置，则默认情况下，这两个上下文都是 *禁用* 的。

`#nullable`指令控制源文本中的批注和警告上下文，并优先于项目级设置。

指令设置上下文 () 它控制后续代码行，直到另一个指令重写它，或直到源文件的结尾。

指令的效果如下所示：

- `#nullable disable`：将可为 null 的批注和警告上下文设置为 *已禁用*
- `#nullable enable`：将可为 null 的批注和警告上下文设置为 *enabled*
- `#nullable restore`：将可为 null 的批注和警告上下文还原到项目设置
- `#nullable disable annotations`：将可为 null 的注释上下文设置为 *已禁用*
- `#nullable enable annotations`：将可为 null 的注释上下文设置为 *enabled*
- `#nullable restore annotations`：将可为 null 的注释上下文还原到项目设置
- `#nullable disable warnings`：将可为 null 的警告上下文设置为 *已禁用*
- `#nullable enable warnings`：将可为 null 的警告上下文设置为 *已启用*
- `#nullable restore warnings`：将可为 null 的警告上下文还原到项目设置

## <a name="nullability-of-types"></a>类型为 Null 性

给定的类型可以具有以下三个 nullabilities 之一： *在意* 、 *不可 null* 和 *可以为 null* 。

如果将可能的值分配给 *不可 null* 类型，则可能会引发警告 `null` 。 但是， *在意* 和可以 *为* null 的类型为 "可 *赋值* "，可以 `null` 为其分配值而不会出现警告。

可以取消引用或分配 *在意* 和 *不可 null* 类型的值，而不会出现警告。 但是， *可以为* null 的类型的值为 " *空* 值"，并可能在取消引用或赋值时导致警告，而不进行正确的 null 检查。

Null 产生类型的 *默认 null 状态* 是 "可能是 null" 或 "可能是默认值"。 非 null 生成类型的默认 null 状态为 "not null"。

类型的类型和在确定其为 null 性时所发生的可为 null 的注释上下文：

- 不可 null 值类型 `S` 始终为 *不可 null*
- 可以为 null 的值类型 `S?` 始终 *可以为 null*
- `C`*禁用* 的注释上下文中的批注引用类型为 *在意*
- `C`*启用* 的批注上下文中的批注引用类型为 *不可 null*
- 可以为 null 的引用类型可以 `C?` *为 null* (但在 *禁用* 的批注上下文中可能生成警告) 

类型参数还会考虑它们的约束：

- `T`如果任何) 为可以为 null 的类型或 `class?` 约束 *可以为 null* ，则为所有约束都 (的类型参数
- 一个类型参数 `T` ，其中至少有一个约束为 *在意* 或 *不可 null* ，或者其中一个 `struct` 或 `class` 或 `notnull` 约束为
    - *禁用* 的注释上下文中的 *在意*
    - *已启用* 的批注上下文中的 *不可 null*
- 可以为 null 的类型参数 `T?` *可以为 null* ，但如果不是值类型，则 *已禁用* 的批注上下文中会生成一个警告。 `T`

### <a name="oblivious-vs-nonnullable"></a>在意 vs 不可 null

`type`当类型的最后一个标记在该上下文中时，将被视为在给定的批注上下文中发生。

源代码中的给定引用类型 `C` 是解释为在意还是不可 null 依赖于源代码的注释上下文。 但一旦建立，就会将其视为该类型的一部分，并在替换泛型类型参数的过程中将其视为 "随之一起移动"。 这就像在类型上有一个批注 `?` ，但不可见。

## <a name="constraints"></a>约束

可以为 null 的引用类型可用作泛型约束。

`class?` 表示 "可能为 null 的引用类型" 的新约束，而 `class` *启用* 的批注上下文中表示 "不可 null 引用类型"。

`default` 一个新约束，用于表示未知类型参数或值类型。 它只能用于重写和显式实现的方法。 对于此约束， `T?` 表示可以为 null 的类型参数，而不是的简写形式 `Nullable<T>` 。

`notnull` 表示不可 null 类型参数的新约束。

类型参数或约束的为空性并不会影响类型是否满足约束，但目前已 (可以为 null 的值类型不满足约束) 的情况除外 `struct` 。 但是，如果类型参数不满足约束的为 null 性要求，则可以提供警告。

## <a name="null-state-and-null-tracking"></a>Null 状态和 null 跟踪

给定源位置中的每个表达式都具有 *null 状态* ，指示其是否被认为可能计算为 null。 Null 状态为 "not null"、"可能为 null" 或 "可能为默认值"。 Null 状态用于确定是否应为不安全的转换和取消引用提供警告。

### <a name="null-tracking-for-variables"></a>变量的 Null 跟踪

对于指定变量或属性的某些表达式，将根据对它们的赋值、对它们执行的测试以及它们之间的控制流，在发生之间跟踪 null 状态。 这类似于为变量跟踪明确赋值的方式。 跟踪的表达式如下所示：

```antlr
tracked_expression
    : simple_name
    | this
    | base
    | tracked_expression '.' identifier
    ;
```

其中，标识符表示字段或属性。

被跟踪变量的 null 状态在无法访问的代码中为 "not null"。 这遵循了有关无法访问的代码的其他决策，如考虑将所有局部变量明确赋值。

***描述类似于明确赋值的空状态转换** _

### <a name="null-state-for-expressions"></a>表达式为 Null

表达式的 null 状态派生自其形式和类型以及它所涉及的变量的 null 状态。

### <a name="literals"></a>文本

和文本的 null 状态 `null` `default` 为 "可能是默认值"。 任何其他文本的 null 状态为 "not null"。

### <a name="simple-names"></a>简单名称

如果 `simple_name` 未归类为值，则其 null 状态为 "not null"。 否则为跟踪的表达式，其 null 状态将为其在此源位置跟踪的 null 状态。

### <a name="member-access"></a>成员访问

如果 `member_access` 未归类为值，则其 null 状态为 "not null"。 否则，如果是跟踪的表达式，则其 null 状态将为其在此源位置跟踪的 null 状态。 否则，其 null 状态为其类型的默认 null 状态。

### <a name="invocation-expressions"></a>调用表达式

如果 `invocation_expression` 调用使用一个或多个特性声明的成员以实现特殊的 null 行为，则 null 状态由这些特性决定。 否则，表达式的 null 状态为其类型的默认 null 状态。

### <a name="element-access"></a>元素访问

如果 `element_access` 调用使用一个或多个特性声明的索引器来实现特殊的 null 行为，则 null 状态由这些特性决定。 否则，表达式的 null 状态为其类型的默认 null 状态。

### <a name="base-access"></a>基本访问权限

如果 `B` 表示封闭类型的基类型， `base.I` 则具有与相同的 null 状态， `((B)this).I` 并且与 `base[E]` 具有相同的 null 状态 `((B)this)[E]` 。

### <a name="default-expressions"></a>默认表达式

`default(T)` 如果 `T` 已知为不可 null 值类型，则具有 null 状态 "not null"。 否则，它的状态为 "可能是默认值"。

### <a name="null-conditional-expressions"></a>Null 条件表达式

`null_conditional_expression`具有 null 状态 "可能为 null"。

### <a name="cast-expressions"></a>强制转换表达式

如果强制转换表达式 `(T)E` 调用用户定义的转换，则表达式的 null 状态为其类型的默认 null 状态。 否则，如果 `T` 为 _nullable *，则 null 状态为 "可能为 null"。 否则，null 状态与的 null 状态相同 `E` 。

***这需要 upddating** _

### <a name="await-expressions"></a>Await 表达式

的 null 状态 `await E` 为其类型的默认 null 状态。

### <a name="the-as-operator"></a>`as` 运算符

`as`表达式的状态为 "可能为 null"。

### <a name="the-null-coalescing-operator"></a>Null 合并运算符

`E1 ?? E2` 具有与相同的空状态 `E2`

### <a name="the-conditional-operator"></a>条件运算符

`E1 ? E2 : E3`如果和的 null 状态为 `E2` `E3` "not null"，则的 null 状态为 "not null"。 否则为 "可能为 null"。

### <a name="query-expressions"></a>查询表达式

查询表达式的 null 状态为其类型的默认 null 状态。

### <a name="assignment-operators"></a>赋值运算符

`E1 = E2` 和 `E1 op= E2` 都具有与 `E2` 应用任何隐式转换相同的空状态。

### <a name="unary-and-binary-operators"></a>一元运算符和二元运算符

如果一元或二元运算符调用了用一个或多个特性声明的用户定义的运算符来实现特殊的 null 行为，则 null 状态由这些特性决定。 否则，表达式的 null 状态为其类型的默认 null 状态。

_*_对于字符串和委托，二进制文件的特殊操作是什么 `+` ？_*_

### <a name="expressions-that-propagate-null-state"></a>传播 null 状态的表达式

`(E)``checked(E)`和 `unchecked(E)` 都具有与相同的 null 状态 `E` 。

### <a name="expressions-that-are-never-null"></a>从不为 null 的表达式

以下表达式格式的 null 状态始终为 "not null"：

- `this` access
- 内插字符串
- `new` 表达式 (对象、委托、匿名对象和数组创建表达式) 
- `typeof` 表达式
- `nameof` 表达式
- 匿名函数 (匿名方法和 lambda 表达式) 
- 包容性表达式
- `is` 表达式

### <a name="nested-functions"></a>嵌套函数

嵌套函数 (lambda 和局部函数) 被视为方法，但其捕获变量除外。
Lambda 或局部函数内捕获的变量的初始状态是该嵌套函数或 lambda 的所有 "使用" 中的变量的可以为 null 的状态的交集。 本地函数的使用是对该函数的调用，或将其转换为委托的位置。 Lambda 的使用是在源中定义它的点。

## <a name="type-inference"></a>类型推理

### <a name="nullable-implicitly-typed-local-variables"></a>可以为 null 的隐式类型的局部变量

`var` 推理引用类型的批注类型和不被约束为值类型的类型参数。
例如：
- 在中， `var s = "";` `var` 将推断为 `string?` 。
- 在中， `var t = new T();` `T` 不受约束的将 `var` 推断为 `T?` 。

### <a name="generic-type-inference"></a>泛型类型推理

泛型类型推理经过了增强，可帮助确定推断的引用类型是否可以为 null。 这是一个最大努力。 它可能会生成有关空性约束的警告，并且在将所选重载的推断类型应用于参数时，可能会导致可以为 null 的警告。

### <a name="the-first-phase"></a>第一阶段

可以为 null 的引用类型从初始表达式流入边界，如下所述。 此外，引入了两种新的界限，即 `null` 和 `default` 。 其目的是在输入表达式中执行 `null` 或 `default` ，这可能会导致推断出的类型可以为 null，即使在其他情况下也是如此。 这也适用于可为 null 的 _value * 类型，这些类型经过增强，可在推断过程中选取 "非 null"。

在第一阶段中，确定要添加的界限如下：

如果参数 `Ei` 具有引用类型，则 `U` 用于推理的类型取决于的 null 状态及其 `Ei` 声明的类型：
- 如果声明的类型是不可 null 引用类型 `U0` 或可以为 null 的引用类型， `U0?` 则
    - 如果的 null 状态 `Ei` 为 "not null"，则 `U` 为 `U0`
    - 如果的 null 状态 `Ei` 为 "可能为 null"，则 `U` 为 `U0?`
- 否则 `Ei` ，如果具有已声明的类型， `U` 则为该类型
- 否则，如果为，则 `Ei` `null` `U` 为特殊界限 `null`
- 否则，如果为，则 `Ei` `default` `U` 为特殊界限 `default`
- 否则，不进行推理。

### <a name="exact-upper-bound-and-lower-bound-inferences"></a>精确、上限和下限推理

在 *从* 类型推断 `U` *为* 类型时 `V` ，如果 `V` 是可以为 null 的引用类型，则将在 `V0?` `V0` `V` 以下子句中使用而不是。
- 如果 `V` 是未固定的类型变量中的一个， `U` 则会像以前一样，添加一个精确、上下限或下限
- 否则，如果 `U` 为 `null` 或 `default` ，则不进行推理
- 否则，如果 `U` 是可以为 null 的引用类型 `U0?` ，则 `U0` 将在 `U` 后续子句中使用而不是。

实质上，与某个未固定的类型变量直接相关的可为 null 将保留在其边界内。 另一方面，如果推断将进一步递归到源和目标类型，则会忽略为空性。 它可以或不匹配，但如果不匹配，则会在以后选择并应用重载时发出警告。

### <a name="fixing"></a>修正 

如果多个边界彼此之间相互转换，但不同，则该规范并不是一种很好的描述。 这种情况可能发生 `object` 在与之间， `dynamic` 在不同于元素名称的元组类型之间、构造它们的类型之间以及 `C` `C?` 引用类型中的类型之间。

此外，我们还需要将 "非 null" 从输入表达式传播到结果类型。

为了应对这些情况，我们添加了更多的修复阶段，这现在是：

1. 将所有边界中的所有类型都作为候选项收集， `?` 从所有可为 null 的引用类型中移除
2. 根据 (保持和限制) 的精确、下限和上限的要求，消除候选项 `null` `default`
3. 消除没有隐式转换为所有其他候选项的候选项
4. 如果剩余的候选项彼此之间没有标识转换，则类型推理将失败
5. 按如下所述 *合并* 剩余候选项
6. 如果生成的候选项是引用类型或不可 null 值类型，并且 *所有* 完全 *限定或下限* 都是可以为 null 的值类型、可以为 null 的引用类型 `null` 或 `default` ，则 `?` 将其添加到生成的候选项，使其成为可以为 null 的值类型或引用类型。

两个候选类型之间介绍了 *合并* 。 它是可传递和可交换的，因此，可按任何顺序将候选项合并，并获得相同的最终结果。 如果两个候选类型不能相互转换，则它是不确定的。

*Merge* 函数采用两种候选类型和方向 ( *+* 或 *-* ) ：

- *Merge* (`T` ， `T` ， *d* ) = T
- *Merge* (`S` ， `T?` ， *+* ) = *merge* (`S?` ， `T` ， *+* ) = *merge* (`S` ， `T` ， *+* ) `?`
- *Merge* (`S` ， `T?` ， *-* ) = *merge* (`S?` ， `T` ， *-* ) = *merge* (`S` ， `T` ， *-* ) 
- *Merge* (`C<S1,...,Sn>` ， `C<T1,...,Tn>` ， *+* ) = `C<` *merge* (`S1` ， `T1` ， *d1* ) `,...,` *merge* (`Sn` ， `Tn` ， *dn* ) `>` ， *其中*
    - `di` = *+* 如果 `i` 的第一个类型参数 `C<...>` 是协变的
    - `di` = *-* 如果的 `i` 第一个类型参数 `C<...>` 为逆变或固定
- *Merge* (`C<S1,...,Sn>` ， `C<T1,...,Tn>` ， *-* ) = `C<` *merge* (`S1` ， `T1` ， *d1* ) `,...,` *merge* (`Sn` ， `Tn` ， *dn* ) `>` ， *其中*
    - `di` = *-* 如果 `i` 的第一个类型参数 `C<...>` 是协变的
    - `di` = *+* 如果的 `i` 第一个类型参数 `C<...>` 为逆变或固定
- *Merge* (`(S1 s1,..., Sn sn)` ， `(T1 t1,..., Tn tn)` ， *d* ) = `(` *merge* (`S1` ， `T1` ， *d* ) `n1,...,` *合并* (`Sn` ， `Tn` ， *d* ) `nn)` ， *其中*
    - `ni` 如果 `si` 和 `ti` 不同，或者两者都不存在，则不存在
    - `ni` 是 `si` `si` 和 `ti` 是否相同
- *Merge* (`object` ， `dynamic`) = *merge* (`dynamic` ， `object`) = `dynamic`

## <a name="warnings"></a>警告

### <a name="potential-null-assignment"></a>潜在 null 赋值

### <a name="potential-null-dereference"></a>潜在的空取消引用

### <a name="constraint-nullability-mismatch"></a>约束为空性不匹配

### <a name="nullable-types-in-disabled-annotation-context"></a>禁用的注释上下文中可以为 null 的类型

### <a name="override-and-implementation-nullability-mismatch"></a>重写和实现为空性不匹配

## <a name="attributes-for-special-null-behavior"></a>特殊 null 行为的特性

