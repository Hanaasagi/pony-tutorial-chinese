# Hello World: 她做了什么

让我们来回顾 `helloworld` 的代码：

```pony
actor Main
  new create(env: Env) =>
    env.out.print("Hello, world!")
```

一行一行来看。

## Line 1

```pony
actor Main
```

这是一个 __类型声明(type declaration)__。关键字 `actor` 意味着我们将要定义一个 actor，这和 Python, Java, C#, C++ 等语言的 class 有点相似。Pony 也有 class，我们之后会见到。

actor 和 class 间的区别是 actor 能具有 __asynchronous__ 方法，这被称作 __behaviours__。之后会详细讨论。

一个 Pony 程序需要有一个 `Main` actor。这和 C 和 C++ 中的 `main` 函数， Java 的 `main` 方法， C# 的 `Main` 方法比较相像。她是执行入口。

## Line 2

```pony
  new create(env: Env) =>
```

这是一个 __constructor__。`new` 关键字意味着她是一个创建类型的实例的函数。这个例子中，她创建了一个新的 __Main__。

与其他语言不同的是，在 Pony 中 constructor 具有名字。这会导致有多种方法去构造一个类型的实例。这个例子中，constructor 的名字是 `create`。

紧接着是函数的参数。这个例子中，我们的构造器接受单一参数： `Env` 类型的 `env`。

在 Pony 中，类型置于参数名称的后面，二者使用 `:` 分隔。在 C, C++, Java or C# 中，你会这样 `Env env`。但我们采取了相反的做法，像 Go, Pascal。

事实证明，我们的 `Main` actor __必须有__ 一个参数为 `Env` 类型的名为 `create` 的 constructor。这是所有程序的起始！所以本质上您的程序是从 constructor 的 body 所开始。

__等一下，body 是什么？__ 她是位于 `=>` 之后的代码。

## Line 3

```pony
    env.out.print("Hello, world!")
```

这是您的程序！她做了些什么？

Pony 和大多数语言相似， `.` 是字段(英文为 field，或许属性更贴切)访问或者方法调用。如果名称后跟着圆括号，它是方法调用。否则是字段访问。

所以在这里，我们首先引用 `env`。然后在我们的 `env` 对象上查找 `out` 字段。该字段表示 __stdout__，通常意味着输出到您的控制台。然后，我们在 `env.out` 上调用 `print` 方法。括号中的内容是函数的参数，这个例子中，我们传入了 __字符串字面量(string literal)__，即双引号之内的内容。

Pony 中字符串字面量可以被双引号包围，这种方式遵循 C/C++ 风格的转义(像 `\n` 这样)。或者可以被三引号 `'''` 包围，与 Python 的三引号字符串相似，这会被认为是原始数据。

__Env 究竟是什么？__ 她是您程序的被调用时的 "environment"。这意味着，她具有命令行参数，环境变量，__stdin__, __stdout__, 和 __stderr__。Pony 没有全局变量，所有的这些东西都是显式地传入你的程序。

## 大功告成!

真的，就是这样。程序始于创建一个 `Main` actor，并在 constructor 中，我们打印了 "Hello, world" 到 __stdout__。接下来，我们将开始学习 Pony 的类型系统。
