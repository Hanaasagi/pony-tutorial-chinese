# Consume and Destructive Read

Pony capabilities 的一个重要方面是能够去说 “我已经做过这件事情”。我们将介绍两种处理此情况的方法：comsuming a variable 和 destructive reads。

## Consuming a variable

有时，你想要将一个对象从一个变量 _转移(move)_ 到另一个变量。换句话说，你不想为此对象创建一个 _新(new)_ 的名字，准确地说是希望将对象从现有的名字下移动到另一个。

你可以使用 `consume` 来做到这一点。当你 `consume` 一个变量时，你会将值取出，将这个变量置空。直到写入一个新的值前，没有代码能够读取此变量。comsume 一个局部变量或者参数可以让你创建一个别名(类型相同)，即使它是 `iso` 或者 `trn`。比如：

```pony
fun test(a: Wombat iso) =>
  var b: Wombat iso = consume a // Allowed!
```

通过 comsume `a`，就意味着你许诺这个变量不会再被使用。如果你违反这一点，便不能通过编译。

```pony
fun test(a: Wombat iso) =>
  var b: Wombat iso = consume a // Allowed!
  var c: Wombat tag = a // Not allowed!
```

上面是一个例子。当你尝试将 `a` 赋值给 `c` 时，编译器会报错。

__field 能够被 `consume` 么？__ 绝对不行！consuming 某个东西便意味着它是空的，也就是说，它没有值。没有办法确保对象的一些别名不会访问该 field。如果我们试图访问一个空的 field，便会崩溃。但是有一种方法可以做你想做的事情：_destructive read_。

## Destructive read

还有另一种方法去将值从一个名称 _转移(mode)_ 到另一个。之前，我们谈论过在 Pony 中赋值会返回左侧变量的 _旧(old)_ 值，而不是新值。这被称为 _destructive read_，我们可以使用它来做我们想做的事情，

还有另一种方法_move_从一个名称到另一个名称的值。 早些时候，我们讨论了在Pony中的任务如何返回左侧的_old_值，而不是新的值。这被称为_destructive read_，我们可以使用它对 field 进行操作。

```pony
class Aardvark
  var buddy: Wombat iso

  fun test(a: Wombat iso) =>
    var b: Wombat iso = buddy = consume a // Allowed!
```

这里，我们 consume `a`，将其赋给 field `buddy`，并将 `buddy` 的 _旧(old)_ 值赋给 `b`。

__为什么能够 destructively read field 却不能 consume？__ 因为当我们做 destructive read 时，我们会赋值给 field，所以它始终有一个值。不像是 `consume`，不会存在 field 为空的情况。这意味着它是安全的，能够通过编译。
