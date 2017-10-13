# Partial Application

Partial application 允许我们向构造函数，函数或者 behaviour 传入部分参数，然后得到一个可以传入剩余参数的东西。

## A simple case

一个简单的例子是创建一个回调函数。比如：

```pony
class Foo
  var _f: F64 = 0

  fun ref addmul(add: F64, mul: F64): F64 =>
    _f = (_f + add) * mul

class Bar
  fun apply() =>
    let foo: Foo = Foo
    let f = foo~addmul(3)
    f(4)
```

这是一个有点蠢的例子，但是它所表达的概念比较清楚。我们在 `foo` 上部分应用 `addmul` 函数，将 `foo` 绑定至 receiver，`3` 绑定到 `add` 参数。然后我们得到一个对象 `f`，他有一个接受一个参数 `mul` 的 `apply` 方法。当它被调用时，它会转换成调用 `foo.addmul(3, mul)`

我们也可以绑定所有的参数：

```pony
let f = foo~addmul(3, 4)
f()
```

或者不进行参数绑定：

```pony
let f = foo~addmul()
f(3, 4)
```

## 非顺序参数(Out of order arguments)

在 Partial application 上使用命名参数可以允许以任意顺序来绑定参数，而不仅仅是从左到右。比如：

```pony
let f = foo~addmul(where mul = 4)
f(3)
```

这里，我们绑定了 `mul` 参数，而 `add` 参数没有被绑定。

## Partially applying a partial application

因为 partial application 返回一个有 apply 方法的对象，我们可以去部分的应用这个结果！

```pony
let f = foo~addmul()
let f2 = f~apply(where mul = 4)
f2(3)
```

## Partial application is an object literal

在底层实现上，我们会为 partial application 组装一个对象字面量。他会对词法作用域中的变量进行捕获(作为 field)，并且有一个 `apply` 方法来减少一些参数。这实际上是语法糖，通过在代码生成前重写 partial application 的抽象语法树来转为对象字面量。

这意味着 partial application 使用一个匿名类并返回一个 `ref`。如果你需要另一种 reference capability，你可以在 `recover` 表达式中包裹 partial application。
