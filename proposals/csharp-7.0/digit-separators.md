---
ms.openlocfilehash: 5476f4438ad79a26b3615154f789d8ed04cb61aa
ms.sourcegitcommit: 94a3d151c438d34ede1d99de9eb4ebdc07ba4699
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/25/2019
ms.locfileid: "79483498"
---
# <a name="digit-separators"></a>数字分隔符

能够在大数值中对数字进行分组会给您带来很大的可读性影响，并且没有明显的缺点。 

添加二进制文本（#215）会增加数值文本的可能性，因此这两个功能互相增强。 

我们将遵循 Java 和其他方法，并使用下划线 `_` 作为数字分隔符。 它可以在数字文本中的任何位置发生（除了第一个字符和最后一个字符），因为不同的分组在不同方案中可能有意义，特别是对于不同的数字基准：

```csharp
int bin = 0b1001_1010_0001_0100;
int hex = 0x1b_a0_44_fe;
int dec = 33_554_432;
int weird = 1_2__3___4____5_____6______7_______8________9;
double real = 1_000.111_1e-1_000;
```

任何数字序列都可以用下划线分隔，两个连续数字之间可能有多个下划线。 它们在小数和指数中是允许的，但在前面的规则后，它们可能不会出现在指数字符（`1.1e_1`）旁边或类型说明符（`10_f`）旁的小数点（`10_.0`）的旁边。 在二进制和十六进制文本中使用时，它们可能不会立即出现在 `0x` 或 `0b`之后。

语法非常简单，分隔符没有语义影响，它们只是被忽略。

这有很大的价值，并且很容易实现。
