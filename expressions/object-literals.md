# 对象字面量(Object Literals)

有时可以很方便的直接去构造一个对象。在 Pony 中，这被称为对象字面量，它和 JavaScript 的对象字面量差不多。可以创建一个立即使用的对象。

但是 Pony 是静态类型的，所以一个对象字面量会产生一个属于其的匿名类型。这和 Java 和 C# 的匿名类相似，一个匿名类型可以有任意数量的 trait 和 interface。

## 然后那是什么样子的呢?

它看起来大致像类型定义，但有一些小差异。下面是一个简单的对象字面量：

```pony
object
  fun apply(): String => "hi"
end
```

好的，这不值一提。让我们来扩展它，使它显式地提供一个接口，以便编译器能确保匿名类型满足此接口。你可以使用相同的表示来提供 trait。

```pony
object is Hashable
  fun apply(): String => "hi"
  fun hash(): U64 => this().hash()
end
```

我们不能在一个对象字面量中指定构造函数，因为字面量即是构造函数。那么我们如何为字段(fields)赋值呢？。嗯，直接赋值即可。比如：

```pony
class Foo
  fun foo(str: String): Hashable =>
    object is Hashable
      let s: String = str
      fun apply(): String => s
      fun hash(): U64 => s.hash()
    end
```

当我们为一个字段赋值时，我们是在对象字面量所处词法作用域上进行的。这是很有趣的东西！它允许我们有复杂的 __闭包(closures)__，甚至可以有多个入口(entry points)(比如在一个闭包上调用函数)。

默认情况下一个有 fields 的对象字面量是作为 `ref` 返回的，除非在 `object` 关键字后指定 capability 来显式的声明 reference capability。举个例子，如果需要一个对象有 sendable captured reference 的话，可以使用 `iso` 声明。

```pony
class Foo
  fun foo(str: String): Hashable iso^ =>
    object iso is Hashable
      let s: String = str
      fun apply(): String => s
      fun hash(): U64 => s.hash()
    end
```

我们可以通过对象字面量隐式的从词法作用域中获取变量的值。有时他们不是局部变量，不是 fields，也不是函数的形参，这些被称为 _自由变量(free variables)_。By using them in a function, we are _closing over_ them - that is, capturing them. The code above could be written without the field `s`:

```pony
class Foo
  fun foo(str: String): Hashable iso^ =>
    object iso is Hashable
      fun apply(): String => str
      fun hash(): U64 => str.hash()
    end
```

## 匿名函数(Lambdas)

闭包能够很复杂，但有时我们仅想要最简单的闭包。在 Pony 中，你可以使用匿名函数(lambdas)。lambda 是写在花括号中的一个函数(隐式的被命名为 `apply`)：

```pony
{(s: String): String => "lambda: " + s }
```

这会产生与以下代码相同的代码：

```pony
object
  fun apply(s: String): String => "lambda: " + s
end
```

lambda 对象的 reference capability 可以在花括号之后进行声明：

```pony
{(s: String): String => "lambda: " + s } iso
```

这会产生与以下代码相同的代码：

```pony
object iso
  fun apply(s: String): String => "lambda: " + s
end
```

Lambdas 可以像对象字面量那样获取词法作用域中的变量

```pony
class Foo
  new create(env:Env) =>
    foo({(s: String) => env.out.print(s) })

  fun foo(f: {(String)}) =>
    f("Hello World")
```

从词法作用域向一个 field 赋值，可以通过在形参后再添加一个参数列表：

```pony
class Foo
  new create(env:Env) =>
    foo({(s: String)(env) => env.out.print(s) })

  fun foo(f: {(String)}) =>
    f("Hello World")
```

也可以使用 _capture list_ 来创建。capture list是紧随形参的一个列表：

```pony
  new create(env:Env) =>
    foo({(s: String)(myenv = env) => myenv.out.print(s) })
```

lambda 的类型可以使用花括号来声明。在花括号内，函数的形参类型在括号内指定，后跟可选的冒号和返回类型。上例使用 `{(String)}` 作为一个匿名函数的类型并使用 `String` 作为参数，且没有返回值。

如果 lambda 对象没有被声明为特定的 reference capability，那么将从 lambda 的结构进行推断。如果 lambda 没有任何 captured references，默认将使用 `val`；如果有 captured references，默认将使用 `ref`。下面是一个 `val` 的 lambda 对象：

```pony
actor Main
  new create(env:Env) =>
    let l = List[U32]
    l.push(10).push(20).push(30).push(40)
    let r = reduce(l, 0, {(a:U32, b:U32): U32 => a + b })
    env.out.print("Result: " + r.string())

  fun reduce(l: List[U32], acc: U32, f: {(U32, U32): U32} val): U32 =>
    try
      let acc' = f(acc, l.shift()?)
      reduce(l, acc', f)
    else
      acc
    end
```

上例中的 `reduce` 方法需要一个 `val` reference capability 的 lambda 类型的参数 `f`。作为参数传入的 lambda 对象不需要显式的生命一个 reference capability 因为 `val` 是没有捕获任何东西的 lambda 的默认 reference capability。

如前文所述，lambda 是一个有 `apply` 方法的对象字面量的语法糖。`apply` 方法的 reference capability 和其他方法一样默认是 `box`。如果 lambda 需要修改任何捕获的变量或者在他们调用 `ref` 方法，那么这个 lambda 需要为 `ref`。方法的 reference capability (相对于上述对象的 reference capability 来说)是通过将 capability 放在参数列表的括号之前来定义的。

```pony
actor Main
  new create(env:Env) =>
    let l = List[String]
    l.push("hello").push("world")
    var count = U32(0)
    for_each(l, {ref(s:String) =>
      env.out.print(s)
      count = count + 1
    })
    // Displays '0' as the count
    env.out.print("Count: " + count.string())

  fun for_each(l: List[String], f: {ref(String)} ref) =>
    try
      f(l.shift()?)
      for_each(l, f)
    end
```

这个例子声明了 lambda 表达式所生成的 apply 函数为 `ref`。`for_each` 方法中的 `f` 形参的 lambda 类型也声明为`ref`。lambda类型的 reference capability 也必须是 `ref`，以该方法可以被调用。lambda 对象不需要显式地声明 reference capability，因为 `ref` 即是默认值。

## Actor 字面量(Actor literals)

通常来说，一个对象字面量是一个匿名类的实例。要产生一个匿名 actor 的实例，只需要在对象字面量的定义内包含一个或多个 behaviours

```pony
object
  be apply() => env.out.print("hi")
end
```

actor 字面量总是作为 `tag` 返回。

## Primitive 字面量(Primitive literals)

当匿名类型没有 fields 和 behaviours(例如一个对象字面量被作为 lambda 字面量声明)时，编译器会将其作为匿名 primitive 生成，除非一个非 `val` 的 reference capability 被显式的给予。这意味着不需要内存分配来生成该类型的实例。

In other words, in Pony, a lambda that doesn't close over anything has no memory allocation overhead. Nice.

一个 primitive 字面量总是作为一个 `val` 返回。
