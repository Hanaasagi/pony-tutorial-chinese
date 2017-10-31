# 别名(Aliasing)

__Aliasing__ 意味着会有多个变量指向同一个对象。

在大多数语言中，aliasing 很简单。你只需要将变量赋值给另一个变量就行了。被赋值的变量需要具有相同的类型(或者是其父类型)，这样就 ok 了。

在 Pony 中，这种方式适用于一些 reference capabilities，并不是全部。

## Aliasing and deny guarantees

这样做的原因是 `iso` reference capability 会拒绝其他 `iso` 变量指向相同的对象。也就是说，你只能有一个 `iso` 变量指向给定的对象。`trn` 也是如此。

```pony
fun test(a: Wombat iso) =>
  var b: Wombat iso = a // Not allowed!
```

这里的函数接受一个 `iso` 的 Wombat。如果我们试图将 `a` 赋给 `b`，我们会打破 reference capability 的规则，所以编译器会禁止这么做。

__`iso` 能有什么样的别名？__ 因为 _任何_ actor 都不能使用其他的变量来读写此对象，所以我们可以为 `iso` 创建不能读写的别名。幸运的是，`tag` 正是这样一个 reference capability。

```pony
fun test(a: Wombat iso) =>
  var b: Wombat tag = a // Allowed!
```

__`trn` 的别名呢？__ 因为 _任何_ actor 都不能使用其他的变量来写入此对象，所以我们需要不允许写入，但也不会阻止 `trn` 变量写入的东西。幸运的是，我们也有一个这样的 reference capability：`box`。

```pony
fun test(a: Wombat trn) =>
  var b: Wombat box = a // Allowed!
```

__除此之外的呢？__ 其他的都可以使用自身类型作为别名。所以 `ref` 可以作为 `ref` 的别名，`val` 可以作为 `val` 的别名，`box` 可以作为 `box` 的别名，`tag` 也可以作为 `tag` 的别名。

## 什么算作别名？(What counts as making an alias?)

有两种情况可以算作别名：

1. 当你为一个变量 __赋值(assign)__ 时。这可能是一个局部变量或者一个 field。
2. 当你为方法 __传递__ 一个值作为参数时。

在这种情况下，你会为对象创建一个新的 _名称(name)_。这可能是一个局部变量的名称，field 的名称，或者方法形参的名称。

## 临时类型(Ephemeral types)

在 Pony 中，每个表达式都具有类型。那么 `consume a` 的类型是什么？它和 `a` 的类型不同，因为不可能为 `a` 创建别名。相反，它是一个 __临时(ephemeral)__ 类型。他是一个为当前没有名字的值而设计的类型(他可能有一个别名，但是不是我们进行 consume 或者 destructively read 的那个)。

为了表示某个类型是临时的，我们将 `^` 放在后面。例如：

```pony
fun test(a: Wombat iso): Wombat iso^ =>
  consume a
```

这里，我们的函数取一个 `iso` 的 Wombat 作为参数然后返回一个临时的 `iso` Wombat。

这对于处理 `iso` 和 `trn` 及泛型类型来说很有用。对构造函数也同样重要。一个构造函数总是返回一个临时类型，因为它是一个新的对象。

## Alias types

For the same reason Pony has ephemeral types, it also has alias types. An alias type is a way of saying "whatever we can safely alias this thing as". It's only needed when dealing with generic types, which we'll discuss later.

同样地，Pony 有临时类型，它也有别名类型。别名类型是一种表述"whatever we can safely alias this thing as"方式。只有当处理泛型时才需要，我们稍后会提到。

我们通过后面追加 `!` 来表示一个别名类型。下面是一个例子：

```pony
fun test(a: A) =>
  var b: A! = a
```

这里我们使用 `A` 作为一个 __类型变量(type variable)__，`A!` 表示任何 `A` 类型的别名。
