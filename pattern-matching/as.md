# As Operator

Pony 中的 `as` 运算符有两个用途。首先，它提供了一种安全的方式去提高对象类型的特殊性(类型转换)。其次，它为程序员提供了一种方式去在数组字面量中指定元素的类型。

## Safely converting to a more specific type (casting)

如果可能的话，`as` 运算符可以被用于创建一个比给定引用的类型更加具体的对象引用。这可以应用于通过继承，union 和 intersection 来关联的类型。这是在运行时进行的，如果失败会抛出错误。

让我们看一个例子。`json` package 提供了一个名为 `JsonDoc` 的类型，可以将字符串解析为 JSON 片段。解析的值储存在对象的 `data` field 中。此 field 的类型是 一个 union，`(F64 | I64 | Bool | None | String | JsonArray | JsonObject)`。所以如果有一个 `JsonDoc` 对象被一个 `jsonDoc` 所引用，那么 `jsonDoc.parse(42)` 将会存储一个等于 `42` 的 `I64` 在 `jsonDoc.data` 中。如果程序员想要将 `jsonDoc.data` 视为 `I64` 那么可以通过 `jsonDoc.data as I64` 来获取对数据的 `I64` 引用。

在下面的程序中，命令行参数被解析为 Json。所有可以被解析为 `I64` 类型数字的参数会被累加，其他参数会被忽略。

```pony
use "json"

actor Main
  new create(env: Env) =>
    var jsonSum: I64 = 0
    let jd: JsonDoc = JsonDoc
    for arg in env.args.slice(1).values() do
      try
        jd.parse(arg)?
        jsonSum = jsonSum + (jd.data as I64)
      end
    end
    env.out.print(jsonSum.string())
```

使用参数 `2 and 4 et 7 y 15` 来运行程序，会输出 `28`。

接口可以做到同样的事情，使用 `as` 来创建一个更加具体的接口或者类。比如，你有一个库和类似啮齿动物的毛茸茸的生物相关。它提供了一个 `Critter` 接口，程序员可以用于创建指定类型的小动物。

```pony
interface Critter
  fun wash(): String
```

程序员使用这个库来创建一个 `Wombat` 和一个 `Capybara` 类。但是 `Capybara` 类提供了一个新的方法，`swim()`，这不是 `Critter` 类的一部分。程序员想要将所有的小动物存储在一个数组中，以便对整个群体进行操作。现在假设当 capybara 清洗完身体后想要去有用。程序员可以通过使用 `as` 来将 `Array[Critter]` 中的每一个 `Critter` 对象视为一个 `Capybara`。如果 `Critter` 不是一个 `Capybara`，那么将会抛出错误；该程序可以略过这个错误，并继续下一个迭代。

```pony
class Wombat is Critter
  fun wash(): String => "I'm a clean wombat!"

class Capybara is Critter
  fun wash(): String => "I feel squeaky clean!"
  fun swim(): String => "I'm swimming like a fish!"

actor Main
  new create(env: Env) =>
    let critters = Array[Critter].push(Wombat).push(Capybara)
    for critter in critters.values() do
      env.out.print(critter.wash())
      try
        env.out.print((critter as Capybara).swim())
      end
    end
```

## Specify the type of items in an array literal

`as` 运算符可以用于告知编译器数组字面量中元素的类型。在很多情况中，编译器可以推断处类型，但是有时是不明确的。

举个例子，在下面的程序中，`foo` 方法可以使用 `Array [U32] ref` 或 `Array [U64] ref`作为参数。如果一个数组字面量被作为参数传给方法，并且没有指定类型，那么编译器会无法推断，因为有两种有效的形式。

```pony
actor Main
  actor Main
  fun foo(xs: (Array[U32] ref | Array[U64] ref)): Bool =>
    // do something boring here

  new create(env: Env) =>
    foo([as U32: 1; 2; 3])
    // the compiler would complain about this:
    //   foo([1; 2; 3])
```

请求的类型必须是数组中元素的有效类型。由于在编译时会对这些类型进行检查，可以保证能够工作，所以程序员不需要错误处理条件。
