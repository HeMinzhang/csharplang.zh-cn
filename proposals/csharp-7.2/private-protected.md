---
ms.openlocfilehash: 6cf489595654236c18edee94c0af380e605c9571
ms.sourcegitcommit: f61a06970fa0562d2e40363fae3948eb168624ca
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/14/2020
ms.locfileid: "79484074"
---
# <a name="private-protected"></a>私有受保护

* [x] 建议
* [x] 原型：[完成](https://github.com/dotnet/roslyn/blob/master/docs/features/private-protected.md)
* [x] 实现：[完成](https://github.com/dotnet/roslyn/blob/master/docs/features/private-protected.md)
* [x] 规范：[完成](#detailed-design)

## <a name="summary"></a>总结
[summary]: #summary

以 `private protected`的C#形式公开 CLR `protectedAndInternal` 的可访问性级别。

## <a name="motivation"></a>动机
[motivation]: #motivation

在许多情况下，API 包含的成员只应由提供该类型的程序集中包含的子类实现和使用。 虽然 CLR 为此目的提供了一个可访问性级别，但它在C#中不可用。 因此，API 所有者被迫使用 `internal` 保护和自助式或自定义分析器，或将 `protected` 与其他文档一起使用，以说明该成员出现在该类型的公共文档中，而不是作为公共 API 的一部分。  有关后者的示例，请参阅名称以 `Common`开头的 Roslyn `CSharpCompilationOptions` 的成员。

在中C#直接提供对此访问级别的支持可使这些情况以语言自然表达。

## <a name="detailed-design"></a>详细设计
[design]: #detailed-design

### <a name="private-protected-access-modifier"></a>`private protected` 访问修饰符

建议添加新的访问修饰符组合 `private protected` （可在修饰符中以任意顺序出现）。 这会映射到 protectedAndInternal 的 CLR 概念，并借用当前在[ C++/cli](https://docs.microsoft.com/cpp/dotnet/how-to-define-and-consume-classes-and-structs-cpp-cli#BKMK_Member_visibility)中使用的语法。

如果 `private protected` 声明的成员，则可以在其容器的子类中访问该成员，前提是该子类与成员在同一程序集中。

修改语言规范，如下所示（以粗体添加）。 节号不如下所示，具体取决于它所集成到的规范版本。

-----

> 成员的声明可访问性可以是以下项之一：
- Public，通过在成员声明中包含一个公共修饰符来选择。 Public 的直观含义是 "访问不受限制"。
- Protected：通过在成员声明中包括受保护的修饰符来选择。 受保护的直观含义为 "访问限制为包含类或派生自包含类的类型"。
- 内部，通过在成员声明中包含内部修饰符来选择。 内部的直观含义为 "访问限制为此程序集"。
- 受保护的内部，通过将受保护的和内部修饰符同时包含在成员声明中来选择。 受保护的内部的直观含义是 "在此程序集内可访问，以及派生自包含类的类型"。
- **私有受保护的，它通过在成员声明中同时包含 private 修饰符和 protected 修饰符来选择。"私有" 受保护的直观含义是 "在此程序集中按派生自包含类的类型进行访问"。**

-----

> 根据成员声明发生的上下文，仅允许某些类型的声明的可访问性。 此外，当成员声明中不包含任何访问修饰符时，在其中进行声明的上下文会确定默认的声明的可访问性。 
- 命名空间隐式具有公开声明的可访问性。 命名空间声明中不允许使用访问修饰符。
- 直接在编译单元或命名空间中声明的类型（而不是在其他类型中）可具有公共或内部声明的可访问性，并且默认为内部声明的可访问性。
- 类成员可以具有五种声明的可访问性，并且默认为私有声明的可访问性。 [注意：声明为类成员的类型可以具有五种声明可访问性中的任何一种，而声明为命名空间成员的类型只能具有公共或内部声明的可访问性。 结束说明]
- 结构成员可具有公共、内部或私有声明的可访问性，并且默认为私有声明的可访问性，因为结构是隐式密封的。 结构中引入的结构成员（即，不是由该结构继承）无法具有受保护*的、* ~~受保护~~的内部或受保护的**私有**可访问性。 [注意：声明为结构成员的类型可以具有公共、内部或私有声明的可访问性，而声明为命名空间成员的类型只能具有公共或内部声明的可访问性。 结束说明]
- 接口成员隐式具有公开声明的可访问性。 接口成员声明中不允许使用访问修饰符。
- 枚举成员隐式具有公开声明的可访问性。 枚举成员声明中不允许使用访问修饰符。

-----

> 在程序 P 中，在类型 T 中声明的嵌套成员 M 的可访问域定义如下（注意： M 本身可能是一个类型）：
- 如果 M 的声明的可访问性是公共的，则 M 的可访问域是 T 的可访问域。
- 如果 M 的声明的可访问性是受保护的，则让 D 成为 P 的程序文本和从 T 派生的任何类型的程序文本的并集。M 的可访问域是 T 的可访问域与 D 的交集。
- **如果 "M" 的声明的可访问性受私有保护，则将 D 作为程序文本 P 和从 T 派生的任何类型的程序文本的交集。M 的可访问域是 T 的可访问域与 D 的交集。**
- 如果 "M" 的声明的可访问性受到保护，则将 D 的程序文本和从 T 派生的任何类型的程序文本合并。M 的可访问域是 T 的可访问域与 D 的交集。
- 如果 M 的声明可访问性为 internal，则 M 的可访问域是 T 的可访问域与 P 的程序文本的交集。
- 如果 M 的声明的可访问性是私有的，则 M 的可访问域是 T 的程序文本。

-----

> 当受保护**或**受保护的私有实例成员在声明它的类的程序文本之外访问时，以及当受保护的内部实例成员在声明它的程序的程序文本的外部访问时，应在从其声明的类派生的类声明中进行访问。 此外，需要通过派生类类型的实例或从其构造的类类型进行访问。 此限制可防止一个派生类访问其他派生类的受保护成员，即使这些成员是从同一个基类继承时也是如此。

-----

> 允许的访问修饰符和类型声明的默认访问权限取决于声明发生的上下文（第9.5.2）：
- 在编译单元或命名空间中声明的类型可以具有公共或内部访问权限。 默认值为 "内部访问"。
- 类中声明的类型可以具有公共、受保护的内部、受保护的**内部、受**保护、内部或私有访问权限。 默认值为 "专用访问"。
- 结构中声明的类型可以具有公共、内部或私有访问权限。 默认值为 "专用访问"。

-----

> 静态类声明受到下列限制：
- 静态类不应包含 sealed 修饰符或 abstract 修饰符。 （但是，由于无法对静态类进行实例化或派生，因此它的行为就像它既是密封的又是抽象的。）
- 静态类不应包括类基规范（第16.2.5），并且不能显式指定基类或已实现接口的列表。 静态类隐式继承自类型对象。
- 静态类只应包含静态成员（第16.4.8）。 [注意：所有常量和嵌套类型都分类为静态成员。 结束说明]
- 静态类不应具有受保护 **、私有保护**或受保护的内部声明可访问性的成员。

> 这是一种编译时错误，违反其中的任何限制。 

-----

> 类成员声明可以具有~~五~~**种可能类型**的声明的可访问性（第9.5.2）中的任意一种：公共、**私有受保护**、受保护的内部、受保护、内部或私有。 除了受保护的内部**和私有保护**组合**以外**，还可以指定多个访问修饰符，这是编译时错误。 当类成员声明不包括任何访问修饰符时，将假定 private。

-----

> 非嵌套类型可以具有公共或内部声明的可访问性，并且默认情况下具有内部声明的可访问性。 嵌套类型也可以具有这些形式的声明的可访问性，还可以包含一个或多个声明的可访问性的其他形式，具体取决于包含类型是否为类或结构：
- 在类中声明的嵌套类型可以具有~~五~~**种**声明的可访问性（公共、**私有受保护**、受保护的内部、受保护、内部或私有）形式的任何形式，与其他类成员一样，默认情况下为私有声明的可访问性。
- 在结构中声明的嵌套类型可以有三种形式的声明的可访问性（public、internal 或 private），与其他结构成员一样，默认为私有声明的可访问性。

-----

> 重写声明重写的方法被称为替代方法的重写基方法。在 c 类中声明的方法是，通过检查 C 的每个基类类型（从 C 的直接基类类型开始）来确定重写的基方法继续使用每个连续的直接基类类型，直到在给定基类类型中，至少有一个可访问方法与类型参数替换后的 M 具有相同的签名。 为了定位重写的基方法，如果方法是公共的（如果它是受保护的），或者**它是受**保护的（如果是受保护的），或者它是在 C 的同一程序中进行内部**或私有保护**并声明的，则该方法将被视为可访问。

-----

> 访问器修饰符的使用受以下限制的约束：
- 访问器修饰符不能在接口或显式接口成员实现中使用。
- 对于没有 override 修饰符的属性或索引器，仅当属性或索引器同时具有 get 访问器和 set 访问器时才允许使用访问器修饰符，只允许在其中一种访问器上使用。
- 对于包含 override 修饰符的属性或索引器，取值函数应与要重写的访问器的访问器修饰符（如果有）匹配。
- 访问器修饰符应声明比属性或索引器本身的声明的可访问性严格更严格的可访问性。 精确：
  - 如果属性或索引器具有 "公共" 的声明的可访问性，则访问器修饰符可以是**私有保护**的、受保护的内部、受保护的或私有的。
  - 如果属性或索引器具有已声明的受保护的可访问性，则访问器修饰符可以是**私有保护**的、内部、受保护的或私有的。
  - 如果属性或索引器具有内部或受保护的声明的可访问性，则访问器修饰符应为**私有保护或**私有。
  - **如果属性或索引器具有已声明的私有受保护的可访问性，则访问器修饰符应为私有。**
  - 如果属性或索引器具有声明的私有可访问性，则不能使用访问器修饰符。

-----

> 由于结构不支持继承，因此无法保护、**私有受保护**或受保护的内部结构成员的可访问性。

-----

## <a name="drawbacks"></a>缺点
[drawbacks]: #drawbacks

与任何语言功能一样，我们必须回答对语言的额外复杂性是否偿还，以提供给将从功能中受益的C#程序的主体。

## <a name="alternatives"></a>备选项
[alternatives]: #alternatives

替代方法是将属性与分析器组合在一起。 特性由程序员放置在一个 `internal` 成员上，以指示该成员仅用于子类中，而分析器会检查这些限制是否已服从。 

## <a name="unresolved-questions"></a>未解决的问题
[unresolved]: #unresolved-questions

实现大致完成。 唯一打开的工作项将为 VB 起草相应规范。

## <a name="design-meetings"></a>设计会议

TBD