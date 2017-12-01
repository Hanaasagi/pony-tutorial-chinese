# Use command

要在代码中使用 package 需要使用 __use__ 命令。这将告诉编译器寻找你需要的 package 并且使其中的类型可用。每个需要知道某位于 package 中的类型的 Pony 文件都需要使用 `use`。

`use` 和 Python、Java 的 "import"，C/C++ "#include"， C# "using" 概念相似，但不完全一样。它们位于 Pony 文件的开头，看起来像这样：

```pony
use "collections"
```

这将寻找所有定义于 _collections_ package 中的公有可见类型，并将它们添加到 `use` 所处的命名空间中。这些类型就像局部定义一般，可以在该文件中使用。

例如，包含 _time_ package 的标准库。这包含以下的定义：

```pony
primitive Time
  fun now(): (I64, I64)
```

要访问 _now_ 函数，需要添加 `use`：

```pony
use "time"

class Foo
  fun f() =>
    (var secs, var nsecs) = Time.now()
```

## Use names

正如我们上面所看到的，`use` 将 package 中所有的公有类型添加到文件的命名空间中。这意味着使用一个 package 会定义一些你接下来想要使用的名称，除此之外，如果你在某个文件内使用两个 package，它们可能会定义相同的类型名称，造成冲突。例如：

```pony
// In package A
class Foo

// In package B
class Foo

// In your code
use "packageA"
use "packageB"

class Bar
  var _x: Foo
```

`_x` 的声明会引发错误，因为我们不知道将引用哪个 `Foo`。实际上，甚至不需要到这一行。仅仅使用 `packageA` 和 `packageB` 就足以引发错误。

为了避免这个问题，`use` 允许你指定一个别名。如果你这样做，只有别名会会放到你的命名空间中。可以使用别名作为修饰符来访问 package 中的类型：

```pony
// In package A
class Foo

// In package B
class Foo

// In your code
use a = "packageA"
use b = "packageB"

class Bar
  var _x: a.Foo  // The Foo from package A
  var _y: b.Foo  // The Foo from package B
```

如果你愿意，你可以只给一个 package 起别名。`Foo` 将会添加到你的命名空间中，其指向未起别名的那个 package：

```pony
// In package A
class Foo

// In package B
class Foo

// In your code
use "packageA"
use b = "packageB"

class Bar
  var _x: Foo  // The Foo from package A
  var _y: b.Foo  // The Foo from package B
```

__我能够仅指定 package 的完整路径或者根本不使用 use 么？__ 不行，Pony 中不能这样做。你不能直接引用一个使用 `use` 导入的 package，并且你不能不使用 `use` 就直接指定 package 中的类型。每个你想要使用的 package 都需要 `use`。

__我所能使用的别名有限制么？__ 别名必须以小写字母开头。除此之外，你可以使用任何你想要使用的名称，只要你没有在此文件中用作其他用途。

## Scheme indicators

我们给 `use` 的字符串被称为 _specifier_。它是由一个 _scheme_ indicator 和一个 _locator_ 组成，使用冒号来分隔。scheme indicator 告诉 `use` 我们想要它做什么，例如，包含 package 的 scheme indicator 是 "package"。如果没有冒号，那么 `use` 假定你的意思是 "package"。

下面是等价得到：

```pony
use "foo"
use "package:foo"
```

如果你使用的 locator 字符串包含冒号，比如，一个 Windows 下的绝对路径，那么你 __必须__ 在 scheme specifier 中包含 "package"：

```pony
use "C:/foo/bar"  // Error, scheme "C" is unknown
use "package:C:/foo/bar"  // OK
```

为了使 `use` 能够在不同的操作系统中移植，避免转义字符的困惑，应当始终将 '/' 用作 `use` 中的路径分隔符，即使是在 Windows 上。
