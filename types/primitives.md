# Primitives

__primitive__ 和 __class__ 相近，但是有两个重要的区别：

1. primitive 没有域(field)
2. 自定义的 primitive 只能存在一个实例

没有域就意味着 primitive 是不可变的。单实例意味着如果您的代码调用 primitive 类型的构造函数，总是会得到相同的结果(除了内建的 "machine word" primitive，下面会提到)

## 您可以使用 __primitive__ 来做什么？

primitive 有三种主要的用途(如果您将内建的 "machine word" 算在内，就是四个)

1. 作为 "marker value"。举个例子，Pony 经常使用 __primitive__ `None` 来表示值不存在的情况。

2. 作为一种枚举类型。使用由 primitive 组成的 __union__，您可以获得类型安全的枚举类型。关于 __union__ 我们之后会讲到。

3. 作为"函数集合"。因为 primitive 可以有函数，您可以将函数聚集在 primitive 类型中。您在标准库中可以见到这种用法，比如关于路径处理的函数都被放到 `Path` primitive中。

primitive 是十分强大，特别是作为枚举。与其他语言的枚举类型不同，Pony 的枚举中的每一个值都是一个完整的类型，这使得对枚举值附加数据和函数变得容易。

## 内建 primitive 类型

__primitive__ 关键字可以被用于引入具体的内建 “machine word” 类型。除了有相关联的值外，她们和自定义的 primitive 相似。她们是：

* __Bool__. `true` 或 `false` 仅占一位。
* __ISize, ILong, I8, I16, I32, I64, I128__. 不同长度的有符号整数。
* __USize, ULong, U8, U16, U32, U64, U128__. 不同长度的无符号整数。
* __F32, F64__. 不同长度的浮点数。

__ISize/USize__ 对应着基本数据类型 `size_t`，长度视平台而异。__ILong/ULong__ 对应着基本类型 `long`，长度也因平台而异。Pony 提供的基本类型 `int` 是所有平台都相同的，您可以使用 __I32/U32__ 来使用。

## primitive 的 initialisation 和 finalisation

primitive 有两种特殊函数，`_init` 和 `_final`。`_init` 在任何 actor 启动之前被调用。`_final` 在所有 actor 终止后被调用。这两种函数没有参数。不同 primitive 的 `_init` 和 `_final` 函数的运行是有序的。

常见的使用场景是初始化和清理 C 库，避免由 actor 使用不当带来的风险。
