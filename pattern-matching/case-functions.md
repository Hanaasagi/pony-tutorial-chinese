# Case Functions

case function 给程序员一种方式去创建不同版本的函数；调用何种 case 基于函数调用时的参数。与 `match` 语句一样，通过参数的值或者类型来选择函数，并且可以使用 guard 语句来做额外的控制。case function 有相同的名字和相同数量的形参列表(尽管不一定是相同的参数类型或返回类型)

典型的例子是递归。例如下面的函数可以计算 `n` 的阶乘：


```
       / f(0) => 1
f(n) = |
       \ f(n) => n * f(n - 1)
```

这可以在 Pony 中使用 `if` 语句实现，通过对变量进行检查决定返回 `1` 还是进行递归调用，但也可以使用一组 case functions。`fac_case` 函数通过匹配参数是否是 `0` 来进行分派。而 `fac_guard` 函数使用 guard 语句来测试传入的参数。在这两个函数中，都有一个默认的 case。

```pony
primitive Factorial
  fun fac_conditional(n: U64): U64 =>
    if n == 0 then
      1
    else
      n * fac_conditional(n - 1)
    end

  // dispatch based on argument match
  fun fac_case(0): U64 => 1
  fun fac_case(n: U64): U64 ? => n * (fac_case(n - 1)? as U64)

  // dispatch based on guard function
  fun fac_guard(n: U64): U64 if n == 0 => 1
  fun fac_guard(n: U64): U64 ? => n * (fac_guard(n - 1)? as U64)

actor Main
  new create(env: Env) =>
    let n: U64 = try
      env.args(1)?.read_int[U64]()?._1
    else
      13
    end

    try
      env.out.print(n.string().add("! = ")
                              .add(Factorial.fac_conditional(n).string()))
      env.out.print(n.string().add("! = ")
                              .add((Factorial.fac_case(n)? as U64).string()))
      env.out.print(n.string().add("! = ")
                              .add((Factorial.fac_guard(n)? as U64).string()))
    end
```

请注意在上面的例子中，`fac_case(n)` 和 `fac_guard(n)` 的返回值必须显式地转换为 `U64`。这是因为编译器不会检查以确保所有的 case 都被匹配，而会创建一个隐式的匹配失败会返回 None 基本 case。因此如果 `T` 是一个 case function 的所有显式返回类型的 union，那么函数的隐含类型实际上是 `(T | None)`。

不同 case 的形参类型不需要相同，但是每个 case 需要具有相同数量的参数，且参数需要具有相同的名字。不同 case 可以具有不同的返回类型；如之前所言，返回类型被组合为一个 union(与 `None` 一起)来作为此函数的最终类型。每个参数的类型和名称必须在 case 函数组合的某处指定，不一定全在一个 case 中(参见下面的 `_fizz_buzz` 方法中的 `x` `f` 和 `b` 形参)

在下面的 [FizzBuzz](http://c2.com/cgi/wiki?FizzBuzzTest) 实现中。`fizz_buzz` 是一个取 `U64` 或者 `Range[U64]` 作为参数并返回一个 `String` 或者 `Array[String]` 的 case function。辅助函数 `_fizz_buzz` 返回对应的值。

```pony
use "collections"

class FizzBuzz
  // case functions for FizzBuzz
  fun _fizz_buzz(_, 0, 0): String => "FizzBuzz"
  fun _fizz_buzz(_, 0, b: U64): String => "Fizz"
  fun _fizz_buzz(_, f: U64, 0): String => "Buzz"
  fun _fizz_buzz(x: U64, _, _): String => x.string()

  // parameters are different, return types are different
  fun fizz_buzz(x: U64): String ? =>
    _fizz_buzz(x, x % 3, x % 5) as String
  fun fizz_buzz(x: Range[U64]): Array[String] ? =>
    let acc = Array[String]
    for z in x do
      acc.push(fizz_buzz(z)? as String)
    end

actor Main
  new create(env: Env) =>
    try
      for x in Range[U64](1, 101) do
        env.out.print(FizzBuzz.fizz_buzz(x)? as String)
      end
      let lines = FizzBuzz.fizz_buzz(Range[U64](1, 101))? as Array[String]
      env.out.print("\n".join(lines))
    end
```

注意到 case function 处理匹配重叠的情形时会采取第一个匹配到的函数是很重要的。举个例子，打印 `foo bar` 是因为 `foo` case 所匹配到的类型 `Foo` 是在同样可以匹配的 `Bar` 之前声明的：

```pony
interface Foo

class Bar is Foo
  new ref create() =>
    None

actor Main
  fun foo(x: Foo): String => "foo"
  fun foo(x: Bar): String => "bar"

  fun bar(x: Bar): String => "bar"
  fun bar(x: Foo): String => "foo"

  new create(env: Env) =>
    let x = Bar
    try
      env.out.print((foo(x) as String) + " " + (bar(x) as String))
    end
```

通过明确的说明何种参数会导致哪一条代码路径被执行，case function 可以提高可读性。他们目前是一种语法糖，会被翻译成 `match` 语句，所以比起程序员自己写 `match` 来说并没有额外的开销。

__Pony 支持函数重载么？__ "重载函数" 没有严格的定义，所以很难对这个问题给出明确的答案。case function 提供了一种方式去提供几个同名的，行为根据参数的纸盒类型而改变的函数；对于一些人来说，这看起来像是函数重载。然而，理解 case function 与重载函数的传统用意间的差异是很重要的。

* 当前 case function 的实现没有进行全面地匹配来决定合成函数的返回类型，所以返回类型总是一个包含 `None` 的 union。
* case function 没有提供一种方式去指定相同名称但是参数个数不同的函数。
* 重叠匹配按照声明的顺序来处理。
根据你的需要，case function 或许可以作为重载函数的替换。
