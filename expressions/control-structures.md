# 控制结构(Control Structures)

在实际的程序中，你需要能够进行选择，迭代集合中的元素并重复一些操作。为此，你需要控制结构。Pony 拥有程序员熟悉的几种控制结构，像 `if`, `while` 和 `for`，但在 Pony 中它们工作起来略有不同。

## 条件(Conditionals)

最简单的控制结构是 `if`。它允许你在条件为真时执行某些操作。在 Pony 中看起来是这样的：

```pony
if a > b then
  env.out.print("a is bigger")
end
```

__我能在条件中像 C 语言那样使用整数或者指针么？__ 不行。在 Pony 中 `if` 条件必须是 Bool 类型，只能是 true 或者 false。如果你想要测试某个数字 `a` 不是 0，你需要显示的说 `a != 0`。这个限制能避免 Pony 程序中的一些潜在错误。

如果你想要在条件失败时执行一些其他的代码，只需要添加一个 `else`：

```pony
if a > b then
  env.out.print("a is bigger")
else
  env.out.print("a is not bigger")
end
```

通常你想要一次性测试多个条件，产生多种不同的结果。你可以嵌套使用 `if` 语句，但是这随着复杂度上升很快会变得很丑陋：

```pony
if a == b then
  env.out.print("they are the same")
else
  if a > b then
    env.out.print("a is bigger")
  else
    env.out.print("b bigger")
  end
end
```

作为替代 Pony 提供组合了 `else` 和 `if` 的 `elseif` 关键字。这和其他语言中的 `else if` 一样，每个 `if` 可以拥有多个的 'elseif'。

```pony
if a == b then
  env.out.print("they are the same")
elseif a > b then
  env.out.print("a is bigger")
else
  env.out.print("b bigger")
end
```

__为什么不使用像 C 语言那样的 `else if`，而是一个额外的关键字？__ C 语言即一些相似语言中的 `if` 和 `else` 之间的关系是具有歧义的，例如：

```C
// C code
if(a)
  if(b)
    printf("a and b\n");
else
  printf("not a\n");
```

这里 `else` 是属于第一个还是第二个 `if` 是不明显的。实际上这个 `else` 是关联着 `if(b)`。所以示例代码包含一个 bug。Pony 通过不同的方式去处理 `if` `else` 并增加了 `elseif` 来避免这类型的 bug /*`end` 终结了 `if` 的范围*/

## 一切皆表达式(Everything is an expression)

Pony 和其他语言的控制结构的最大区别是在 Pony 中一切皆为表达式。在像 C++ 和 Java 的语言里，`if` 是一个语句，而不是表达式。这就意味着你不能在表达式里含有 `if`，必须去使用一个单独的条件运算符 `?`。

在 Pony 里没有语句只有表达式，一切都有返回值。你的 `if` 会给你返回值。你的 `for` 循环(我们稍后会提到)也会给你返回值。

这意味着你可以在计算中直接使用 `if`：

```pony
x = 1 + if lots then 100 else 2 end
```

这会赋给 __x__ 3 或者 101，取决于变量 __lots__。

如果 `if` 的 `then` 和 `else` 分支返回的类型不同，那么 `if` 返回的实际上是两者的 __union__。

```pony
var x: (String | Bool) =
  if friendly then
    "Hello"
  else
    false
  end
```

__但如果 if 没有 else 呢？__ 任何不存在的 `else` 分支可以认为是隐式的返回 `None`。

```pony
var x: (String | None) =
  if friendly then
    "Hello"
  end
```

__Pony 有条件运算符 "?" 么？__ 不，这不需要。使用 `if` 即可。

## 循环(Loops)

`if` 允许你去选择要做什么，但当要做的事不止一次时你需要循环。

### While

Pony 的 `while` 循环和其他语言的十分相近。一个条件表达式被求值，如果结果为 true 则执行循环体内的代码。当完成后再次计算条件表达式执行循环体内代码直到它的结果为 false。

下面是一个输出数字 1 到 10 的例子：

```pony
var count: U32 = 1

while count <= 10 do
  env.out.print(count.string())
  count = count + 1
end
```

就像 `if`，`while` 也是一个表达式。返回值是最后一次执行的循环体中表达式的结果。对于上面这个例子，返回值是 count 累加到 11 时的 `count = count + 1` 的返回值。因为 Pony 的赋值返回的是 _旧_ 值，所以 `while` 循环将会返回 10。

__但如果条件表达式第一次便为 false，根本不会执行循环内部时会是什么情况呢？__ 在 Pony 中 `while` 表达式也可以有 `else`。通常来说，当 `else` 所关联的表达式没有提供返回值时，`else` 便会承担起提供返回值的职责。如果第一次条件便为 false 则 `while` 没有返回值，所以由 `else` 的返回值作为替代。

__它是否类似于 Python 中 while 循环的 else？__ 不，完全不同。在 Python 中，`else` 会在 `while` 全部执行完后执行。而在 Pony 中，`else` 只在 `while` 未执行时执行。

### Break

有时你想让一次循环半途终止，并完全退出。Pony 对此有 `break` 关键字，与 C++，C# 和 Python 等语言相似。

`break` 会让最内层循环立即退出，由于循环必须返回一个值，所以 `break` 可以为一个表达式。这是可选的，如果没有则返回 `else` 块中的值。

我们举一个例子。假设你想在一些从别处获得的名单里寻找 "Jack" 或者 "Jill"。如果二者都没有出现，你会拿最后一个名字。如果名单是空的你会取 "Herbert"。

```pony
var name =
  while moreNames() do
    var name' = getName()
    if name' == "Jack" or name' == "Jill" then
      break name'
    end
    name'
  else
    "Herbert"
  end
```

所以首先我们去查看是否还有剩余的名字。如果有，我们会去拿一个名字然后看它是否是 "Jack" 或者 "Jill"。如果是则已经完成任务，退出循环返回我们找到的名字。如果不是，我们会再次尝试。

`name'` 出现在循环的尾部。因此如果 "Jack" 和 "Jill" 都没有找到，它会成为循环的返回值。

如果没有名字则 `else` 为我们提供了返回值 "Herbert"。

__我可以像 Java 那样跳出多层嵌套循环？__ 不，Pony 不支持。如果你想要跳出多层循环，你应该重构你的代码或者使用函数。

### Continue

有时你想半途停止某次循环然后开始下一次。像其他语言一样，Pony 使用 `continue` 关键字。

`continue` 停止当前最内层循环并计算条件表达式决定是否进行下一次循环。

如果 `continue` 在最后一次循环中执行，那么循环将没有返回的值。这种情况下，我们使用循环的 `else` 表达式来当作返回值。与`if`表达式相同，如果没有 `else` 表达式，则返回`None`。

__我可以像 Java 那样 continue 一个外层的循环么？__ 不，Pony 不支持。如果你需要 continue 一个外层循环你大概要重构你的代码。

### For

为了迭代一个集合的元素，Pony 使用 `for` 关键字。这和 C# 中的 `foreach`，Python 中的`for` .. `in`，Java 中的 `for` 非常相似。和 C 与 C++ 的 `for` 有所不同。

Pony 的 `for` 循环使用迭代器来迭代集合元素。每次迭代中，我们询问迭代器是否还有元素需要处理，如果有我们则拿出下一个元素。

比如我们想要打印出数组的所有字符串：

```pony
for name in ["Bob"; "Fred"; "Sarah"].values() do
  env.out.print(name)
end
```

注意需要在数组上调用 `values()`，这是因为循环需要一个迭代器而不是数组。

迭代器不必是某种特殊的类型，但是必须提供下面的方法：

```pony
  fun has_next(): Bool
  fun next(): T?
```

其中 T 是集合中对象的类型。除非你要写你自己的迭代器，否则你不需要担心这个。使用已有的集合，例如标准库中提供的集合，你可以直接使用 `for`。如果你想要编写自己的迭代器，请注意要使用 structural typing，因为迭代器不需要声明它提供特定的类型。

你可以认为上面的示例和下面相等：

```pony
let iterator = ["Bob"; "Fred"; "Sarah"].values()
while iterator.has_next() do
  let name = iterator.next()?
  env.out.print(name)
end
```

注意变量 __name__ 使用 _let_ 声明，你不能在循环中再次赋值。

__我可以在 for 循环中使用 break 和 continue 么？__ 可以，`for` 循环也可以有 `else`。也可以用 `break` 和 `continue` 就像 `while` 一样。

### Repeat

Pony 提供的最后一种循环结构是 `repeat` `until`。首先去执行循环体中的表达式然后计算条件表达式是否满足，否则再次执行循环体。

这和 C++ C# 和 Java 的 `do` `while` 类似，除了终止条件是相反的。这些语言在条件表达式为 false 时终止蓄暖，Pony 在条件表达式为 true 时终止循环。

Pony 中 `while` 和 `repeat` 的区别是：

1. `repeat` 至少会执行一次循环体，而 `while` 可能一次都不会执行。
2. 终止条件是相反的。

假设我们在尝试制造某种东西，知道它足够好之前我们会一直尝试：

```pony
repeat
  var present = makePresent()
until present.marksOutOfTen() > 7 end
```

就像 `while` 循环，`repeat` 循环的返回值是循环最后一次执行的表达式的值。在 `repeat` 内也可以使用 `break` 和 `continue`。

__因为你总会执行 repeat 循环至少一次，所以不需要给一个 else 表达式么？__ 不，你可能需要。在 `repeat` 循环最后一次迭代的 `continue` 需要有一个返回值，可以使用 `else` 表达式。
