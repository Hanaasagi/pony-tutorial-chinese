# Generics and Reference Capabilities

在之前的例子中，我们显式的将 reference capability 设置为 `val`：

```pony
class Foo[A: Any val]
```

如果类型参数中没有写 reference capability，那么泛型类或者函数可以接受任意 reference capability。这看起来像下面这样：

```pony
class Foo[A: Any]
```

它可以更短一些，因为 `Any` 是默认的约束：

```pony
class Foo[A]
```

如果可以接受任意的 reference capability 则先前的例子看起来像这样：

```pony
// Note - this won't compile
class Foo[A]
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
    env.out.print(c.get().string())
```


不幸的是，这不能通过编译。对于要进行编译的泛型类，必须保证对所有满足类型参数约束的类型和 reference capability 都是可以编译的。上例中，是任意类型和任意 reference capability。如之前所言，此类可以在 reference capability 为 `val` 的情况下工作，那如果是 `ref` 呢？

```pony
class Foo
  var _c: String ref

  new create(c: String ref) =>
    _c = c

  fun get(): String ref => _c

  fun ref set(c: String ref) => _c = c

actor Main
  new create(env:Env) =>
    let a = Foo(recover ref String end)
    env.out.print(a.get().string())
    a.set(recover ref String end)
    env.out.print(a.get().string())
```

能够编译通过且运行，所以 `ref` 是合法的。真正将受到考验的是 `iso`。让我们将这个类转换成 `iso`，然后一点点分析：

## An `iso` specific class

```pony
// Note - this won't compile
class Foo
  var _c: String iso

  new create(c: String iso) =>
    _c = c

  fun get(): String iso => _c

  fun ref set(c: String iso) => _c = c

actor Main
  new create(env:Env) =>
    let a = Foo(recover iso String end)
    env.out.print(a.get().string())
    a.set(recover iso String end)
    env.out.print(a.get().string())
```

编译失败，第一个错误是：

```
main.pony:5:8: right side must be a subtype of left side
    _c = c
       ^
    Info:
    main.pony:4:17: String iso! is not a subtype of String iso: iso! is not a subtype of iso
      new create(c: String iso) =>
                ^
```

这个错误是在说，我们正在为 `String iso` 起一个别名。在 `iso` 的 `!` 表示他是一个已存在的 `iso` 的别名。查看一下出现此错误的代码：

```pony
new create(c: String iso) =>
  _c = c
```

我们有一个 `iso` 的 `c` 并且尝试将其赋给 `_c`。这会为同一个对象创建两个别名，这是 `iso` 所不允许的。为了解决此问题，我们需要 `consume` 这个参数。正确的构造函数应该如下所示：

```pony
new create(c: String iso) =>
  _c = consume c
```

`get` 存在同样的问题。

```pony
fun get(): String iso => _c
```

我们不能返回一个内部 field `_c` 的 `iso` 别名，因为我们再次犯了为同一个 `iso` 别名的错误。一个在类中，一个返回给调用者。我们不能在这里使用 `consume` 因为我们的 `_c` field 没有值。相反，我们可以返回一个可以作为 `iso` 别名的 reference capability。请参考 [capability subtyping aliased substitution rules](../capabilities/capability-subtyping.html)，我们需要的 capability 是 `tag`。这里是正确的类定义：

```pony
class Foo
  var _c: String iso

  new create(c: String iso) =>
    _c = consume c

  fun get(): String tag => _c

  fun ref set(c: String iso) => _c = consume c

actor Main
  new create(env:Env) =>
    let a = Foo(recover iso String end)
//    env.out.print(a.get().string())
    a.set(recover iso String end)
//    env.out.print(a.get().string())
```

这能通过编译但是有一个问题。我们不能打印处 `get` 的结果因为它是一个 `tag`。`tag` 是不允许读或者写的。他只能用于调用 behaviours(ie. on an `actor`)或者用于对象标识。

我们可以使用 [an arrow type](../capabilities/arrow-types.html) 来解决。将 `get` 这么定义：

```pony
fun get(): this->String iso => _c
```

带有 "this->" 的 arrow type 会使用实际的接收者的 capability(上例中为 `ref`)，而不是方法的能力(这里默认为 `box`)。通过 [viewpoint adaption](../capabilities/combining-capabilities.html) 这将是 `ref->iso`。通过 [automatic receiver recovery](../capabilities/recovering-capabilities.html) 的魔法，我们可以在其上调用 `string` 方法。下面的定义可以正常工作：

```pony
class Foo
  var _c: String iso

  new create(c: String iso) =>
    _c = consume c

  fun get(): this->String iso => _c

  fun ref set(c: String iso) => _c = consume c

actor Main
  new create(env:Env) =>
    let a = Foo(recover iso String end)
    env.out.print(a.get().string())
    a.set(recover iso String end)
    env.out.print(a.get().string())
```

## A capability generic class

现在我们让 `iso` 能工作了，并且我们知道如何编写一个能工作于 `iso` 的泛型类。它也可以用于其他的 capabilities：

```pony
class Foo[A]
  var _c: A

  new create(c: A) =>
    _c = consume c

  fun get(): this->A => _c

  fun ref set(c: A) => _c = consume c

actor Main
  new create(env:Env) =>
    let a = Foo[String iso]("Hello".clone())
    env.out.print(a.get().string())

    let b = Foo[String ref](recover ref "World".clone() end)
    env.out.print(b.get().string())

    let c = Foo[U8](42)
    env.out.print(c.get().string())
```

让泛型类或者方法工作于所有的 capability 类型需要付出一些努力，特别是对于 `iso`。有一些方法可以将泛型限制到一些 capabilities 的子集，这是下一节的主题。
