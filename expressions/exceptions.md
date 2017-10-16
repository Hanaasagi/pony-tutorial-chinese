# 异常(Exceptions)

Pony 提供了简单的异常机制来进行错误处理。在任何时候，代码都可能会宣告有一个 `error` 发生了。此时代码会停止执行并回退调用链直到找到一个包裹着它的错误处理程序。这在编译器都会进行完备的检查，所以错误不会导致整个程序崩溃。

## 抛出并处理错误(Raising and handling errors)

使用 `error` 来抛出错误。使用 `try`-`else` 语法来声明错误处理。

```pony
try
  callA()
  if not callB() then error end
  callC()
else
  callD()
end
```

上面代码的 `callA()` 和 `callB()` 总是会被执行。如果 `callB()` 的结果是 true 那么会执行 `callC()`，`callD()` 将不会执行。

但是如果 `callB()` 返回 false，那么会抛出错误。此时将停止执行并且会寻找并执行最近的异常处理程序。在这个例子中是我们的 `else` 块，因此 `callD()` 将会执行。

在任何情况下都会继续执行 `try end` 之后的代码。

__我必须给出异常处理代码么？__ 不，`try` 块会捕获任何错误。如果你不提供错误处理程序，则不会执行错误处理操作。接着将执行 `try` 表达式之后的代码。

如果你想做的事情可能会抛出一些你并不在乎的错误，你可以将他们放在没有 `else` 的 `try` 块中。

```pony
try
  // Do something that may raise an error
end
```

__有什么是我的错误处理程序必须做的么？__ 没有。如果你提供了错误处理程序，那么你必须包含一些代码，但这些完全由你自己来做。

__try 块的结果值是什么？__ `try` 块的结果是 `try` 块中最后一个语句的值，或者为 `else` 子句中的最后一个语句的值(抛出错误时)。 如果出现错误，并且没有提供 `else` 子句，结果值将为 `None`。

## 偏函数(Partial functions)

Pony 不要求所有的错误都像先前的例子那样立即被处理。相反，函数可以抛出错误，由调用方来进行处理。这些被称为偏函数(这是一个数学术语意味着函数没有为所有可能的输入(比如参数)预定结果)。偏函数必须使用 `?` 标记。`?` 需要同时位于函数签名(返回类型之后)和调用处(在圆括号之后)。

例如，如果输入一个负数，某些版本的计算有符号整数的阶乘函数将会出错。它只对合法的部分输入类型进行了定义。

```pony
fun factorial(x: I32): I32 ? =>
  if x < 0 then error end
  if x == 0 then
    1
  else
    x * factorial(x - 1)?
  end
```

每处可能产生错误的代码(error, 偏函数调用，或者某些内建结构)必须放在 `try` 块中或者一个偏函数。这会在编译时进行检查，确保错误不会逃匿使程序崩溃。

在 Pony 0.16.0 之前，调用偏函数时不需要使用 `?` 标记。阅读代码时，这经常会在控制流上造成困惑。在每个偏函数上使用标记会使读者马上明白这处代码块可能会跳到最近的错误处理上，使控制流更加清晰明确。

## Partial constructors and behaviours

构造函数也可以为 partial 的。如果一个构造函数抛出错误，那么构造被认为失败，正在构造的对象被丢弃，不会返回给调用者。

当 actor 的构造函数被调用，actor 被创建并立即返回它的引用。然而，构造函数的代码是异步执行的。如果一个 actor 的构造函数抛出异常，可能延期报告给调用者。因此，actor 的构造函数不能为 partial。

behaviours 也是异步执行的，同理也不能为 partial。

## Try-then blocks

除了错误处理 `else`，`try` 也可以有 `then`。这是在 `try` 的其余部分执行后执行，无论错误是否被抛出或者处理。根据先前的例子扩展一下：

```pony
try
  callA()
  if not callB() then error end
  callC()
else
  callD()
then
  callE()
end
```

`callE()` 总是会被执行。如果 `callB()` 返回 true，那么执行顺序是 `callA()`, `callB()`, `callC()`, `callE()`。如果 `callB()` 返回 false，那么执行顺序是 `callA()`, `callB()`, `callD()`, `callE()`。

__我必须在有 else 的情况下才能有 then 么？__ 不，你可以使用 `try`-`then` 如果你想要的话。

__我的 then 真的总是会执行么，即使我在 try 内使用 return？__ 是的。当 `try` 执行完后，你的 `then` 表达式 __总是__ 会执行。惟一的方法是 `try` 不会执行完(由于无限循环，机器被关机，或者进程被杀死)。

## With blocks

`with` 表达式用于确保在不在需要对象时销毁对象。常见的使用场景是在数据库连接使用完后进行关闭以防止服务器上资源泄露。例如：

```pony
with obj = SomeObjectThatNeedsDisposing() do
  // use obj
end
```

`obj.dispose()` 将在 `with` 代码块完成后或者抛出错误后被调用。为了使用 `with` 表达式，对象需要提供一个用于资源清理的 `dispose()` 方法：

```pony
class SomeObjectThatNeedsDisposing
  // constructor, other functions

  fun dispose() =>
    // release resources
```

也可以提供一个 `else` 字句，发生错误时会调用。

```pony
with obj = SomeObjectThatNeedsDisposing() do
  // use obj
else
  // only run if an error has occurred
end
```

可以设置多个进行销毁的对象：

```pony
with obj = SomeObjectThatNeedsDisposing(), other = SomeOtherDisposableObject() do
  // use obj
end
```

`with` 表达式的值是代码块中最后一个表达式的值，或者是有错误发生时的 `else` 块中最后的表达式的值。

## Language constructs that can raise errors

除了 `error` 和调用偏函数外，`as` 也可能抛出错误。如果可以的话，`as` 会将给定值转换为指定的类型。如果不可以，则会抛出错误。这意味着 `as` 只能在 try 或偏函数中使用。

## Comparison to exceptions in other languages

Pony 的异常和 C++, Java, C#, Python, Ruby 等十分相似。关键的区别是 Pony 的异常没有相关联的类型或者实例。这使得它们和 C++ 的异常相似，如果一直抛出固定字面量的话，比如 `throw 3;`。这种差异简化了程序员的异常处理，并带来了更好的运行时错误处理性能。

`try` 表达式中的错误处理 `else` 部分和 C++ 的 `catch(...)`，Java/C# 的 `catch(Exception e)`，Python 的 `except:`，Ruby 的 `rescue` 一样。由于异常没有类型，因此不需要错误处理程序来指定类型或在 try 块中具有多个处理程序。

`try` 表达式的 `then` 块和 Java/C#/Python 的 `finally`， Ruby 的 `ensure` 相同。

如果需要的话，错误处理可以通过使用 `error` 来重新抛出错误。
