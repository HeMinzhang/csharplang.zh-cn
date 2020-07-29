---
ms.openlocfilehash: 9cfc0758f16b2153d52faec1d19f0ecd817cde3b
ms.sourcegitcommit: 0c25406d8a99064bb85d934bb32ffcf547753acc
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/28/2020
ms.locfileid: "87297253"
---
# <a name="covariant-return-types"></a>协变返回类型

* [x] 建议
* [] 原型：未启动
* [] 实现：未启动
* [X] 规范：未启动

## <a name="summary"></a>摘要
[summary]: #summary

支持_协变返回类型_。 具体而言，允许重写方法以返回比它重写的方法更多的派生返回类型，同样，允许重写只读属性以返回派生程度更高的返回类型。 方法或属性的调用方将从调用中静态接收更精确的返回类型，并且在更多派生的类型中出现的重写将要求至少提供一个返回类型，使其在其基类型的重写中出现。

## <a name="motivation"></a>动机
[motivation]: #motivation

代码中的一种常见模式是，必须使用不同的方法名称来解决语言约束，而重写必须返回与重写的方法相同的类型。

这在工厂模式下非常有用。 例如，在 Roslyn 代码库中，我们将

``` cs
class Compilation ...
{
    virtual Compilation WithOptions(Options options)...
}
```

``` cs
class CSharpCompilation : Compilation
{
    override CSharpCompilation WithOptions(Options options)...
}
```

## <a name="detailed-design"></a>详细设计
[design]: #detailed-design

这是在 c # 中建议用于[协变返回类型](https://github.com/dotnet/csharplang/issues/49)的建议。  我们的目的是允许重写方法以返回比它重写的方法更派生的返回类型，同样，允许重写只读属性以返回派生程度更高的返回类型。  方法或属性的调用方将从调用中静态接收更精确的返回类型，并且在更多派生的类型中出现的重写将要求至少提供一个返回类型，使其在其基类型的重写中出现。

这是第一次起草，因此它必须从头开始。  引入的许多创意都是暂定的，可能会在将来的修订版中修改或消除。

--------------

### <a name="class-method-override"></a>类方法重写

[类重写方法上的现有约束](https://github.com/dotnet/csharplang/blob/master/spec/classes.md#override-methods)

> - 重写方法和重写的基方法具有相同的返回类型。

已修改为

> - 重写方法必须有一个返回类型，该类型可通过标识或隐式引用转换转换为重写基方法的返回类型。

以下附加要求将追加到该列表：

> - 重写方法必须有一个返回类型，该返回类型可通过标识或隐式引用转换转换为重写方法的（直接或间接）基类型中声明的重写基方法的返回类型。
> - 重写方法的返回类型必须至少具有与 override 方法（[可访问域](https://github.com/dotnet/csharplang/blob/master/spec/basic-concepts.md#accessibility-domains)）相同的可访问性。

此约束允许类中的重写方法 `private` 具有 `private` 返回类型。  但是，它要求 `public` 类型中的重写方法 `public` 具有 `public` 返回类型。

### <a name="class-property-and-indexer-override"></a>类属性和索引器替代

[类重写属性的现有约束](https://github.com/dotnet/csharplang/blob/master/spec/classes.md#virtual-sealed-override-and-abstract-property-accessors)

> 重写属性声明应将完全相同的可访问性修饰符和名称指定为继承的属性，并且应在~~重写的类型和继承的属性之间~~进行标识转换。 如果继承的属性只有一个访问器（即，如果继承的属性是只读的或只写的），则重写属性应仅包含该访问器。 如果继承的属性包含两个访问器（即，如果继承的属性是读写的），则重写属性可以包含单个访问器或两个访问器。

已修改为

> 重写属性声明应将完全相同的可访问性修饰符和名称指定为继承的属性，并且应**从重写属性的类型到继承属性的类型的隐式引用转换（如果继承的属性是只读的）隐式引用转换**。 如果继承的属性只有一个访问器（即，如果继承的属性是只读的或只写的），则重写属性应仅包含该访问器。 如果继承的属性包含两个访问器（即，如果继承的属性是读写的），则重写属性可以包含单个访问器或两个访问器。 **重写属性的类型必须至少具有与重写属性（[可访问域](https://github.com/dotnet/csharplang/blob/master/spec/basic-concepts.md#accessibility-domains)）相同的可访问性。**

***下面的草案规范的其余部分建议更进一步扩展接口方法的协变返回，以备稍后考虑。***

### <a name="interface-method-property-and-indexer-override"></a>接口方法、属性和索引器替代

使用 c # 8.0 添加 DIM 功能，将添加到接口中允许的成员种类，进一步增加了对成员的支持 `override` 以及协变返回。  这些规则按照为 `override` 类指定的成员规则进行，但存在以下差异：

类中的以下文本：

> 重写声明重写的方法称为***重写基方法***。 对于 `M` 在类中声明的重写方法 `C` ，重写的基方法是通过检查的每个基类确定的，该方法 `C` 从的直接基类开始， `C` 然后继续使用每个连续的直接基类，直到至少有一个可访问方法与 `M` 类型参数替换后具有相同签名的可访问方法。

为提供接口的相应规范：

> 重写声明重写的方法称为***重写基方法***。 对于 `M` 在接口中声明的重写方法 `I` ，重写的基方法是通过检查的每个直接或间接基接口确定的 `I` ，收集一组接口，该接口声明一个具有与 `M` 类型参数替换相同的签名的可访问方法。  如果这组接口具有派生程度*最高的类型*，则为该集合中的每个类型都有一个标识或隐式引用转换，并且该类型包含一个唯一的此类方法声明，即*重写的基方法*。

同样，允许 `override` 接口中的属性和索引器在*15.7.6 Virtual、sealed、override 和 abstract 访问器*中为类指定。

### <a name="name-lookup"></a>名称查找

存在类声明的名称查找 `override` 当前修改名称查找的结果，方法是在从 `override` 标识符的限定符的类型（或没有限定符时）开始，从类层次结构中派生程度最高的声明施加找到成员的详细信息 `this` 。  例如，在*12.6.2.2*中，有

> 对于在类中定义的虚拟方法和索引器，将从从接收方静态类型开始时找到的函数成员的第一个声明或重写中选取参数列表，并搜索其基类。

为此，我们将

> 对于在接口中定义的虚拟方法和索引器，将从派生程度最大的类型中找到的函数成员的声明或重写中选取参数列表，这些类型包含函数成员的重写声明。  如果不存在任何唯一的此类类型，则会发生编译时错误。

对于属性或索引器访问的结果类型，现有文本

> - 如果 I 标识实例属性，则结果是具有关联的实例表达式 E 和关联类型（属性的类型）的属性访问。 如果 T 是类类型，则将从从 T 开始时找到的属性的第一个声明或重写中选取关联的类型，并搜索其基类。

扩充了

> 如果 T 是接口类型，则会从在 T 的派生或其直接或间接基接口中找到的属性的声明或重写中选取关联的类型。  如果不存在任何唯一的此类类型，则会发生编译时错误。

应在*12.7.7.3 索引器访问*中进行类似的更改

在*12.7.6 调用表达式*中，我们增加了现有文本

> - 否则，结果为具有方法或委托返回类型的关联类型的值。 如果调用的是实例方法，并且接收方属于类类型 T，则会从第一个声明中选取关联的类型，或者在从 T 开始并搜索其基类时从找到的方法中选取关联的类型。

替换为

> 如果调用的是实例方法，并且接收方是接口类型 T，则会从 T 及其直接和间接基接口的派生接口中找到的方法的声明或重写来选取关联的类型。  如果不存在任何唯一的此类类型，则会发生编译时错误。

### <a name="implicit-interface-implementations"></a>隐式接口实现

规范的此部分

> 出于接口映射的目的，类成员将 `A` 在以下情况下匹配接口成员 `B` ：
> 
> - `A`和 `B` 是方法，且和的名称、类型和形参列表 `A` `B` 完全相同。
> - `A`和 `B` 是属性，和的名称和类型 `A` `B` 相同，并且 `A` 具有与相同的访问器 `B` （ `A` 如果它不是显式接口成员实现，则允许具有其他访问器）。
> - `A`和 `B` 是事件，而和的名称和类型 `A` `B` 是相同的。
> - `A`和 `B` 为索引器，和的类型参数列表和形参列表 `A` `B` 相同，并且 `A` 具有与相同的访问器 `B` （ `A` 如果它不是显式接口成员实现，则允许具有其他访问器）。

按如下所示进行修改：

> 出于接口映射的目的，类成员将 `A` 在以下情况下匹配接口成员 `B` ：
> 
> - `A`和 `B` 是方法，且和的名称和形参列表相同，并且的返回类型可 `A` `B` `A` `B` 通过隐式引用转换的标识转换为返回类型 `B` 。
> - `A`和 `B` 是属性、和的名称 `A` `B` 相同，具有与 `A` 相同的访问器 `B` （ `A` 如果它不是显式接口成员实现，则允许具有其他访问器）; 的类型 `A` 可以转换为通过标识转换的返回类型， `B` 或者如果 `A` 是一个只读属性，则为隐式引用转换。
> - `A`和 `B` 是事件，而和的名称和类型 `A` `B` 是相同的。
> - `A`和 `B` 为索引器，和的形参列表 `A` `B` 相同， `A` 具有与相同的访问器 `B` （ `A` 如果它不是显式接口成员实现，则允许具有其他访问器）; 的类型 `A` 可以转换为通过标识转换的返回类型， `B` 或（如果 `A` 是只读索引器）隐式引用转换。

从技术上讲，这是一项重大更改，如以下程序所示： C1。"今天，但会打印" C2。"M"。

``` c#
using System;

interface I1 { object M(); }
class C1 : I1 { public object M() { return "C1.M"; } }
class C2 : C1, I1 { public new string M() { return "C2.M"; } }
class Program
{
    static void Main()
    {
        I1 i = new C2();
        Console.WriteLine(i.M());
    }
}
```

由于此重大更改，我们可能会认为不支持隐式实现的协变返回类型。

### <a name="constraints-on-interface-implementation"></a>接口实现的约束

**我们将需要一条规则，指出显式接口实现必须声明一个返回类型，该返回类型的派生程度不小于在其基接口中的任何重写中声明的返回类型。**

### <a name="api-compatibility-implications"></a>API 兼容性问题

*TBD*

### <a name="open-issues"></a>未结的问题

该规范并不表示调用方如何获取更精确的返回类型。 可能会以类似于调用方获取最常获得的重写参数规范的方式来完成。

--------------

如果有以下接口：

```csharp
interface I1 { I1 M(); }
interface I2 { I2 M(); }
interface I3: I1, I2 { override I3 M(); }
```

请注意，在中 `I3` ，方法 `I1.M()` 和已 `I2.M()` "合并"。  实现时 `I3` ，必须同时实现它们。

通常，我们需要显式实现来引用原始方法。  问题在于，在类中

```csharp
class C : I1, I2, I3
{
    C IN.M();
}
```

这是什么意思？  *N*应该是什么？

我建议您允许实现 `I1.M` 或 `I2.M` （但不是两者），并将其视为实现两者。

## <a name="drawbacks"></a>缺点
[drawbacks]: #drawbacks

- [] 每个语言更改都必须为其本身付费。
- [] 我们应该确保性能合理，甚至在深层继承层次结构中也是如此
- [] 我们应该确保翻译策略的项目不会影响语言语义，即使在使用旧编译器的新 IL 时也是如此。

## <a name="alternatives"></a>备选项
[alternatives]: #alternatives

我们可以略微放宽语言规则，在源中，

```csharp
abstract class Cloneable
{
    public abstract Cloneable Clone();
}

class Digit : Cloneable
{
    public override Cloneable Clone()
    {
        return this.Clone();
    }

    public new Digit Clone() // Error: 'Digit' already defines a member called 'Clone' with the same parameter types
    {
        return this;
    }
}
```

## <a name="unresolved-questions"></a>未解决的问题
[unresolved]: #unresolved-questions

- [] 编译为使用此功能的 Api 将在较旧版本的语言中工作？

## <a name="design-meetings"></a>设计会议

- 一些讨论 <https://github.com/dotnet/roslyn/issues/357> 。
- https://github.com/dotnet/csharplang/blob/master/meetings/2020/LDM-2020-01-08.md
- 针对决策的离线讨论，以支持仅在 c # 9.0 中重写类方法。

