# Recovering Capabilities

`recover` 表达式可以让你“提升(lift)”结果的 reference capability。可变的 reference capability (`iso`, `trn`, `ref`) 可以转换为 _任何_ reference capability，不可变 reference capability (`val`, `box`) 可以转换为任何不可变或者 opaque 的 reference capability。

## Why is this useful?

`recover` 的最直接用法是获取一个可以传递给另一个 actor 的 `iso`。但它也可以用于做其他的事情，例如：

* 创建一个循环不变的数据结构(cyclic immutable data structure /*不知道什么意思*/)。即，你可以在一个 `recover` 表达式中创建一个复合可变数据结构，然后将结果从 `ref` 提升至 `val`
* 将 `iso` 暂时作为 `ref` 使用("Borrow" an `iso` as a `ref`)，对它进行一些复杂的可变操作，并再次返回 `iso`
* 从 `iso` 中提取出一个可变的 field，并作为 `iso` 返回

## What does this look like?

```pony
recover Array[String] end
```

这是一个简单的例子：它返回一个 `Array[String] iso`，而不是平常的 `Array[String] ref`

标准库中的一个例子：

```pony
recover
  var s = String((prec + 1).max(width.max(31)))
  var value = x

  try
    if value == 0 then
      s.push(table(0)?)
    else
      while value != 0 do
        let index = ((value = value / base) - (value * base))
        s.push(table(index.usize())?)
      end
    end
  end

  _extend_digits(s, prec')
  s.append(typestring)
  s.append(prestring)
  _pad(s, width, align, fill)
  s
end
```

这段代码来自于 `format/_FormatInt`。它创建了一个 `String ref`，进行了一些处理，最后将其作为 `String iso` 返回。

这两个示例没有进行特别指定所有都使用 `recover` 表达式的默认 reference capability。对于任何可变的 reference capability 默认为 `iso`，对于任何不可变 reference capability 默认为 `val`。当然你可以显式的指定：

```pony
let key = recover val line.substring(0, i).strip() end
```

这段代码来自 `net/http/_PayloadBuilder`。我们获取 `line` 的子串，它是 `String iso^` 的，然后我们在其上调用 `strip`，这会返回自身。但是由于 `strip` 函数是 `ref` 的，它会将自身作为 `String ref^` 返回。所以我们使用了 `recover val`。

## How does this work?

在 `recover` 表达式中，你的代码只能访问词法闭包作用域中的 __sendable__ values。换句话说，你只能使用 `recover` 表达式外部的一些为 `iso`，`val`，`tag` 的东西。/*enclosing lexical scope 词法闭包作用域*/

这意味着当 `recover` 表达式结束时，除 `iso`，`val` 和 `tag` 以外类型的表达式结果的别名将不再存在。这样可以安全的去提升表达式结果的 reference capability。

如果 `recover` 表达式能够访问到词法作用域中的 __non-sendable__ values，那么提升结果的 reference capability 是不安全的。Some of those values could "leak" into an `iso` or `val` result, and result in data races.

## Automatic receiver recovery

当你有一个 `iso` 或者 `trn` 的接收者(receiver)时，你不能在其上调用 `ref` 方法。因为接受者也是方法的一个参数，这意味着方法体和调用者都能够同时访问接收者。那么我们必须去给 receiver 起一个别名，`iso` 的为 `tag`，`trn` 的为 `box`。

但是我们可以绕过这个！如果方法调用时的所有参数都是 __sendable__，并且返回类型也是 __sendable__ 或者并没有在调用处被使用，那么可以“自动恢复(automatically recover)”接收者。这样我们便不需要对 receiver 进行别名操作。能够在 `iso` 或者 `trn` 上调用 `ref` 方法了，由于 `iso` 和 `trn` 都是 `ref` 的子类型。

需要注意的是这项技术取决于调用处，而不是被调用的方法的定义。这使得它更灵活。例如，如果被调用的方法需要一个 `ref` 参数，并且我们传递一个 `iso` 参数，那么在调用处是 __sendable__ 的，所以我们仍然可以对接受者自动恢复。

这可能听起来有点复杂，但实际上，这意味着你可以编写像 `ref` 一样处理 `iso` 的代码，并且会编译通过。例如：

```pony
let s = recover String end
s.append("hi")
```

这里我们创建了一个 `String iso` 然后给它添加了一些文本。`append` 方法取一个 `ref` 接收者和一个 `box` 参数。我们将自动恢复 `iso` 接收者因为我们传了一个 `val` 参数，所以一切安好。
