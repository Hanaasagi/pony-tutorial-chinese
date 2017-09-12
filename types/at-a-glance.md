# Pony 类型系统概览

Pony 是一门 _静态类型(statically typed)_ 语言，与 Java, C#, C++ 等相似。这意味着编译器知晓您程序中每一件事物的类型。这有别于 _动态类型(dynamically typed)_ 语言，像是 Python, Lua, JavaScript, 和 Ruby。

## 静态 vs 动态：区别是什么？

在这两类语言中，您的数据都是具有类型的，所以区别是什么？

_动态类型_ 的语言中，一个变量可以在不同时期指向不同类型的对象。这十分灵活，如果您有一个变量 `x`，您可以为其赋值一个整数，然后赋值一个字符串。您的编译器或者解释器并不会为此抱怨。

__但如果在赋值一个整数后，尝试对 `x` 做字符串操作会发生什么呢？__ 通常来说，您的程序会抛出错误。您可能会使用某些方法来处理这个错误，视语言而定，如果您不这么做，您的程序会崩溃。

当您使用 _静态类型_ 语言，一个变量具有一种类型。她只能指向某一种类型的对象(虽然在 Pony 中，一种类型实际上可以为一些类型的集合，我们之后会见到)。如果您有一个期待指向整数的变量 `x`，您便不能将字符串赋值给她。您的编译器会在您尝试运行您的程序之前发出警告。

## 类型即是保证(Types are guarantees)

When the compiler knows what types things are, it can make sure some things in your program work without you having to run it or test it. These things are the _guarantees_ that a language's type system provides.

当编译器了解类型时，她可以确保您程序中的一些部分可以正常工作，无需运行或者测试。这些部分是语言类型系统所提供的 _保证(guarantees)_

类型系统越强大，她所能提供保证便越多，您不必一次次的去运行。

__动态类型也能提供保证？__ 使得，但她们是在运行时去做。举个例子，如果您调用一个不存在的方法，您会得到某些异常。但您只能在运行程序的时候发现。

## Pony 的类型系统为我们提供了哪些保证？

Pony 的类型系统提供了许多保证，甚至超过其他的静态类型语言。

* 编译的程序不会崩溃
* 不会出现未处理的异常
* There's no such thing as `null`, so your program will never try to dereference `null`.
* 没有像 `null` 这样的东西，所以您的程序不会尝试去取消对空指针的引用(dereference `null`)
* 没有竞态问题
* 您的程序绝不会有死锁
* 您的代码总是 capabilities-secure
* All message passing is
[causal](https://www.google.com/search?q=causal+definition). (Not ca**su**al!)

现在您可能认为其中一些是比较好的，另一些对您来说并不意味着什么(如 capabilities-security 和 causal messaging)，但我们稍后会介绍这些概念。

__如果我使用 Pony 的 FFI 去调用其他语言编写的代码， Pony 也会保证这些代码具有上述的优点么？__ 抱歉，不能。Pony 的类型系统仅能应用于 Pony 代码。其他语言的代码由其语言自身提供保证。
