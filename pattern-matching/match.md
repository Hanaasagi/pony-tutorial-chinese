# 匹配表达式(Match Expressions)

如果我们想要对表达式和值进行比较，那么我们会使用 `if`。但是如果想同时和多个值进行比较，这就有点无趣了。Pony 提供了强有力的模式匹配功能，组合了值和类型上的匹配，并且不需要任何特殊的代码。

## Matching: the basics

下面是一个匹配表达式的例子，用于生成对应的字符串。

```pony
match x
| 2 => "int"
| 2.0 => "float"
| "2" => "string"
else
  "something else"
end
```

如果你曾使用过函数式语言，这应该令你感到熟悉。

对于那些熟悉 C 或者 Java 系列语言的读者，可以将其看做一个 switch 语句。但是你可以使用整数之外的值，比如字符串。事实上，你可以使用任何提供了比较函数的类型，包含那些你自己实现的类。也可以使用运行时表达式的类型。

匹配需要使用 `match` 关键字，然后是待匹配的表达式，即 match __operand__。对于上例中，操作数(operand)就是变量 `x`。它也可以是任意的表达式。

大多数匹配表达式由一系列的 __cases__ 组成。每个 case 由管道符号(`|`)，进行匹配的 __pattern(模式)__，箭头符号(`=>`)和匹配成功后的返回值组成。

匹配一个接一个地进行，直至匹配成功。(Actually, in practice the compiler is a lot more intelligent than that and uses a combination of sequential checks and jump tables to be as efficient as possible.)

请注意每个 match case 都有一个待求值的表达式并且他们是独立的。case 之间不存在 C 语言中的 "fall through"。

如果匹配表达式产生的值未被使用那么 case 会忽略箭头和待计算的表达式。这可用于在更普遍的情况之前排除特定情况。(If the value produced by the match expression isn't used then the cases can omit the arrow and expression to evaluate. This can be useful for excluding specific cases before a more general case.)

### Else cases

和 Pony 的控制结构一样，匹配表达式的 else case 会在我们没有可以匹配的 case (换句话说，所有的情况都不能匹配)时被使用。else case 必须放在最后。

如果要使用匹配表达式产生的值那么你需要有一个 else case，除非编译器能够清楚地认识到匹配是有穷的。如果你忽略它，则默认求值结果为 `None`。

当所有模式类型的 union 是匹配表达式的父类型时，编译器能够识别出匹配是有穷的。换句话说，当你的 case 足以覆盖匹配表达式所有可能的类型时，编译器不会隐式的添加 `else None`。

## Matching on values

最简单的匹配表达式是对值进行匹配。

```pony
fun f(x: U32): String =>
  match x
  | 1 => "one"
  | 2 => "two"
  | 3 => "three"
  | 5 => "not four"
  else
    "something else"
  end
```

对于值的匹配，模式是我们想要匹配的值，就像 C 的 switch 语句一样。

编译器会调用操作数的 `eq()` 方法，将模式作为参数传入。这就意味着你可以使用你自己的类型来作为操作数和模式，只要你定义了 `eq()` 方法。

```pony
class Foo
  var _x: U32

  new create(x: U32) =>
    _x = x

  fun eq(that: Foo): Bool =>
    _x == that._x

actor Main
  fun f(x: Foo): String =>
    match x
    | Foo(1) => "one"
    | Foo(2) => "two"
    | Foo(3) => "three"
    | Foo(5) => "not four"
    else
      "something else"
    end
```

## Matching on type and value

如果 match operand 和 case pattern 具有相同的类型，那么对值进行匹配就可以了。然而，匹配可以针对多个不同的类型。每一个 case pattern 会先检查是否和操作数的运行时类型相同。只有相同的情况下才会匹配值。

```pony
fun f(x: (U32 | String | None)): String =>
  match x
  | None => "none"
  | 2 => "two"
  | 3 => "three"
  | "5" => "not four"
  else
    "something else"
  end
```

许多使用运行时类型信息的语言，此操作代价是昂贵的，因此通常尽可能避免。

在 Pony 中，这种操作很节能。Pony 的 "whole program" 编译意味着编译器在编译时会尽可能的提取信息。类型检查的运行时开销通常只是单个指针的比较。当然任何可以在编译期局决定的检查都是这样。所以向上转型跟被没有运行时开销。

## Captures

有时你想要匹配某个类型的任何值。为此你需要使用 __capture__。这定义了一个局部变量，只在 case 中有效，包含了操作数的值。如果操作数不是指定的类型，那么将匹配失败。

capture 看起来像在模式中生命一个变量。就像普通的变量一样，他们可以使用 `var` 或者 `let` 生命。如果你不打算在 case 表达式中重新赋值，那么使用 `let` 是一个好习惯。

```pony
fun f(x: (U32 | String | None)): String =>
  match x
  | None => "none"
  | 2 => "two"
  | 3 => "three"
  | let u: U32 => "other integer"
  | let s: String => s
  else
    "something else"
  end
```

__我能够省略 capture 的类型，就像局部变量那样？__ 不幸的是，这是不可能的。因为我们对类型进行匹配，所以编译器需要知道模式是何种类型，这是不能被推断的。

## Matching tuples

如果你想要一次匹配多个操作数，可以使用元组(tuple)。只有元组所有的元素都匹配时，才会视为匹配成功。

```pony
fun f(x: (String | None), y: U32): String =>
  match (x, y)
  | (None, let y: U32) => "none"
  | (let s: String, 2) => s + " two"
  | (let s: String, 3) => s + " three"
  | (let s: String, let u: U32) => s + " other integer"
  else
    "something else"
  end
```

__我必须指定元组中的所有元素么？__ 不必要。元组中的任何元素都可以使用下划线(`_`)来标记为 "don't care"。上例中的第一个和第四个 case 并不关心 `U32` 元素，所以可以忽略。

```pony
fun f(x: (String | None), y: U32): String =>
  match (x, y)
  | (None, _) => "none"
  | (let s: String, 2) => s + " two"
  | (let s: String, 3) => s + " three"
  | (let s: String, _) => s + " other integer"
  else
    "something else"
  end
```

## Guards

除了类型和值的匹配外，匹配中的每个 case 可以具有 guard 条件。这是一个简单的表达式，在类型和值匹配之后执行，若匹配则为 true。如果为 false，那么 case 不会匹配，按照通常的情况步入下一个 case。

Guards 有 `if` 关键字(_0.2.1 前为 `where`_)引入。

guard 表达式可以使用任何从 case 中捕获的值，这可以用于处理范围和复杂的情况。

```pony
fun f(x: (String | None), y: U32): String =>
  match (x, y)
  | (None, _) => "none"
  | (let s: String, 2) => s + " two"
  | (let s: String, 3) => s + " three"
  | (let s: String, let u: U32) if u > 14 => s + " other big integer"
  | (let s: String, _) => s + " other small integer"
  else
    "something else"
  end
```
