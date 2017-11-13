# Chapter 5: 泛型(Generics)

通常在编写代码时你会创建一些仅在操作的类型上有所不同的类或函数。典型的例子是集合类(collection classes)。你想要能够创建一个通用的 `Array` 而不是一个个地去创建 `IntArray`, `StringArray` 等。这就是泛型大显身手的地方。

## Generic Classes

就像方法可以拥有参数一样，泛型类(generic class)也可以有参数。泛型类的参数是类型(types)。使用方括号将参数引入至类中。

下面是一个非泛型类的例子：

```pony
class Foo
  var _c: U32

  new create(c: U32) =>
    _c = c

  fun get(): U32 => _c

  fun ref set(c: U32) => _c = c

actor Main
  new create(env:Env) =>
    let a = Foo(42)
    env.out.print(a.get().string())
    a.set(21)
    env.out.print(a.get().string())
```

此类仅适用于类型为 `U32`，无符号 32 位整数。我们可以通过使用类型作为类的参数来使其工作于任意类型。它看起来像下面这样：

```pony
class Foo[A: Any val]
  var _c: A

  new create(c: A) =>
    _c = c

  fun get(): A => _c

  fun ref set(c: A) => _c = c

actor Main
  new create(env:Env) =>
    let a = Foo[U32](42)
    env.out.print(a.get().string())
    a.set(21)

    env.out.print(a.get().string())
    let b = Foo[F32](1.5)
    env.out.print(b.get().string())

    let c = Foo[String]("Hello")
    env.out.print(c.get().string())
```

The first thing to note here is that the `Foo` class now takes a type parameter in square brackets, `[A: Any val]`. That syntax for the type parameter is:

首先要注意的是 `Foo` 类现在有了一个参数(方括号中)，`[A: Any val]`。该类型参数的语法是：

    Name: Constraint ReferenceCapability

在上例中，名称是 `A`，约束(constraint)是 `Any`，reference capability 是 `val`。`Any` 用于表示该类型可以为任意类型。类定义的剩余部分使用类型名称 `A` 来替换 `U32`。

使用此类时必须提供一个类型：

```pony
let a = Foo[U32](42)
let b = Foo[F32](1.5)
let c = Foo[String]("Hello")
```

这将告诉编译器使用提供的类型去替换 `A`，然后创建指定的类。举个例子，`Foo[String]` 的用法等同于：

```pony
class FooString
  var _c: String val

  new create(c: String val) =>
    _c = c

  fun get(): String val => _c

  fun ref set(c: String val) => _c = c
```

## 泛型方法(Generic Methods)

方法也可以使用泛型。它们的定义方式除了方法名后跟方括号指定类型参数之外与普通方法相同：

```pony
primitive Foo
  fun bar[A: Stringable val](a: A): String =>
    a.string()

actor Main
  new create(env:Env) =>
    let a = Foo.bar[U32](10)
    env.out.print(a.string())

    let b = Foo.bar[String]("Hello")
    env.out.print(b.string())
```

这个例子展示了一个非 `Any` 的约束。`Stringable` 类型是任意拥有可以转换为 `String` 的 `string()` 方法的类型。

这些例子展示了泛型背后的基本思想以及如何使用泛型。 真实世界的使用情况会复杂得多，下面的章节将会深入讨论如何使用它们。
