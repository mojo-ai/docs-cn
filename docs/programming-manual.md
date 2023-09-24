# Mojo🔥编程手册

Mojo 是一种与 Python 一样易于使用的编程语言，但具有 C++ 和 Rust 的性能。此外，Mojo 还提供了利用整个 Python 库生态系统的能力。

Mojo 通过利用具有集成缓存、多线程和云分发技术的下一代编译器技术来实现这一功能。此外，Mojo 的自动调整和编译时元编程功能允许您将编写的代码移植到不同硬件上。

更重要的是，**Mojo 允许您利用整个 Python 生态系统**，以便您可以继续使用您熟悉的工具。Mojo 在通过保留 Python 的动态功能同时，添加新的[系统编程](https://en.wikipedia.org/wiki/Systems_programming)原语，随着时间的推移成为 Python 的**超集**。这些新的系统编程原语将允许 Mojo 开发人员构建高性能库，这些库以前可能需要 C、C++、Rust、CUDA 和其他加速器系统构建。通过汇集最好的动态语言和系统语言，我们希望提供一个**统一的**跨抽象级别工作的编程模型，对新手程序员友好，并且可以扩展到从加速器到应用程序编程和脚本编写的许多用例。

本文档是 Mojo 编程语言的介绍，而不是完整的语言指南。它假定您了解 Python 和系统编程概念。目前，Mojo 仍在开发中，其文档面向具有系统编程经验的开发人员。随着该语言的发展和变得更加广泛可用，我们希望它对每个人（包括初学者程序员）都友好且易于使用。只是目前还不行。

## [使用 Mojo 编译器](#using-the-mojo-compiler)

使用[Mojo SDK](https://docs.modular.com/mojo/manual/get-started/)，您可以从终端运行 Mojo 程序，就像使用 Python 一样。因此，如果您有一个名为`hello.mojo`（或`hello.🔥`，是的，文件扩展名可以是表情符号！）的文件，只需键入`mojo hello.mojo`：

```
$ cat hello.🔥
def main():
    print("hello world")
    for x in range(9, 0, -3):
        print(x)
$ mojo hello.🔥
hello world
9
6
3
$
```

同样，您可以使用`.mojo`或`.🔥`后缀。

有关 Mojo 编译器工具的更多详细信息，请参阅[`mojo`CLI 文档](https://docs.modular.com/mojo/cli/)。

## [基本系统编程扩展](#basic-systems-programming-extensions)

考虑到我们的兼容性目标以及 Python 在高级应用程序和动态 API 方面的优势，我们不必花费太多时间来解释该语言的这些部分是如何工作的。另一方面，Python 对系统编程的支持主要委托给 C，我们希望提供一个在该领域非常出色的单一系统。因此，本节详细介绍了每个主要组件和功能，并通过示例描述了如何使用它们。

### [`let`和`var`声明](#let-and-var-declarations)

在 Mojo 的`def`内部，您可以为名称分配一个值，它会隐式创建一个函数作用域变量，就像在 Python 中一样。这提供了一种非常动态且简单的代码编写方式，但它也会造成一些影响，原因有二：

1. 系统程序员通常希望声明一个值是不可变的，以保证类型安全和性能。
2. 如果他们在赋值中输错了变量名，他们可能希望得到一个错误。

为了支持这一点，Mojo 提供了作用域运行时值声明：`let`是不可变的，而`var`是可变的。这些值使用词法作用域并支持名称覆盖：

```
def your_function(a, b):
    let c = a
    # Uncomment to see an error:
    # c = b  # error: c is immutable

    if c != b:
        let d = b
        print(d)

your_function(2, 3)
```

```
3
```

`let`和`var`声明支持类型说明、模式以及后期初始化：

```
def your_function():
    let x: Int = 42
    let y: Float64 = 17.0

    let z: Float32
    if x != 0:
        z = 1.0
    else:
        z = foo()
    print(z)

def foo() -> Float32:
    return 3.14

your_function()
```

```
1.0
```

请注意，在`def`函数中，`let`和`var`是完全可选的（您可以使用隐式声明的值，就像 Python 一样），但对于`fn`函数中的所有变量它们都是必需的。

另请注意，在 REPL 环境（例如本笔记）中使用 Mojo 时，顶级变量（位于函数或结构之外的变量）会被视为`def`中的变量，因此它们允许隐式值类型声明（它们不需要`var`或`let`声明，也不是类型声明）。这与 Python REPL 行为相匹配。

### [`struct`类型](#struct-types)

Mojo 基于 MLIR 和 LLVM，提供了多种编程语言的尖端编译器和代码生成系统。可以更好地控制数据组织、直接访问数据字段以及拥有其他提高性能的方法。现代系统编程语言的一个重要特征是能够在这些复杂的低级操作之上构建高级且安全的抽象，而不会造成任何性能损失。在 Mojo 中，这是由`struct`类型提供的。

Mojo 中的`struct`与 Python 类似`class`：它们都支持方法、字段、运算符重载、元编程装饰器等。它们的区别如下：

* Python 类是动态的：它们允许动态调度、猴子修补（或“swizzling”）以及在运行时动态绑定实例属性。
* Mojo 结构是静态的：它们在编译时绑定（不能在运行时添加方法）。结构允许您以灵活性换取性能，同时安全且易于使用。

这是结构体的简单定义：

```
struct MyPair:
    var first: Int
    var second: Int

    # We use 'fn' instead of 'def' here - we'll explain that soon
    fn __init__(inout self, first: Int, second: Int):
        self.first = first
        self.second = second

    fn __lt__(self, rhs: MyPair) -> Bool:
        return self.first < rhs.first or
              (self.first == rhs.first and
               self.second < rhs.second)
```

从语法上讲，与 Python 相比最大的区别`class`是`struct`中的所有实例属性都 **必须**使用`var`、`let`声明或类型声明。

在Mojo中，`struct`的结构和内容是预先设置的，并且在程序运行时不能更改。与 Python 不同的是，在 Python 中您可以动态添加、删除或更改对象的属性，而 Mojo 不允许对结构进行这种操作。这意味着您不能`del`在程序运行过程中删除方法或更改其值。

然而，静态特性`struct`有一些很大的好处！它可以帮助 Mojo 更快地运行您的代码。程序确切地知道在哪里可以找到结构体的信息以及如何使用它，而无需任何额外的步骤或延迟。

Mojo 的结构也可以很好地与您可能已经从 Python 中了解的功能配合使用，例如运算符重载（让您可以更改 + 和 - 等数学符号如何处理您自己的数据）。此外，_所有_“标准类型”（例如`Int`、`Bool`、`String`甚至`Tuple`）都是使用结构体创建的。这意味着它们是您可以使用的标准工具集的一部分，而不是硬连接到语言本身中。这为您在编写代码时提供了更大的灵活性和控制力。

`self`参数中`inout`的含义：这表明参数是可变的，并且函数内部所做的更改对调用者可见。有关详细信息，请参阅下面有关[inout Arguments](https://docs.modular.com/mojo/programming-manual.html#mutable-arguments-inout)的内容。

#### [`Int`与`int`](#int-vs-int)

在 Mojo 中，您可能会注意到我们使用`Int`（大写“I”），这与 Python `int`（小写“i”）不同。这种差异是故意的，而且实际上是一件好事！

在 Python 中，该`int`类型可以处理非常大的数字，并且具有一些额外的功能，例如检查两个数字是否是同一个对象。但这带来了一些额外的负担，可能会减慢速度。Mojo的`Int`则不同。它的设计简单、快速，并针对您的计算机硬件进行了调整，以便快速处理。

这样做有两个主要原因：

1. 我们希望为深入系统底层硬件的程序员（系统程序员）提供一种透明且可靠的与硬件交互的方式。我们不想依靠花哨的技巧（比如 JIT 编译器）。
2. 我们希望 Mojo 能够与 Python 很好地配合，而不会引起任何问题。通过使用不同的名称（Int 而不是 int），我们可以在 Mojo 中保留这两种类型，而无需更改 Python int 的工作方式。

作为奖励，`Int`遵循的命名风格，与您可能在 Mojo 中创建的其他自定义数据类型相同。此外，`Int`本身是`struct`，包含在 Mojo 的标准工具集中。

### [强类型检查](#strong-type-checking)

尽管您仍然可以像在 Python 中一样使用灵活的类型，但 Mojo 允许您使用严格的类型检查。类型检查可以使您的代码更可预测、更易于管理且更安全。

使用强类型检查的主要方法之一是使用 Mojo 的`struct`类型。Mojo 中的定义`struct`定义了一个编译时绑定的名称，并且对所定义的值来说，在类型上下文中对该名称的引用被视为强规范。例如，考虑以下代码，使用上述结构`MyPair`：

```
def pair_test() -> Bool:
    let p = MyPair(1, 2)
    # Uncomment to see an error:
    # return p < 4 # gives a compile time error
    return True
```

如果取消注释第一个 return 语句并运行它，您将收到一个编译时错误，告诉您无法`4`转换为`MyPair`，这是`__lt__()`右侧所需要的（在`MyPair`定义中）。

在使用系统编程语言时，这种做法很熟悉，但 Python 并不是这样工作的。Python 对于[MyPy](https://mypy.readthedocs.io/)类型注释具有语法相同的功能，但它们不是由编译器强制执行的：相反，它们是通知静态分析的提示。通过将类型绑定到特定声明，Mojo 可以处理经典类型注释提示和强类型规范，而不会破坏兼容性。

类型检查并不是强类型的唯一用例。由于我们知道类型是准确的，因此我们可以根据这些类型优化代码，在寄存器中传递值，并在参数传递和其他低级细节方面保持与 C 一样高效。这是 Mojo 为系统程序员提供安全性和可预测性保证的基础。

### [重载的函数和方法](#overloaded-functions-and-methods)

与 Python 一样，您可以在 Mojo 中定义函数而无需指定参数数据类型，Mojo 将动态处理它们。当您想要富有表现力的 API ，接受任意输入并让动态调度决定如何处理数据时，这很好。然而，当您想要确保类型安全时，如上所述，Mojo 还提供对重载函数和方法的全面支持。

这允许您定义多个具有相同名称但具有不同参数的函数。这是许多语言（例如 C++、Java 和 Swift）中的常见功能。

解析函数调用时，Mojo 会尝试每个候选函数并使用有效的一个（如果只有一个有效），或者选择最接近的匹配（如果可以确定一个接近的匹配），或者报告调用不明确如果不知道该选哪一个。在后一种情况下，您可以通过在调用站点上添加显式转换来解决歧义。

让我们看一个例子：

```
struct Complex:
    var re: Float32
    var im: Float32

    fn __init__(inout self, x: Float32):
        """Construct a complex number given a real number."""
        self.re = x
        self.im = 0.0

    fn __init__(inout self, r: Float32, i: Float32):
        """Construct a complex number given its real and imaginary components."""
        self.re = r
        self.im = i
```

您可以重载结构和类中的方法以及重载模块级函数。

Mojo 不支持仅在结果类型上重载，并且不使用结果类型或上下文类型信息进行类型推断，从而使事情保持简单、快速和可预测。Mojo 永远不会产生“表达式太复杂”错误，因为它的类型检查器是简单且快速的。

同样，如果您的参数名称不带类型定义，则该函数的行为就像具有动态类型的 Python 一样。一旦定义了单个参数类型，Mojo 将查找重载函数并解析函数调用，如上所述。

虽然我们还没有讨论参数（它们与函数参数不同），但您也可以[基于参数重载函数和方法](https://docs.modular.com/mojo/programming-manual.html#overloading-on-parameters)。

### [`fn`定义](#fn-definitions)

上述扩展是提供低级编程和抽象功能的基石，但许多系统程序员喜欢更多可控性和可预测性，相比与 Mojo 中`def`所提供的。回顾一下，`def`它的定义必须非常动态、灵活并且通常与 Python 兼容：参数是可变的，局部变量在第一次使用时隐式声明，并且不强制执行作用域。这对于高级编程和脚本编写来说非常有用，但对于系统编程来说并不总是那么好。为了补充这一点，Mojo 提供了一个`fn`声明，类似于`def`的严格模式.

> 替代方案：我们可以添加修饰符或装饰器类似于`@strict def`，而不是使用像`fn`这样的新关键字。然而，无论如何我们都需要采用新的关键词，而且这样做有一定成本。此外，在系统编程领域的实践中，`fn`它一直被使用，因此使其成为第一选择。

就调用者而言，`fn`和`def`是可以互换的：没有什么是 `def`可以提供而 `fn`不能提供的（反之亦然）。不同之处在于，在`fn`_内部_受到更多限制和控制（或者：迂腐和严格）。具体来说，`def`与函数相比，`fn`有许多限制：

1. 参数值默认在函数体内是不可变的（像`let`一样），而不是可变的（像`var`一样）。这会捕获意外突变，并允许使用不可复制的类型作为参数。
2. 参数值需要类型规范（`self`方法中除外），以捕获类型规范的意外遗漏。同样，缺少返回类型说明符会被解释为返回`None`而不是未知的返回类型。请注意，两者都可以显式声明为返回`object`，这允许人们根据需要选择加入`def`。
3. 局部变量的隐式声明被禁用，因此必须声明所有局部变量。这可以捕获名称拼写错误并与`let`和`var`提供的范围相吻合。
4. 两者都支持引发异常，但`fn`必须使用`raises`关键字显式声明。

不同团队的编程模式会有很大差异，这种严格程度并不适合所有人。我们希望习惯 C++ 并已在 Python 中使用 MyPy 风格类型注释的人们更喜欢使用`fn`，但更高级别的程序员和 ML 研究人员将继续使用`def`. Mojo 允许您自由地混合`def`和`fn`声明，例如用一种方法实现某些方法，用另一种方法实现另一些方法，并允许每个团队或程序员决定什么最适合他们的用例。

[有关 Mojo 函数中参数行为的更多信息，请参阅下面有关参数传递控制和内存所有权](https://docs.modular.com/mojo/programming-manual.html#argument-passing-control-and-memory-ownership)的部分。

### [`copyinit`、`moveinit`和`takeinit`特殊方法](#the-copyinit-moveinit-and-takeinit-special-methods)

Mojo 支持完整的“值语义”，如 C++ 和 Swift 等语言中所示，并且它使得定义简单的字段聚合变得非常容易通过[`@value`装饰器](https://docs.modular.com/mojo/programming-manual.html#value-decorator)。

对于高级用例，Mojo 允许您定义自定义构造函数（使用 Python 现有的`__init__`特殊方法） 、自定义析构函数（使用现有的`__del__`特殊方法）以及自定义复制和移动构造函数（使用`__copyinit__`、`__moveinit__`和`__takeinit__`特殊方法）。

这些低级定制挂钩函数在进行低级系统编程（例如手动内存管理）时非常有用。例如，考虑一个动态字符串类型，它需要在构造时为字符串数据分配内存，并在销毁值时销毁它：

```
from memory.unsafe import Pointer

struct HeapArray:
    var data: Pointer[Int]
    var size: Int

    fn __init__(inout self, size: Int, val: Int):
        self.size = size
        self.data = Pointer[Int].alloc(self.size)
        for i in range(self.size):
            self.data.store(i, val)
     
    fn __del__(owned self):
        self.data.free()

    fn dump(self):
        print_no_newline("[")
        for i in range(self.size):
            if i > 0:
                print_no_newline(", ")
            print_no_newline(self.data.load(i))
        print("]")
```

该数组类型是使用低级函数实现的，以展示其工作原理的简单示例。但是，如果您尝试使用`=运算符复制`HeapArray`的实例`，您可能会感到惊讶：

```
var a = HeapArray(3, 1)
a.dump()   # Should print [1, 1, 1]
# Uncomment to see an error:
# var b = a  # ERROR: Vector doesn't implement __copyinit__

var b = HeapArray(4, 2)
b.dump()   # Should print [2, 2, 2, 2]
a.dump()   # Should print [1, 1, 1]
```

```
[1, 1, 1]
[2, 2, 2, 2]
[1, 1, 1]
```

如果您取消注释将`a`复制到`b`的行，您将看到 Mojo 不允许您复制的数组：`HeapArray`包含一个`Pointer`实例（相当于低级 C 指针），而 Mojo 不允许知道它指向什么类型的数据或如何复制它。更常见的说，某些类型（如原子序数）无法复制或移动，因为它们的地址提供了一个**身份**，就像类实例一样。

在这种情况下，我们确实希望我们的数组是可复制的。为了实现这一点，我们必须实现`__copyinit__`特殊的方法，通常是这样实现的：

```
struct HeapArray:
    var data: Pointer[Int]
    var size: Int

    fn __init__(inout self, size: Int, val: Int):
        self.size = size
        self.data = Pointer[Int].alloc(self.size)
        for i in range(self.size):
            self.data.store(i, val)

    fn __copyinit__(inout self, other: Self):
        self.size = other.size
        self.data = Pointer[Int].alloc(self.size)
        for i in range(self.size):
            self.data.store(i, other.data.load(i))
            
    fn __del__(owned self):
        self.data.free()

    fn dump(self):
        print_no_newline("[")
        for i in range(self.size):
            if i > 0:
                print_no_newline(", ")
            print_no_newline(self.data.load(i))
        print("]")
```

通过此实现，我们上面的代码可以正常工作，并且`b = a`副本会生成逻辑上不同的数组实例，并具有自己的生命周期和数据：

```
var a = HeapArray(3, 1)
a.dump()   # Should print [1, 1, 1]
# This is no longer an error:
var b = a

b.dump()   # Should print [1, 1, 1]
a.dump()   # Should print [1, 1, 1]
```

```
[1, 1, 1]
[1, 1, 1]
[1, 1, 1]
```

Mojo 还支持`__moveinit__`方法允许 Rust 风格的移动（当源生命周期结束时将值从一个地方传输到另一个地方）和`__takeinit__`C++ 风格的移动方法（其中值的内容从源中传输出来，但其析构函数仍在运行），并允许自定义移动逻辑。有关更多详细信息，请参阅下面的[“价值生命周期”](https://docs.modular.com/mojo/programming-manual.html#value-lifecycle-birth-life-and-death-of-a-value)部分。

Mojo 提供对值的生命周期的完全控制，包括使类型可复制、仅移动和不可移动的能力。这比 Swift 和 Rust 等语言提供的控制能力更强，后者要求值至少是可移动的。如果您好奇如何在不创建副本的情况下将`existing`传递到`__copyinit__`方法中，请查看下面[有关借用参数的](https://docs.modular.com/mojo/programming-manual.html#immutable-arguments-borrowed)部分。

## [参数传递控制和内存所有权](#argument-passing-control-and-memory-ownership)

在 Python 和 Mojo 中，大部分语法都围绕函数调用：许多内置行为是在标准库中使用“dunder”方法实现的。在这些神奇的函数内部，许多内存所有权是通过参数传递来确定的。

让我们回顾一下有关 Python 和 Mojo 如何传递参数的一些细节：

* 所有传递到_Python_ `def`函数的值都使用引用语义。这意味着该函数可以修改传递给它的可变对象，并且这些更改在函数外部可见。然而，这种行为有时会让外行人感到惊讶，因为您可以更改参数指向的对象，并且该更改在函数外部不可见。
* 默认情况下，传递到_Mojo_ `def`函数的所有值都使用值语义。与 Python 相比，这是一个重要的区别：Mojo`def`函数接收所有参数的副本，它可以修改函数内部的参数，但更改在函数外部**不可见。**
* 默认情况下，传递到 Mojo[`fn`函数](https://docs.modular.com/mojo/programming-manual.html#fn-definitions)的所有值都是不可变引用。这意味着该函数可以读取原始对象（它不是_副本_），但它根本无法修改该对象。

这种在 Mojo 中传递不可变参数的约定`fn`称为“借用”。在以下部分中，我们将解释如何更改 Mojo 中`def`和`fn`函数的参数传递行为。

### [为什么论证约定很重要](#why-argument-conventions-are-important)

在 Python 中，所有基本值都是对对象的引用，如上所述，Python 函数可以修改原始对象。因此，Python 开发人员习惯于将一切都视为参考语义。然而，在 CPython 或机器级别，您可以看到引用本身实际上是*通过复制*传递的：Python 复制指针并调整引用计数。

这种 Python 方法为大多数人提供了一个舒适的编程模型，但它要求所有值都进行堆分配（由于引用共享，结果有时会令人惊讶）。Mojo 类（将来会实现）对大多数对象遵循相同的引用语义方法，但这对于系统编程上下文中的整数等简单类型来说并不实用。在这些场景中，我们希望这些值存在于堆栈中，甚至存在于硬件寄存器中。因此，Mojo 结构总是内联到其容器中，无论是作为另一种类型的字段还是包含函数的堆栈帧。

这提出了一些有趣的问题：如何实现需要改变`self`结构类型的方法，例如`__iadd__`？`let`是如何工作的以及如何防止突变？如何控制这些值的生命周期以使 Mojo 成为内存安全的语言？

答案是，Mojo 编译器使用数据流分析和类型注释来提供对值副本、引用别名和突变控制的完全控制。这些功能在很多方面与 Rust 语言中的功能相似，但它们的工作方式有所不同，以便使 Mojo 更容易学习，并且它们可以更好地集成到 Python 生态系统中，而不需要大量的注释负担。

在以下部分中，您将了解如何控制传递到 Mojo`fn`函数的对象的内存所有权。

### [不可变参数( `borrowed` )](#immutable-arguments-borrowed)

借用对象是对函数接收的对象的**不可变引用**，而不是接收该对象的副本。因此，被调用函数具有对该对象的完全读取和执行访问权限，但无法修改它（调用者仍然拥有该对象的独占“所有权”）。

例如，考虑这个结构，我们在传递它的实例时不想复制它：

```
# Don't worry about this code yet. It's just needed for the function below.
# It's a type so expensive to copy around so it does not have a
# __copyinit__ method.
struct SomethingBig:
    var id_number: Int
    var huge: HeapArray
    fn __init__(inout self, id: Int):
        self.huge = HeapArray(1000, 0)
        self.id_number = id

    # self is passed by-reference for mutation as described above.
    fn set_id(inout self, number: Int):
        self.id_number = number

    # Arguments like self are passed as borrowed by default.
    fn print_id(self):  # Same as: fn print_id(borrowed self):
        print(self.id_number)
```

当将`SomethingBig`的实例传递给函数时，有必要传递引用，因为`SomethingBig`无法复制（它没有`__copyinit__`方法）。并且，如上所述，`fn`默认情况下参数是不可变引用，但您可以使用`borrowed`关键字显式定义它，如此处函数所示`use_something_big()`：

```
fn use_something_big(borrowed a: SomethingBig, b: SomethingBig):
    """'a' and 'b' are both immutable, because 'borrowed' is the default."""
    a.print_id()
    b.print_id()

let a = SomethingBig(10)
let b = SomethingBig(20)
use_something_big(a, b)
```

```
10
20
```

此默认值统一适用于所有参数，包括`self`方法的参数。当传递大值或传递很耗资源的值（如引用计数指针）（这是 Python/Mojo 类的默认值）时，这会更有效，因为传递参数时不必调用复制构造函数和析构函数。

因为`fn`函数的默认参数约定是`borrowed`，所以 Mojo 具有简单且符合逻辑的代码，默认情况下可以执行正确的操作。例如，我们不想复制或移动整个`SomethingBig`，仅仅为了调用`print_id()`方法或调用`use_something_big()`.

这种借用参数约定在某些方面类似于 C++ 中的参数传递`const&`，这避免了值的副本并禁止被调用者中修改。然而，借用的约定和 C++ 中`const&`在两个重要方面有所不同：

1. Mojo 编译器实现了一个借用检查器（类似于 Rust），该检查器可以防止代码在存在未完成的不可变引用时动态形成对某个值的可变引用，并且它可以防止对同一值进行多个可变引用。您可以进行多次借用（如`use_something_big`上面的调用所示），但不能通过可变引用传递某些内容并同时借用。（TODO：当前未启用）。
2. 像`Int`、`Float`和`SIMD`这样的小值直接在机器寄存器中传递，而不是通过额外的间接传递（这是因为它们是用装饰器[`@register_passable`](https://docs.modular.com/mojo/programming-manual.html#register_passable-struct-decorator)声明的）。与 C++ 和 Rust 等语言相比，这是一个显着的性能增强，并将这种优化从每个调用站点转移到对类型进行声明。

与 Rust 类似，Mojo 的借用检查器强制执行不变量的排他性。Rust 和 Mojo 之间的主要区别在于，Mojo 不需要调用方有一个印记来传递借用。此外，Mojo 在传递小值时效率更高，而 Rust 默认移动值而不是通过借用传递它们。这些策略和语法决策使 Mojo 能够提供更易于使用的编程模型。

### [可变参数 ( `inout`)](#mutable-arguments-inout)

另一方面，如果您定义一个`fn`函数并希望参数可变，则必须使用`inout`关键字将参数声明为可变的。

**提示：**当您看到时`inout`，这意味着对函数内部参数所做的任何更改在**函数外部**都是可见的。

考虑以下示例，其中`__iadd__`函数（实现自增操作，例如`x += 2`）尝试修改`self`：

```
struct MyInt:
    var value: Int
    
    fn __init__(inout self, v: Int):
        self.value = v
  
    fn __copyinit__(inout self, other: MyInt):
        self.value = other.value
        
    # self and rhs are both immutable in __add__.
    fn __add__(self, rhs: MyInt) -> MyInt:
        return MyInt(self.value + rhs.value)

    # ... but this cannot work for __iadd__
    # Uncomment to see the error:
    #fn __iadd__(self, rhs: Int):
    #    self = self + rhs  # ERROR: cannot assign to self!
```

如果取消注释该`__iadd__()`方法，您将收到编译器错误。

这里的问题是`self` 是不可变的，因为这是一个 Mojo`fn`函数，所以它不能改变参数的内部状态（默认参数约定是`borrowed`）。`inout`解决方案是通过在`self`参数名称上添加关键字来声明参数是可变的：

```
struct MyInt:
    var value: Int
    
    fn __init__(inout self, v: Int):
        self.value = v

    fn __copyinit__(inout self, other: MyInt):
        self.value = other.value
        
    # self and rhs are both immutable in __add__.
    fn __add__(self, rhs: MyInt) -> MyInt:
        return MyInt(self.value + rhs.value)
        
    # ... now this works:
    fn __iadd__(inout self, rhs: Int):
        self = self + rhs
```

现在`self`参数在函数中是可变的，并且任何更改在调用者中都是可见的，因此我们可以使用以下命令执行就地加法`MyInt`：

```
var x: MyInt = 42
x += 1
print(x.value) # prints 43 as expected

# However...
let y = x
# Uncomment to see the error:
# y += 1       # ERROR: Cannot mutate 'let' value
```

```
43
```

如果取消注释上面的最后一行，则`let`值的修改会失败，因为不可能形成对不可变值的可变引用（`let`使变量不可变）。

当然，您可以声明多个`inout`参数。例如，您可以像这样定义和使用交换函数：

```
fn swap(inout lhs: Int, inout rhs: Int):
    let tmp = lhs
    lhs = rhs
    rhs = tmp

var x = 42
var y = 12
print(x, y)  # Prints 42, 12
swap(x, y)
print(x, y)  # Prints 12, 42
```

```
42 12
12 42
```

该系统的一个非常重要的方面是它的所有组成都是正确的。

请注意，我们不将此参数称为“通过引用”传递。尽管`inout`约定在概念上是相同的，但我们不将其称为按引用传递，因为实现实际上可能使用指针传递值。

### [传递参数 (`owned`和`^`)](#transfer-arguments-owned-and)

Mojo 支持的最后一个参数约定是`owned`。此约定用于想要获得值的独占所有权的函数，并且通常与后缀运算`^`符一起使用。

例如，假设您正在使用仅移动类型，例如唯一指针：

```
# This is not really a unique pointer, we just model its behavior here
# to serve the examples below.
struct UniquePointer:
    var ptr: Int
    
    fn __init__(inout self, ptr: Int):
        self.ptr = ptr
    
    fn __moveinit__(inout self, owned existing: Self):
        self.ptr = existing.ptr
        
    fn __del__(owned self):
        self.ptr = 0
```

虽然该`borrow`约定使得无需仪式即可轻松使用这个独特的指针，但在某些时候您可能希望将所有权转移给某些其他函数。在这种情况下，您需要`^`对可移动类型使用“传输”操作符。

运算`^`符结束值绑定的生命周期，并将值所有权转移给其他（在下面的示例中，所有权转移给函数`take_ptr()`）。为了支持这一点，您可以将函数定义为接受`owned`参数。例如，您可以定义`take_ptr()`获取参数的所有权，如下所示：

```
fn take_ptr(owned p: UniquePointer):
    print("take_ptr")
    print(p.ptr)

fn use_ptr(borrowed p: UniquePointer):
    print("use_ptr")
    print(p.ptr)
    
fn work_with_unique_ptrs():
    let p = UniquePointer(100)
    use_ptr(p)    # Pass to borrowing function.
    take_ptr(p^)  # Pass ownership of the `p` value to another function.

    # Uncomment to see an error:
    # use_ptr(p) # ERROR: p is no longer valid here!

work_with_unique_ptrs()
```

```
use_ptr
100
take_ptr
100
```

请注意，如果取消对第二次调用的注释`use_ptr()`，则会收到错误，因为该`p`值已传输到该`take_ptr()`函数，因此该`p`值被销毁。

因为它已被声明`owned`，所以该`take_ptr()`函数知道它具有对该值的唯一访问权。这对于诸如唯一指针之类的事情非常重要，并且当您想要避免复制时它很有用。

例如，您将特别看到`owned`关于析构函数和消耗移动初始值设定项的约定。例如，`HeapArray`结构体在其`__del__()`方法中使用`owned`，因为需要拥有一个值才能销毁它（或者在移动构造函数的情况下窃取这部分）。

### 比较`def`和`fn`参数传递[](#comparing-def-and-fn-argument-passing)

Mojo 的`def`函数本质上只是为`fn`函数加糖：

* 没有显式类型注释的参数`def`默认为`Object`.
* `def`没有约定关键字（例如`inout`或`owned`）的参数通过隐式复制传递到与参数同名的可变变量中。（这要求该类型有一个`__copyinit__`方法。）

例如，这两个函数具有相同的行为：

```
def example(inout a: Int, b: Int, c):
    # b and c use value semantics so they're mutable in the function
    ...

fn example(inout a: Int, b_in: Int, c_in: Object):
    # b_in and c_in are immutable references, so we make mutable shadow copies
    var b = b_in
    var c = c_in
    ...
```

卷影副本通常不会增加开销，因为像这样的小类型的引用`Object`复制起来很便宜。昂贵的部分是调整引用计数，但这可以通过移动优化来消除。

## Python集成[](#python-integration)

在 Mojo 中使用您熟悉和喜爱的 Python 模块非常简单。您可以将任何 Python 模块导入到 Mojo 程序中，并从 Mojo 类型创建 Python 类型。

### 导入Python模块[](#importing-python-modules)

要在 Mojo 中导入 Python 模块，只需`Python.import_module()`使用模块名称进行调用：

```
from python import Python

# This is equivalent to Python's `import numpy as np`
let np = Python.import_module("numpy")

# Now use numpy as if writing in Python
array = np.array([1, 2, 3])
print(array)
```

```
[1 2 3]
```

是的，这会导入 Python NumPy，并且您可以导入*任何其他 Python 模块*。

目前，您无法导入单个成员（例如单个 Python 类或函数），您必须导入整个 Python 模块，然后通过模块名称访问成员。

### Python 中的 Mojo 类型[](#mojo-types-in-python)

Mojo 原始类型隐式转换为 Python 对象。今天，我们支持列表、元组、整数、浮点数、布尔值和字符串。

例如，给定这个打印 Python 类型的 Python 函数：

```
%%python
def type_printer(my_list, my_tuple, my_int, my_string, my_float):
    print(type(my_list))
    print(type(my_tuple))
    print(type(my_int))
    print(type(my_string))
    print(type(my_float))
```

您可以毫无问题地传递 Python 函数 Mojo 类型：

```
type_printer([0, 3], (False, True), 4, "orange", 3.4)
```

```
<class 'list'>
<class 'tuple'>
<class 'int'>
<class 'str'>
<class 'float'>
```

请注意，在 Jupyter 笔记本中，上面声明的 Python 函数可自动供以下代码单元中的任何 Mojo 代码使用。

Mojo 还没有标准字典，因此还无法从 Mojo 字典创建 Python 字典。不过，您可以在 Mojo 中使用 Python 字典！要创建 Python 字典，请使用以下`dict`方法：

```
from python import Python
from python.object import PythonObject

var dictionary = Python.dict()
dictionary["fruit"] = "apple"
dictionary["starch"] = "potato"

var keys: PythonObject = ["fruit", "starch", "protein"]
var N: Int = keys.__len__().__index__()
print(N, "items")

for i in range(N):
    if Python.is_type(dictionary.get(keys[i]), Python.none()):
        print(keys[i], "is not in dictionary")
    else:
        print(keys[i], "is included")
```

```
3 items
fruit is included
starch is included
protein is not in dictionary
```

#### 导入本地Python模块[](#importing-local-python-modules)

如果您想在 Mojo 中使用一些本地 Python 代码，只需将目录添加到 Python 路径，然后导入模块即可。

例如，假设您有一个名为的 Python 文件`mypython.py`：

```
import numpy as np

def my_algorithm(a, b):
    array_a = np.random.rand(a, a)
    return array_a + b
```

以下是导入它并在 Mojo 文件中使用它的方法：

```
from python import Python

Python.add_to_path("path/to/module")
let mypython = Python.import_module("mypython")

let c = mypython.my_algorithm(2, 3)
print(c)
```

在 Mojo 中使用 Python 时无需担心内存管理问题。一切都正常，因为 Mojo 从一开始就是为 Python 设计的。

## 参数化：编译时元编程[](#parameterization-compile-time-metaprogramming)

Python 最令人惊奇的功能之一是其可扩展的运行时元编程功能。这使得广泛的库成为可能，并提供了灵活且可扩展的编程模型，世界各地的 Python 程序员都可以从中受益。不幸的是，这些功能也是有代价的：因为它们是在运行时评估的，所以它们直接影响底层代码的运行时效率。由于 IDE 不知道它们，因此代码完成等 IDE 功能很难理解它们并使用它们来改善开发人员体验。

在Python生态系统之外，静态元编程也是开发的重要组成部分，可以开发新的编程范例和高级库。这个领域有许多现有技术的例子，具有不同的权衡，例如：

1. 预处理器（例如C 预处理器、Lex/YACC 等）可能是最繁重的。它们完全通用，但在开发人员体验和工具集成方面最差。
2. 一些语言（如 Lisp 和 Rust）支持（有时是“卫生的”）宏扩展功能，通过更好的工具集成实现语法扩展和样板文件减少。
3. _一些较旧的语言（例如 C++）具有非常庞大且复杂的元编程语言（模板），它们是运行时_语言的双重语言。这些尤其难以学习，并且编译时间和错误消息都很差。
4. 有些语言（如 Swift）以一流的方式将许多功能构建到核心语言中，为常见情况提供良好的人体工程学效果，但牺牲了通用性。
5. 一些较新的语言（例如 Zig）将语言解释器集成到编译流程中，并允许解释器在编译 AST 时进行反映。这使得许多与宏系统相同的功能具有更好的扩展性和通用性。

对于 Modular 在人工智能、高性能机器学习内核和加速器方面的工作，我们需要先进的元编程系统提供的高抽象能力。我们需要高级的零成本抽象、富有表现力的库以及多种算法变体的大规模集成。我们希望库开发人员能够扩展系统，就像他们在 Python 中所做的那样，提供一个可扩展的开发平台。

也就是说，我们不愿意牺牲开发人员体验（包括编译时间和错误消息），也没有兴趣构建难以教授的并行语言生态系统。我们可以向这些以前的系统学习，但也可以在其基础上构建新技术，包括 MLIR 和细粒度语言集成缓存技术。

因此，Mojo 支持编译器中内置的编译时元编程，作为编译的一个单独阶段——在解析、语义分析和 IR 生成之后，但在降低到特定于目标的代码之前。它对运行时程序使用与元程序相同的宿主语言，并利用 MLIR 以可预测的方式表示和评估这些程序。

让我们看一些简单的例子。

**关于“参数”：** Python 开发人员可以互换地使用“参数”和“参数”这两个词来表示“传递到函数中的东西”。我们决定回收“参数”和“参数表达式”来表示 Mojo 中的编译时值，并继续使用“参数”和“表达式”来引用运行时值。这使我们能够围绕“参数化”和“参数化”等词进行对齐，以进行编译时元编程。

### 定义参数化类型和函数[](#defining-parameterized-types-and-functions)

[您可以通过在方括号中指定参数名称和类型来参数化结构和函数（使用PEP695 语法](https://peps.python.org/pep-0695/)的扩展版本）。[与参数值不同，参数值在编译时是已知的，这可以实现额外级别的抽象和代码重用，以及自动调整](https://docs.modular.com/mojo/programming-manual.html#autotuning-adaptive-compilation)等编译器优化。

例如，让我们看一下[SIMD](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data)类型，它表示硬件中保存标量数据类型的多个实例的低级向量寄存器。硬件加速器不断引入新的向量数据类型，甚至CPU也可能有512位或更长的SIMD向量。为了访问这些处理器上的 SIMD 指令，必须将数据整形为正确的 SIMD 宽度（数据类型）和长度（向量大小）。

然而，用 Mojo 的内置类型定义所有不同的 SIMD 变体是不可行的。因此，Mojo 的`SIMD`类型（定义为结构）在其方法中公开了常见的 SIMD 操作，并使 SIMD 数据类型和大小值参数化。这使您可以将数据直接映射到任何硬件上的 SIMD 向量。

这是 Mojo 类型定义的精简（非功能）版本`SIMD`：

```
struct SIMD[type: DType, size: Int]:
    var value: … # Some low-level MLIR stuff here

    # Create a new SIMD from a number of scalars
    fn __init__(inout self, *elems: SIMD[type, 1]):  ...

    # Fill a SIMD with a duplicated scalar value.
    @staticmethod
    fn splat(x: SIMD[type, 1]) -> SIMD[type, size]: ...

    # Cast the elements of the SIMD to a different elt type.
    fn cast[target: DType](self) -> SIMD[target, size]: ...

    # Many standard operators are supported.
    fn __add__(self, rhs: Self) -> Self: ...
```

使用参数定义每个 SIMD 变体非常有利于代码重用，因为该`SIMD`类型可以静态地表达所有不同的向量变体，而不需要语言预先定义每个变体。

因为`SIMD`是参数化类型，所以`self`其函数中的实参携带这些参数——完整的类型名称是`SIMD[type, size]`。尽管将其写出是有效的（如 的返回类型所示`splat()`），但这可能会很冗长，因此我们建议像示例一样使用该`Self`类型（来自[PEP673](https://peps.python.org/pep-0673/)）`__add__`。

### 参数重载[](#overloading-on-parameters)

函数和方法可以在其参数签名上重载。重载决策逻辑根据以下规则（按优先顺序）过滤候选者：

1. 具有最少数量隐式转换（在实参和参数中）的候选者。
2. 没有可变参数的候选人。
3. 没有可变参数的候选者。
4. 具有最短参数签名的候选者。

如果应用这些规则后存在多个候选者，则重载决策将失败。例如：

```
@register_passable("trivial")
struct MyInt:
    """A type that is implicitly convertible to `Int`."""
    var value: Int

    @always_inline("nodebug")
    fn __init__(_a: Int) -> Self:
        return Self {value: _a}

fn foo[x: MyInt, a: Int]():
  print("foo[x: MyInt, a: Int]()")

fn foo[x: MyInt, y: MyInt]():
  print("foo[x: MyInt, y: MyInt]()")

fn bar[a: Int](b: Int):
  print("bar[a: Int](b: Int)")

fn bar[a: Int](*b: Int):
  print("bar[a: Int](*b: Int)")

fn bar[*a: Int](b: Int):
  print("bar[*a: Int](b: Int)")

fn parameter_overloads[a: Int, b: Int, x: MyInt]():
    # `foo[x: MyInt, a: Int]()` is called because it requires no implicit
    # conversions, whereas `foo[x: MyInt, y: MyInt]()` requires one.
    foo[x, a]()

    # `bar[a: Int](b: Int)` is called because it does not have variadic
    # arguments or parameters.
    bar[a](b)

    # `bar[*a: Int](b: Int)` is called because it has variadic parameters.
    bar[a, a, a](b)

parameter_overloads[1, 2, MyInt(3)]()
```

```
foo[x: MyInt, a: Int]()
bar[a: Int](b: Int)
bar[*a: Int](b: Int)
```

### 使用参数化类型和函数[](#using-parameterized-types-and-functions)

您可以通过将值传递给方括号中的参数来实例化参数类型和函数。例如，对于`SIMD`上面的类型，`type`指定数据类型并`size`指定SIMD向量的长度（必须是2的幂）：

```
# Make a vector of 4 floats.
let small_vec = SIMD[DType.float32, 4](1.0, 2.0, 3.0, 4.0)

# Make a big vector containing 1.0 in float16 format.
let big_vec = SIMD[DType.float16, 32].splat(1.0)

# Do some math and convert the elements to float32.
let bigger_vec = (big_vec+big_vec).cast[DType.float32]()

# You can write types out explicitly if you want of course.
let bigger_vec2 : SIMD[DType.float32, 32] = bigger_vec

print('small_vec type:', small_vec.element_type, 'length:', len(small_vec))
print('bigger_vec2 type:', bigger_vec2.element_type, 'length:', len(bigger_vec2))
```

```
small_vec type: float32 length: 4
bigger_vec2 type: float32 length: 32
```

请注意，该`cast()`方法还需要一个参数来指定您想要的类型转换（上面的方法定义需要一个`target`参数值）。`SIMD`因此，就像结构是泛型类型定义一样，`cast()`方法也是泛型方法定义，它根据参数值在编译时而不是运行时实例化。

上面的代码显示了具体类型的使用（即，它`SIMD`使用已知类型值进行实例化），但参数的主要功能来自于定义参数算法和类型（使用参数值的代码）的能力。例如，以下是如何定义与`SIMD`类型和宽度无关的参数算法：

```
from math import sqrt

fn rsqrt[dt: DType, width: Int](x: SIMD[dt, width]) -> SIMD[dt, width]:
    return 1 / sqrt(x)

print(rsqrt[DType.float16, 4](42))
```

```
[0.154296875, 0.154296875, 0.154296875, 0.154296875]
```

请注意，`x`参数实际上是`SIMD`基于函数参数的类型。运行时程序可以使用参数的值，因为参数在运行时程序需要它们之前在编译时解析（但编译时参数表达式不能使用运行时值）。

Mojo 编译器对于参数的类型推断也很智能。请注意，上述函数无需[`sqrt()`](https://docs.modular.com/mojo/MojoStdlib/Math.html#sqrt)指定参数即可调用参数函数 - 编译器会推断其参数，就像您`sqrt[type, simd_width](x)`显式编写一样。另请注意，`rsqrt()`选择定义其第一个参数 name ，`width`即使`SIMD`类型将其命名为 it `size`，也没有问题。

### 参数表达式只是 Mojo 代码[](#parameter-expressions-are-just-mojo-code)

`a+b`参数表达式是出现在需要参数的位置的任何代码表达式（例如）。参数表达式支持运算符和函数调用，就像运行时代码一样，并且所有参数类型都使用与运行时程序相同的类型系统（例如`Int`和`DType`）。

由于参数表达式使用与运行时 Mojo 代码相同的语法和类型，因此您可以使用许多“依赖类型”功能。例如，您可能想要定义一个辅助函数来连接两个 SIMD 向量：

```
fn concat[ty: DType, len1: Int, len2: Int](
        lhs: SIMD[ty, len1], rhs: SIMD[ty, len2]) -> SIMD[ty, len1+len2]:

    var result = SIMD[ty, len1 + len2]()
    for i in range(len1):
        result[i] = SIMD[ty, 1](lhs[i])
    for j in range(len2):
        result[len1 + j] = SIMD[ty, 1](rhs[j])
    return result

let a = SIMD[DType.float32, 2](1, 2)
let x = concat[DType.float32, 2, 2](a, a)

print('result type:', x.element_type, 'length:', len(x))
```

```
result type: float32 length: 4
```

请注意，结果长度是输入向量长度的总和，您可以通过简单的`+`操作来表达它。对于更复杂的示例，请看一下[`SIMD.shuffle()`](https://docs.modular.com/mojo/MojoStdlib/SIMD.html#shuffle)标准库中的方法：它接受两个输入 SIMD 值、一个向量洗牌掩码作为列表，并返回一个与洗牌掩码长度匹配的 SIMD。

### 强大的编译时编程[](#powerful-compile-time-programming)

虽然简单的表达式很有用，但有时您希望使用控制流编写命令式编译时逻辑。例如，`isclose()`Mojo`Math`模块中的函数对整数使用精确相等，但对浮点使用“接近”比较。您甚至可以进行编译时递归。例如，下面是一个示例“树缩减”算法，它将向量的所有元素递归地求和为标量：

```
fn slice[ty: DType, new_size: Int, size: Int](
        x: SIMD[ty, size], offset: Int) -> SIMD[ty, new_size]:
    var result = SIMD[ty, new_size]()
    for i in range(new_size):
        result[i] = SIMD[ty, 1](x[i + offset])
    return result

fn reduce_add[ty: DType, size: Int](x: SIMD[ty, size]) -> Int:
    @parameter
    if size == 1:
        return x[0].to_int()
    elif size == 2:
        return x[0].to_int() + x[1].to_int()

    # Extract the top/bottom halves, add them, sum the elements.
    alias half_size = size // 2
    let lhs = slice[ty, half_size, size](x, 0)
    let rhs = slice[ty, half_size, size](x, half_size)
    return reduce_add[ty, half_size](lhs + rhs)
    
let x = SIMD[DType.index, 4](1, 2, 3, 4)
print(x)
print("Elements sum:", reduce_add[DType.index, 4](x))
```

```
[1, 2, 3, 4]
Elements sum: 10
```

这利用了该`@parameter if`功能，即`if`在编译时运行的语句。它要求其条件是有效的参数表达式，并确保只有语句的实时分支`if`被编译到程序中。

### Mojo 类型只是参数表达式[](#mojo-types-are-just-parameter-expressions)

虽然我们已经展示了如何在类型中使用参数表达式，但类型注释本身可以是任意表达式（就像在 Python 中一样）。Mojo 中的类型具有特殊的元类型，允许定义类型参数算法和函数。

例如，我们可以创建一个`Array`支持任意类型元素的简化版本（通过`AnyType`参数）：

```
struct Array[T: AnyType]:
    var data: Pointer[T]
    var size: Int

    fn __init__(inout self, size: Int, value: T):
        self.size = size
        self.data = Pointer[T].alloc(self.size)
        for i in range(self.size):
            self.data.store(i, value)
              
    fn __getitem__(self, i: Int) -> T:
        return self.data.load(i)

    fn __del__(owned self):
        self.data.free()

var v = Array[Float32](4, 3.14)
print(v[0], v[1], v[2], v[3])
```

```
3.1400001049041748 3.1400001049041748 3.1400001049041748 3.1400001049041748
```

请注意，`T`参数被用作参数的形式类型`value`和函数的返回类型`__getitem__`。参数允许`Array`类型根据不同的用例提供不同的 API。

还有许多其他情况可以从参数的更高级使用中受益。例如，您可以并行执行闭包 N 次，并从上下文中输入一个值，如下所示：

```
fn parallelize[func: fn (Int) -> None](num_work_items: Int):
    # Not actually parallel: see the 'algorithm' module for real implementation.
    for i in range(num_work_items):
        func(i)
```

另一个重要的例子是可变参数泛型，其中可能需要在异构类型列表上定义算法或数据结构，例如元组：

```
struct Tuple[*Ts: AnyType]:
    var _storage : *Ts
```

尽管我们还没有足够的元类型助手，但我们将来应该能够编写类似的东西（尽管重载仍然是处理此问题的更好方法）：

```
struct Array[T: AnyType]:
    fn __getitem__[IndexType: AnyType](self, idx: IndexType)
       -> (ArraySlice[T] if issubclass(IndexType, Range) else T):
       ...
```

### `alias`: 命名参数表达式[](#alias-named-parameter-expressions)

想要_命名_编译时值是很常见的。虽然`var`定义了运行时值并`let`定义了运行时常量，但我们需要一种方法来定义编译时临时值。为此，Mojo 使用`alias`声明。

例如，该`DType`结构使用枚举器的别名实现一个简单的枚举，如下所示（实际`DType`实现细节略有不同）：

```
struct DType:
    var value : UI8
    alias invalid = DType(0)
    alias bool = DType(1)
    alias int8 = DType(2)
    alias uint8 = DType(3)
    alias int16 = DType(4)
    alias int16 = DType(5)
    ...
    alias float32 = DType(15)
```

这允许客户端自然地用作`DType.float32`参数表达式（也可以用作运行时值）。`DType`请注意，这是在编译时调用运行时构造函数。

类型是别名的另一个常见用途。因为类型是编译时表达式，所以能够很方便地执行以下操作：

```
alias Float16 = SIMD[DType.float16, 1]
alias UInt8 = SIMD[DType.uint8, 1]

var x : Float16   # FLoat16 works like a "typedef"
```

与`var`和一样`let`，别名遵循范围，并且您可以按照预期在函数中使用本地别名。

顺便说一下， 和`None`都`AnyType`被定义为[类型别名](https://docs.modular.com/mojo/MojoBuiltin/TypeAliases.html)。

### 自动调整/自适​​应编译[](#autotuning-adaptive-compilation)

Mojo 参数表达式允许您像在其他语言中一样编写可移植的参数算法，但是在编写高性能代码时，您仍然必须选择用于参数的具体值。例如，在编写高性能数值算法时，您可能希望使用内存平铺来加速算法，但要使用的维度在很大程度上取决于可用的硬件功能、缓存的大小、融合到内核中的内容以及许多其他繁琐的细节。

即使向量长度也可能难以管理，因为典型机器的向量长度取决于数据类型，并且某些数据类型`bfloat16`并不完全支持所有实现。Mojo 通过`autotune`在标准库中提供一个函数来提供帮助。例如，如果您想将与向量长度无关的算法写入数据缓冲区，您可以这样编写：

```
from autotune import autotune, search
from benchmark import Benchmark
from memory.unsafe import DTypePointer
from algorithm import vectorize

fn buffer_elementwise_add_impl[
    dt: DType
](lhs: DTypePointer[dt], rhs: DTypePointer[dt], result: DTypePointer[dt], N: Int):
    """Perform elementwise addition of N elements in RHS and LHS and store
    the result in RESULT.
    """
    @parameter
    fn add_simd[size: Int](idx: Int):
        let lhs_simd = lhs.simd_load[size](idx)
        let rhs_simd = rhs.simd_load[size](idx)
        result.simd_store[size](idx, lhs_simd + rhs_simd)
    
    # Pick vector length for this dtype and hardware
    alias vector_len = autotune(1, 4, 8, 16, 32)

    # Use it as the vectorization length
    vectorize[vector_len, add_simd](N)

fn elementwise_evaluator[dt: DType](
    fns: Pointer[fn (DTypePointer[dt], DTypePointer[dt], DTypePointer[dt], Int) -> None],
    num: Int,
) -> Int:
    # Benchmark the implementations on N = 64.
    alias N = 64
    let lhs = DTypePointer[dt].alloc(N)
    let rhs = DTypePointer[dt].alloc(N)
    let result = DTypePointer[dt].alloc(N)

    # Fill with ones.
    for i in range(N):
        lhs.store(i, 1)
        rhs.store(i, 1)

    # Find the fastest implementation.
    var best_idx: Int = -1
    var best_time: Int = -1
    for i in range(num):
        @parameter
        fn wrapper():
            fns.load(i)(lhs, rhs, result, N)
        let cur_time = Benchmark(1).run[wrapper]()
        if best_idx < 0 or best_time > cur_time:
            best_idx = i
            best_time = cur_time
        print("time[", i, "] =", cur_time)
    print("selected:", best_idx)
    return best_idx

fn buffer_elementwise_add[
    dt: DType
](lhs: DTypePointer[dt], rhs: DTypePointer[dt], result: DTypePointer[dt], N: Int):
    # Forward declare the result parameter.
    alias best_impl: fn(DTypePointer[dt], DTypePointer[dt], DTypePointer[dt], Int) -> None

    # Perform search!
    search[
      fn(DTypePointer[dt], DTypePointer[dt], DTypePointer[dt], Int) -> None,
      buffer_elementwise_add_impl[dt],
      elementwise_evaluator[dt] -> best_impl
    ]()

    # Call the select implementation
    best_impl(lhs, rhs, result, N)
```

我们现在可以像往常一样调用我们的函数：

```
let N = 32
let a = DTypePointer[DType.float32].alloc(N)
let b = DTypePointer[DType.float32].alloc(N)
let res = DTypePointer[DType.float32].alloc(N)
# Initialize arrays with some values
for i in range(N):
    a.store(i, 2.0)
    b.store(i, 40.0)
    res.store(i, -1)
    
buffer_elementwise_add[DType.float32](a, b, res, N)
print(a.load(10), b.load(10), res.load(10))
```

```
time[ 0 ] = 23
time[ 1 ] = 6
time[ 2 ] = 4
time[ 3 ] = 3
time[ 4 ] = 4
selected: 3
2.0 40.0 42.0
```

编译此代码的实例时，Mojo 会分叉此算法的编译，并通过测量在实践中最适合目标硬件的值来决定使用哪个值。它评估表达式的不同值`vector_len`，并根据用户定义的性能评估器选择最快的值。例如，因为它单独测量和评估每个选项，所以它可能会为`float32`选取不同的向量长度。`int8`这个简单的功能非常强大——超越了简单的整数常量——因为函数和类型也是参数表达式。

请注意，最佳向量长度的搜索是由该[`search()`](https://docs.modular.com/mojo/stdlib/autotune/autotuning.html#search)函数执行的。`search()`接受一个评估器和一个分叉函数，并返回评估器选择的最快实现作为参数结果。[要更深入地了解此主题，请查看有关Mojo 中的](https://docs.modular.com/mojo/notebooks/Memset.html)[矩阵乘法](https://docs.modular.com/mojo/notebooks/Matmul.html)和Fast Memset的笔记本。[](https://docs.modular.com/mojo/notebooks/Memset.html)

自动调优本质上是一种指数技术，受益于 Mojo 编译器堆栈的内部实现细节（特别是 MLIR、集成缓存和编译分发）。这也是一个高级用户功能，需要随着时间的推移不断开发和迭代。

## “价值生命周期”：价值的诞生、存在和消亡[](#value-lifecycle-birth-life-and-death-of-a-value)

此时，您应该了解 Mojo 函数和类型的核心语义和功能，因此我们现在可以讨论它们如何组合在一起以在 Mojo 中表达新类型。

许多现有语言都通过不同的权衡来表达设计点：例如，C++ 非常强大，但经常被指责“默认设置错误”，从而导致错误和错误功能。Swift 易于使用，但其模型的可预测性较差，会复制大量值，并且依赖于“ARC 优化器”来提高性能。Rust 从强大的价值所有权目标开始，以满足其借用检查器的要求，但依赖于可移动的值，这使得表达自定义移动构造函数变得具有挑战性，并且会给性能带来很大的压力`memcpy`。在Python中，一切都是对类的引用，因此它永远不会真正面临类型问题。

对于 Mojo，我们从这些现有系统中学习，我们的目标是提供一个非常强大且易于学习和理解的模型。我们也不想要求“尽最大努力”和难以预测的优化过程内置到“足够智能”的编译器中。

为了探索这些问题，我们研究了不同的价值分类和表达它们的相关 Mojo 功能，并自下而上构建。我们在示例中使用 C++ 作为主要比较点，因为它众所周知，但如果其他语言提供了更好的比较点，我们偶尔会参考它们。

### 无法实例化的类型[](#types-that-cannot-be-instantiated)

Mojo 中最简单的类型是不允许创建它的实例的类型：这些类型根本没有初始值设定项，如果它们有析构函数，则永远不会调用它（因为无法销毁实例） ）：

```
struct NoInstances:
    var state: Int  # Pretty useless

    alias my_int = Int

    @staticmethod
    fn print_hello():
        print("hello world")
```

Mojo 类型默认情况下不会获得默认构造函数、移动构造函数、成员初始化器或其他任何内容，因此不可能创建此`NoInstances`类型的实例。为了获得它们，您需要定义一个`__init__`方法或使用合成初始化程序的装饰器。如图所示，这些类型可用作“命名空间”，因为您可以引用静态成员，例如`NoInstances.my_int`或`NoInstances.print_hello()`即使您无法实例化该类型的实例。

### 不可移动和不可复制类型[](#non-movable-and-non-copyable-types)

如果我们在复杂性的阶梯上更进一步，我们将得到可以实例化的类型，但是一旦它们被固定到内存中的地址，它们就不能被隐式移动或复制。这对于实现原子操作（例如`std::atomic`在 C++ 中）等类型或其他类型非常有用，其中值的内存地址是其标识并且对其用途至关重要：

```
struct Atomic:
    var state: Int

    fn __init__(inout self, state: Int = 0):
        self.state = state

    fn __iadd__(inout self, rhs: Int):
        #...atomic magic...

    fn get_value(self) -> Int:
        return atomic_load_int(self.state)
```

此类定义了一个初始化程序，但没有复制或移动构造函数，因此一旦初始化，就永远无法移动或复制。这是安全且有用的，因为 Mojo 的所有权系统是完全“地址正确的”——当它被初始化到堆栈或其他类型的字段中时，它永远不需要移动。

请注意，Mojo 的方法仅控制内置移动操作，例如`a = b`复制和[`^`传输运算符](https://docs.modular.com/mojo/programming-manual.html#owned-arguments)。您可以用于自己的类型（如上`Atomic`）的一种有用模式是添加显式`copy()`方法（非“dunder”方法）。当程序员知道实例是安全的时，这对于制作实例的显式副本很有用。

### 独特的“仅移动”类型[](#unique-move-only-types)

如果我们在能力的阶梯上再上一层楼，我们将遇到“唯一”的类型 - C++ 中有很多这样的例子，例如类似的类型，甚至是拥有底层 POSIX 文件描述符的`std::unique_ptr`类型`FileDescriptor`。这些类型在 Rust 等语言中很普遍，不鼓励复制，但“移动”是免费的。`__moveinit__`在 Mojo 中，您可以通过定义获取唯一类型所有权的方法来实现这些类型的移动。例如：

```
# This is a simple wrapper around POSIX-style fcntl.h functions.
struct FileDescriptor:
    var fd: Int

    # This is how we move our unique type.
    fn __moveinit__(inout self, owned existing: Self):
        self.fd = existing.fd

    # This takes ownership of a POSIX file descriptor.
    fn __init__(inout self, fd: Int):
        self.fd = fd

    fn __init__(inout self, path: String):
        # Error handling omitted, call the open(2) syscall.
        self = FileDescriptor(open(path, ...))

    fn __del__(owned self):
        close(self.fd)   # pseudo code, call close(2)

    fn dup(self) -> Self:
        # Invoke the dup(2) system call.
        return Self(dup(self.fd))
    fn read(...): ...
    fn write(...): ...
```

消费移动构造函数 ( `__moveinit__`) 获取现有 的所有权`FileDescriptor`，并将其内部实现细节移动到新实例。这是因为 的实例`FileDescriptor`可能存在于不同的位置，并且它们可以在逻辑上移动——窃取一个值的主体并将其移动到另一个值中。

这是一个会`__moveinit__`多次调用的令人震惊的示例：

```
fn egregious_moves(owned fd1: FileDescriptor):
    # fd1 and fd2 have different addresses in memory, but the
    # transfer operator moves unique ownership from fd1 to fd2.
    let fd2 = fd1^

    # Do it again, a use of fd2 after this point will produce an error.
    let fd3 = fd2^

    # We can do this all day...
    let fd4 = fd3^
    fd4.read(...)
    # fd4.__del__() runs here
```

请注意如何使用后缀“转移”运算符在拥有该值的各个值之间转移该值的所有权`^`，这会破坏先前的绑定并将所有权转移到新的常量。如果您熟悉 C++，那么考虑转移运算符的简单方法就像`std::move`，但在这种情况下，我们可以看到它能够移动事物而不将它们重置为可以销毁的状态：在 C++ 中，如果您移动运算符无法更改旧值的`fd`实例，它将被关闭两次。

Mojo 跟踪值的活跃度并允许您定义自定义移动构造函数。这很少需要，但一旦需要就非常强大。例如，某些类型喜欢[`llvm::SmallVector type`](https://llvm.org/docs/ProgrammersManual.html#llvm-adt-smallvector-h)使用“内联存储”优化技术，并且它们可能希望通过“内部指针”来实现到其实例中。这是一个众所周知的技巧，可以减轻 malloc 内存分配器的压力，但这意味着“移动”操作需要自定义逻辑来在发生这种情况时更新指针。

使用 Mojo，这就像实现自定义`__moveinit__`方法一样简单。这在 C++ 中也很容易实现（不过，在不需要自定义逻辑的情况下可以使用样板），但在其他流行的内存安全语言中很难实现。

另一点需要注意的是，虽然 Mojo 编译器提供了良好的可预测性和控制，但它也非常复杂。它保留消除临时和相应的复制/移动操作的权利。如果这不适合您的类型，您应该使用显式方法（例如）`copy()`而不是 dunder 方法。

### 支持“采取行动”的类型[](#types-that-support-a-taking-move)

内存安全语言面临的一个挑战是，它们需要围绕编译器能够跟踪的内容提供可预测的编程模型，而编译器中的静态分析本质上是有限的。例如，虽然编译器可以理解下面第一个示例中的两个数组访问是针对不同的数组元素，但（通常）不可能推理第二个示例（这是 C++ 代码）：

```
std::pair<T, T> getValues1(MutableArray<T> &array) {
    return { std::move(array[0]), std::move(array[1]) };
}
std::pair<T, T> getValues2(MutableArray<T> &array, size_t i, size_t j) {
    return { std::move(array[i]), std::move(array[j]) };
}
```

这里的问题是根本没有办法（仅查看上面的函数体）知道或证明 和 的动态值`i`不`j`相同。虽然可以维护动态状态来跟踪数组的各个元素是否处于活动状态，但这通常会导致大量的运行时开销（即使不使用移动/传输），这是 Mojo 和其他系统编程语言不热衷的事情做。解决这个问题的方法有很多种，包括一些相当复杂且并不总是容易学习的解决方案。

Mojo 采用务实的方法，让 Mojo 程序员无需处理其类型系统即可完成工作。如上所示，它并不强制类型可复制、可移动，甚至可构造，但它确实希望类型能够表达其完整契约，并且它希望实现程序员期望从 C++ 等语言中获得的流畅设计模式。这里的（众所周知的）观察是，许多对象具有可以“拿走”的内容，而无需禁用其析构函数，或者因为它们具有“空状态”（如可选类型或可为空指针），或者因为它们具有空值可以高效地创建并且无操作可以销毁（例如， `std::vector`其数据可以有一个空指针）。

为了支持这些用例，[`^`转移运算符](https://docs.modular.com/mojo/programming-manual.html#owned-arguments)支持任意左值，并且当应用于其中一个时，它会调用“采取移动构造函数”，拼写为`__takeinit__`。此构造函数必须将新值设置为活动状态，并且它可以改变旧值，但它必须将旧值置于其析构函数仍然可以工作的状态。例如，如果我们想将我们`FileDescriptor`放入一个向量中并移出它，我们可能会选择扩展它以知道它`-1`是一个哨兵，这意味着它是“空”。我们可以这样实现：

```
# This is a simple wrapper around POSIX-style fcntl.h functions.
struct FileDescriptor:
    var fd: Int

    # This is the new key capability.
    fn __takeinit__(inout self, inout existing: Self):
        self.fd = existing.fd
        existing.fd = -1  # neutralize 'existing'.

    fn __moveinit__(inout self, owned existing: Self): # as above
    fn __init__(inout self, fd: Int): # as above
    fn __init__(inout self, path: String): # as above

    fn __del__(owned self):
        if self.fd != -1:
            close(self.fd)   # pseudo code, call close(2)
```

请注意“窃取移动”构造函数如何从现有值中获取文件描述符并改变该值，以便其析构函数不会执行任何操作。这种技术需要权衡，并不是对每种类型都是最好的。我们可以看到它向析构函数添加了一个（廉价的）分支，因为它必须检查哨兵情况。通常也认为使此类类型可为空是不好的形式，因为像类型这样的更通用的功能`Optional[T]`是处理这种情况的更好方法。

此外，我们计划在 Mojo 本身中实现`Optional[T]`，并且`Optional`需要此功能。我们还相信，库作者比语言设计者更了解他们的领域问题，并且通常更愿意赋予库作者对该领域的全部权力。因此，您可以选择（但不必）让您的类型以选择加入的方式参与此行为。

### 可复制类型[](#copyable-types)

可移动类型的下一步是可复制类型。可复制类型也很常见 - 程序员通常期望字符串和数组之类的东西是可复制的，并且每个 Python 对象引用都是可复制的 - 通过复制指针和调整引用计数。

有很多方法可以实现可复制类型。一种可以实现像 Python 或 Java 这样的引用语义类型，在其中传播共享指针，一种可以使用易于共享的不可变数据结构，因为它们一旦创建就不会发生变化，一种可以通过惰性写入时复制来实现深层值语义就像斯威夫特那样。这些方法中的每一种都有不同的权衡，Mojo 认为，虽然我们需要一些通用的集合类型集，但我们也可以支持广泛的专注于特定用例的专用集合类型。

在 Mojo 中，您可以通过实现该`__copyinit__`方法来做到这一点。这是使用简单的示例`String`（伪代码）：

```
struct MyString:
    var data: Pointer[UI8]

    # StringRef is a pointer + length and works with StringLiteral.
    def __init__(inout self, input: StringRef):
        self.data = ...

    # Copy the string by deep copying the underlying malloc'd data.
    def __copyinit__(inout self, existing: Self):
        self.data = strdup(existing.data)

    # This isn't required, but optimizes unneeded copies.
    def __moveinit__(inout self, owned existing: Self):
        self.data = existing.data

    def __del__(owned self):
        free(self.data.address)

    def __add__(self, rhs: MyString) -> MyString: ...
```

这个简单类型是一个指向用 malloc 分配的“空终止”字符串数据的指针，为清楚起见，使用老式 C API。它实现了`__copyinit__`，它维护了每个 实例都拥有其底层指针并在销毁时释放它的不变式`MyString`。这个实现建立在我们上面看到的技巧的基础上，并实现了一个`__moveinit__`构造函数，这使得它可以在某些常见情况下完全消除临时副本。您可以在此代码序列中看到此行为：

```
fn test_my_string():
    var s1 = MyString("hello ")

    var s2 = s1    # s2.__copyinit__(s1) runs here

    print(s1)

    var s3 = s1^   # s3.__moveinit__(s1) runs here

    print(s2)
    # s2.__del__() runs here
    print(s3)
    # s3.__del__() runs here
```

在这种情况下，您可以明白为什么需要复制构造函数：如果没有复制构造函数，将值复制`s1`到`s2`将是一个错误 - 因为您不能拥有同一不可复制类型的两个实时实例。移动构造函数是可选的，但有助于分配到`s3`：没有它，编译器将从 s1 调用复制构造函数，然后销毁旧`s1`实例。这在逻辑上是正确的，但会带来额外的运行时开销。

Mojo 会急切地销毁值，这使得它能够将复制+销毁对转换为单个移动操作，这可以带来比 C++ 更好的性能，而不需要对`std::move`.

### 琐碎类型[](#trivial-types)

最灵活的类型就是“比特袋”。这些类型是“微不足道的”，因为它们可以被复制、移动和销毁，而无需调用自定义代码。像这样的类型可以说是我们周围最常见的基本类型：像整数和浮点值这样的东西都是微不足道的。从语言的角度来看，Mojo 不需要对这些进行特殊支持，类型作者将这些东西实现为无操作并允许内联器让它们消失是完全可以的。

这种方法不是最理想的有两个原因：一是我们不希望在简单类型上定义一堆方法的样板，二是我们不希望生成和推送的编译时开销一堆函数调用，只是让它们内联到什么都没有。此外，还存在一个正交问题，即许多类型在另一种意义上是微不足道的：它们很小，应该在 CPU 的寄存器中传递，而不是间接在内存中传递。

因此，Mojo 提供了一个结构装饰器来解决所有这些问题。您可以使用装饰器实现类型`@register_passable("trivial")`，这告诉 Mojo 该类型应该是可复制和可移动的，但它没有用户定义的逻辑来执行此操作。它还告诉 Mojo 更喜欢传递 CPU 寄存器中的值，这可以带来效率优势。

TODO：这个装饰器需要重新考虑。缺乏自定义逻辑复制/移动/销毁逻辑和“寄存器中的可传递性”是正交的问题，应该分开。前面的逻辑应该包含在一个更通用的`@value("trivial")`装饰器中，它与 正交`@register_passable`。

### `@value`装饰者[](#value-decorator)

Mojo 的[价值生命周期](https://docs.modular.com/mojo/programming-manual.html#value-lifecycle-birth-life-and-death-of-a-value)提供了简单且可预测的钩子，使您能够正确表达异国情调的低级事物`Atomic`。这对于控制和简单的编程模型来说非常有用，但大多数结构都是其他类型的简单聚合，我们不想为它们编写大量样板文件。为了解决这个问题，Mojo 提供了一个`@value`结构装饰器，可以为您合成大量样板文件。

您可以将`@value`其视为 Python 的扩展[`@dataclass`](https://docs.python.org/3/library/dataclasses.html)，它还可以处理 Mojo`__moveinit__`和`__copyinit__`方法。

装饰`@value`器会查看您类型的字段，并生成一些缺少的成员。例如，考虑这样一个简单的结构：

```
@value
struct MyPet:
    var name: String
    var age: Int
```

Mojo 会注意到您没有成员初始化器、移动构造函数或复制构造函数，它会为您合成这些，就像您编写的一样：

```
struct MyPet:
    var name: String
    var age: Int

    fn __init__(inout self, owned name: String, age: Int):
        self.name = name^
        self.age = age

    fn __copyinit__(inout self, existing: Self):
        self.name = existing.name
        self.age = existing.age

    fn __moveinit__(inout self, owned existing: Self):
        self.name = existing.name^
        self.age = existing.age
```

当您添加`@value`装饰器时，Mojo 仅当这些特殊方法不存在时才会合成它们。您可以通过定义自己的版本来覆盖一个或多个版本的行为。例如，想要自定义复制构造函数但使用默认的成员方式和移动构造函数是相当常见的。

由于结构取得所有权并存储值，因此 的参数`__init__`全部作为参数传递。`owned`这是一个有用的微观优化，可以使用仅移动类型。像这样的简单类型`Int`也可以作为自有值传递，但由于这对它们来说没有任何意义，因此`^`为了清楚起见，我们省略了标记和传输运算符 ( )。

**注意：**如果您的类型包含任何[仅移动](https://docs.modular.com/mojo/programming-manual.html#unique-move-only-types)字段，Mojo 将不会生成复制构造函数，因为它无法复制这些字段。此外，`@value`装饰器仅适用于其成员可复制和/或可移动的类型。如果您`Atomic`的结构中有类似的内容，那么它可能不是值类型，并且您无论如何都不想要这些成员。

另请注意，`MyPet`上面的结构不包括`__del__()`析构函数 - Mojo 也综合了它，但它不需要装饰器（请参阅下面有关[析构函数](https://docs.modular.com/mojo/programming-manual.html#behavior-of-destructors)`@value`的部分）。[](https://docs.modular.com/mojo/programming-manual.html#behavior-of-destructors)

此时没有办法抑制特定方法的生成或自定义生成，但`@value`如果有需求，我们可以向生成器添加参数来执行此操作。

## 析构函数的行为[](#behavior-of-destructors)

Mojo 中的任何结构都可以有一个析构函数（一种`__del__()`方法），该析构函数会在值的生命周期结束时（通常是最后使用该值的时间点）自动运行。例如，一个简单的字符串可能如下所示（伪代码）：

```
@value
struct MyString:
    var data: Pointer[UInt8]

    def __init__(inout self, input: StringRef): ...
    def __add__(self, rhs: String) -> MyString: ...
    def __del__(owned self):
        free(self.data.address)
```

Mojo使用每次调用后运行的**“尽快”**`MyString` (ASAP) 策略来销毁诸如（它调用析构函数）之类的值。Mojo 不会_等到_代码块结束时才销毁未使用的值。即使在像这样的表达式中，一旦不再需要中间表达式，Mojo 就会立即销毁它们，而不会等到语句结束。`__del__()`****__`a+b+c+d`

当值失效时，Mojo 编译器会自动调用析构函数，并为析构函数何时运行提供强有力的保证。Mojo 使用静态编译器分析来推理代码并决定何时插入对析构函数的调用。例如：

```
fn use_strings():
    var a = String("hello a")
    let b = String("hello b")
    print(a)
    # a.__del__() runs here for "hello a"


    print(b)
    # b.__del__() runs here

    a = String("temporary a")
    # a.__del__() runs here because "temporary a" is never used

    # Other stuff happens here

    a = String("final a")
    print(a)
    # a.__del__() runs again here for "final a"

use_strings()
```

```
hello a
hello b
final a
```

在上面的代码中，您将看到 和`a`值`b`是在早期创建的，并且值的每个初始化都与对析构函数的调用相匹配。请注意，它`a`会被多次销毁——每次收到新值时都会被销毁一次。

现在，这可能会让 C++ 程序员感到惊讶，因为它与[RAII 模式不同，在 RAII 模式](https://en.cppreference.com/w/cpp/language/raii)中，C++ 在作用域末尾销毁值。Mojo 还遵循这样的原则：值在构造函数中获取资源并在析构函数中释放资源，但 Mojo 中的急切销毁比 C++ 中基于范围的销毁具有许多强大的优势：

* Mojo 方法消除了类型实现重新赋值运算符的需要，例如C++ 中的`operator=(const T&)`and `operator=(T&&)`，从而更容易定义类型并消除概念。
* Mojo 不允许可变引用与其他可变引用或不可变借用重叠。它提供可预测编程模型的一个主要方法是确保对对象的引用尽快消失，避免编译器认为一个值可能仍然存在并干扰另一个值的混乱情况，但这对于用户。
* 在最后使用时销毁值与“移动”优化很好地结合在一起，它将“复制+删除”对转换为“移动”操作，这是 C++ 移动优化的推广，如 NRVO（称为返回值优化）。
* 对于某些常见模式（例如尾递归），在 C++ 中销毁作用域末尾的值是有问题的，因为析构函数调用发生在尾调用之后。对于某些函数式编程模式来说，这可能是一个重大的性能和内存问题。

重要的是，Mojo 的热切销毁在 Python 风格的`def`函数中也能很好地工作，以在细粒度级别提供销毁保证（无需垃圾收集器）——回想一下，Python 并不真正提供超出函数的作用域，因此 Mojo 中的 C++ 风格销毁会少很多用处。

**注意：** Mojo 还支持 Python 风格的[`with`语句](https://docs.python.org/3/reference/compound_stmts.html#the-with-statement)，它提供了对资源更有意范围的访问。

Mojo 方法与 Rust 和 Swift 的工作方式更相似，因为它们都具有强大的价值所有权跟踪并提供内存安全。一个区别是它们的实现需要使用[动态“放置标志”](https://doc.rust-lang.org/nomicon/drop-flags.html) ——它们维护隐藏的影子变量来跟踪值的状态以提供安全性。这些通常会被优化掉，但 Mojo 方法完全消除了这种开销，使生成的代码更快并避免歧义。

### 现场敏感的生命周期管理[](#field-sensitive-lifetime-management)

Mojo 的生命周期分析除了完全控制流感知之外，它还完全字段敏感（结构的每个字段都是独立跟踪的）。也就是说，Mojo 单独跟踪“整个对象”是完全还是仅部分初始化/销毁。

例如，考虑以下代码：

```
@value
struct TwoStrings:
    var str1: String
    var str2: String

fn use_two_strings():
    var ts = TwoStrings("foo", "bar")
    print(ts.str1)
    # ts.str1.__del__() runs here

    # Other stuff happens here

    ts.str1 = String("hello") # Overwrite ts.str1
    print(ts.str1)
    # ts.__del__() runs here

use_two_strings()
```

```
foo
hello
```

请注意，该`ts.str1`字段几乎立即被销毁，因为 Mojo 知道它将被下面覆盖。[您还可以在使用转移运算符](https://docs.modular.com/mojo/programming-manual.html#owned-arguments)时看到这一点，例如：

```
fn consume(owned arg: String):
    pass

fn use(arg: TwoStrings):
    print(arg.str1)

fn consume_and_use_two_strings():
    var ts = TwoStrings("foo", "bar")
    consume(ts.str1^)
    # ts.str1.__moveinit__() runs here

    # ts is now only partially initialized here!

    ts.str1 = String("hello")  # All together now
    use(ts)                    # This is ok
    # ts.__del__() runs here

consume_and_use_two_strings()
```

```
hello
```

请注意，代码转移了该`str1`字段的所有权：在 的持续时间内`other_stuff()`，该`str1`字段完全未初始化，因为所有权已转移到`consume()`。然后`str1`在被`use()`函数使用之前重新初始化（如果不是，Mojo 会拒绝代码并显示未初始化字段错误）。

Mojo 对此的规则非常强大且有意直接：字段可以临时转移，但“整个对象”必须使用聚合类型的初始值设定项构造并使用聚合析构函数销毁。这意味着不可能通过仅初始化其字段来创建对象，也不可能通过仅销毁其字段来拆除对象。例如，以下代码无法编译：

```
fn consume_and_use_two_strings():
    let ts = TwoStrings("foo", "bar") # ts is initialized
    # Uncomment to see an error:
    # consume(ts.str1^)
    # Because `ts` is not used anymore, it should be destroyed here, but
    # the object is not whole, preventing the overall value from being destroyed

    let ts2 : TwoStrings # ts2 type is declared but not initialized
    ts2.str1 = String("foo")
    ts2.str2 = String("bar")  # Both the member are initalized
    # Uncomment to see an error:
    # use(ts2) # Error: 'ts2' isn't fully initialized
```

虽然我们可以允许这样的模式发生，但我们拒绝这种情况，因为值不仅仅是其各部分的总和。考虑包含 POSIX 文件描述符作为整数值的 a `FileDescriptor`：销毁整数（无操作）和销毁`FileDescriptor`（可能调用`close()`系统调用）之间存在很大差异。因此，我们要求所有全值初始化都经过初始值设定项并用其全值析构函数销毁。

就其价值而言，Mojo 内部确实具有与 Rust 函数等效的[`mem::forget`](https://doc.rust-lang.org/std/mem/fn.forget.html)功能，它显式禁用析构函数，并具有用于“祝福”对象的相应内部功能，但此时它们尚未公开供用户使用。

### 现场寿命`__init__`[](#field-lifetimes-in-__init__)

方法的行为`__init__`几乎与任何其他方法一样 - 有一点神奇：它知道对象的字段未初始化，但它相信整个对象已初始化。这意味着`self`一旦所有字段都初始化，您就可以将其用作整个对象：

```
fn use(arg: TwoStrings2):
    pass

struct TwoStrings2:
    var str1: String
    var str2: String

    fn __init__(inout self, cond: Bool, other: String):
        self.str1 = String()
        if cond:
            self.str2 = other
            use(self)  # Safe to use immediately!
            # self.str2.__del__(): destroyed because overwritten below.

        self.str2 = self.str1
        use(self)  # Safe to use immediately!
```

类似地，Mojo 中的初始化器完全覆盖 是安全的`self`，例如通过委托给其他初始化器：

```
struct TwoStrings3:
    var str1: String
    var str2: String

    fn __init__(inout self):
        self.str1 = String()
        self.str2 = String()

    fn __init__(inout self, one: String):
        self = TwoStrings3()  # Delegate to the basic init
        self.str1 = one
```

### 和`owned`中参数的字段生命周期`__moveinit__``__del__`[](#field-lifetimes-of-owned-arguments-in-__moveinit__-and-__del__)

移动初始值设定项和析构`owned`函数的参数存在最后一点魔力。回顾一下，这些方法签名如下所示：`__moveinit__()``__del__()`

```
struct TwoStrings:
    ...
    fn __moveinit__(inout self, owned existing: Self):
        # Initializes a new `self` by consuming the contents of `existing`
    fn __del__(owned self):
        # Destroys all resources in `self`
```

这些方法面临一个有趣但晦涩难懂的问题：这两种方法都负责拆解`owned` `existing`/`self`值。也就是说，`__moveinit__()`销毁 的子元素`existing`以便将所有权转移到新实例，同时`__del__()`实现其 的删除逻辑`self`。因此，他们都希望拥有并转换该`owned`值的元素，并且他们绝对不希望该`owned`值的析构函数也运行（在该方法的情况下`__del__()`，这将变成无限循环）。

为了解决这个问题，Mojo 专门处理这两个方法，假设它们的整个值在从方法返回时被销毁。这意味着在传输字段值之前可以使用整个对象。例如，这将按您的预期工作：

```
fn consume(owned str: String):
    print('Consumed', str)

struct TwoStrings4:
    var str1: String
    var str2: String

    fn __init__(inout self, one: String):
        self.str1 = one
        self.str2 = String("bar")

    fn __moveinit__(inout self, owned existing: Self):
        self.str1 = existing.str1
        self.str2 = existing.str2

    fn __del__(owned self):
        self.dump() # Self is still whole here
        # Mojo calls self.str2.__del__() since str2 isn't used anymore

        consume(self.str1^)
        # str1 has now been transferred;
        # `self.__del__()` is not called (avoiding an infinite loop).
    
    fn dump(inout self):
        print('str1:', self.str1)
        print('str2:', self.str2)

fn use_two_strings():
    let two_strings = TwoStrings4("foo")

# We use a function call to ensure the `two_strings` ownership is enforced
# (Currently, ownership is not enforced for top-level code in notebooks)
use_two_strings()
```

```
str1: foo
str2: bar
Consumed foo
```

您通常不必考虑这一点，但如果您的逻辑具有指向成员的内部指针，则可能需要使它们在析构函数或移动初始值设定项本身中的某些逻辑中保持活动状态。您可以通过分配`_`“discard”模式来做到这一点：

```
fn __del__(owned self):
    self.dump() # Self is still whole here

    consume(self.str1^)
    _ = self.str2
    # self.str2.__del__(): Mojo destroys str2 after its last use.
```

在这种情况下，如果以某种方式`consume()`隐式引用某个值`str2`，这将确保`str2`直到最后一次使用丢弃模式访问它时才被销毁`_`。

### 定义析`__del__`构函数[](#defining-the-__del__-destructor)

您应该定义`__del__()`方法来执行类型所需的任何类型的清理。通常，这包括释放任何不平凡或不可破坏的字段的内存 - 一旦不再使用任何平凡和可破坏的类型，Mojo 就会自动销毁它们。

例如，考虑这个结构：

```
struct MyPet:
    var name: String
    var age: Int

    fn __init__(inout self, owned name: String, age: Int):
        self.name = name^
        self.age = age
```

不需要定义该`__del__()`方法，因为`String`它是可破坏的（它有自己的`__del__()`方法），一旦不再使用（即`MyPet`不再使用实例时），Mojo 就会销毁它，并且`Int`是一个[普通类型](https://docs.modular.com/mojo/programming-manual.html#trivial-types)，Mojo 会回收这个记忆也尽快（虽然有点不同，而不需要一个`__del__()`方法）。

然而，以下结构必须定义`__del__()`释放为其分配的内存的方法`Pointer`：

```
struct Array[Type: AnyType]:
    var data: Pointer[Type]
    var size: Int

    fn __init__(inout self, size: Int, value: Type):
        self.size = size
        self.data = Pointer[Type].alloc(self.size)
        for i in range(self.size):
            self.data.store(i, value)
            
    fn __del__(owned self):
        self.data.free()
```

## 生命周期[](#lifetimes)

TODO：解释返回引用如何工作，与与参数相吻合的生命周期相关联。该功能尚未启用。

## 类型特征[](#type-traits)

这是一个非常类似于 Rust 特征或 Swift 协议或 Haskell 类型类的功能。请注意，这尚未实施。

## 高级/晦涩的 Mojo 功能[](#advancedobscure-mojo-features)

本节介绍对于构建标准库最底层非常重要的高级用户功能。这一级别的堆栈包含一些狭窄的功能，需要具备编译器内部的经验才能有效地理解和利用。

### `@register_passable`结构装饰器[](#register_passable-struct-decorator)

处理值的默认模型是它们存在于内存中，因此它们具有标识，这意味着它们间接地传入和传出函数（等效地，它们在机器级别“通过引用”传递）。这对于无法移动的类型非常有用，并且对于大型对象或具有昂贵复制操作的事物来说是安全的默认值。然而，对于像单个整数或浮点数这样的微小事物来说，它的效率很低。

为了解决这个问题，Mojo 允许结构体选择在寄存器中传递，而不是使用装饰器通过内存传递`@register_passable`。`Int`您将在标准库中的类型上看到此装饰器：

```
@register_passable("trivial")
struct Int:
    var value: __mlir_type.`!pop.scalar<index>`

    fn __init__(value: __mlir_type.`!pop.scalar<index>`) -> Self:
        return Self {value: value}
    ...
```

基本`@register_passable`装饰器不会改变类型的基本行为：它仍然需要有一个`__copyinit__`可复制的方法，可能仍然有一个`__init__`and`__del__`方法等。这个装饰器的主要影响是在内部实现细节上：`@register_passable`类型通常是传入的机器寄存器（取决于底层架构的详细信息）。

对于典型的 Mojo 程序员来说，这个装饰器只有一些可观察到的效果：

1. `@register_passable`类型无法保存不是其本身的类型的实例`@register_passable`。
2. 类型的实例`@register_passable`不具有可预测的标识，因此`self`指针不稳定/不可预测（例如在哈希表中）。
3. `@register_passable`参数和结果直接暴露给 C 和 C++，而不是通过指针传递。
4. 这种类型的和方法是隐式静态的（就像在 Python 中一样），并按值返回其结果而不是`__init__`采用。`__copyinit__``__new__``inout self`

我们预计该装饰器将普遍用于核心标准库类型，但对于一般应用程序级代码可以安全地忽略。

上面的例子`Int`实际上使用了这个装饰器的“简单”变体。它改变了如上所述的传递约定，但也不允许复制和移动构造函数和析构函数（将它们全部简单地综合起来）。

> TODO：Trivial 需要与其自己的装饰器解耦，因为它也适用于内存类型。

### `@always_inline`装饰者[](#always_inline-decorator)

`@always_inline("nodebug")`：同样的事情，但没有调试信息，因此您不会进入 Int 上的 + 方法。

### `@parameter`装饰者[](#parameter-decorator)

装饰`@parameter`器可以放置在捕获运行时值的嵌套函数上，以创建“参数”捕获闭包。这是 Mojo 中的一个不安全功能，因为我们目前没有对引用捕获的生命周期进行建模。此功能的一个特殊方面是它允许捕获运行时值的闭包作为参数值传递。

### 魔术师[](#magic-operators)

C++ 代码有许多与值生命周期相交的神奇运算符，例如“placement new”、“placement delete”和“operator=”，它们会在现有值上重新分配。当您使用 Mojo 的所有语言功能并在安全结构之上进行组合时，Mojo 是一种安全语言，但任何堆栈都是 C 风格指针和猖獗的不安全性的世界。Mojo 是一种实用语言，由于我们对与 C/C++ 互操作以及直接在 Mojo 本身中实现安全结构（如 String）感兴趣，因此我们需要一种方法来表达不安全的事物。

Mojo 标准库`Pointer[element_type]`类型是通过 MLIR 中的底层`!kgen.pointer<element_type>`类型实现的，我们希望有一种方法在 Mojo 中实现这些与 C++ 等效的不安全构造。最终，这些将迁移到 Pointer 类型上的所有方法，但在此之前，有些需要作为内置运算符公开。

### 直接访问 MLIR[](#direct-access-to-mlir)

Mojo 提供对 MLIR 方言和生态系统的完全访问。请查看[Mojo 中的低级 IR，](https://docs.modular.com/mojo/notebooks/BoolMLIR.html)了解如何使用`__mlir_type`、`__mlir_op`和`__mlir_type`结构。所有内置和标准库 API 都是通过调用底层 MLIR 构造来实现的，在这样做时，Mojo 有效地充当了 MLIR 之上的语法糖。
