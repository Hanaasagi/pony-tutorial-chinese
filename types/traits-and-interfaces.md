# Traits and Interfaces

像其他面向对象语言， Pony 有 __subtyping__。一些类型作为 _categories_，其他类型可以是其成员。

编程语言中存在两种 __subtyping__：__nominal__ 和 __structural__。她们间有着微妙的不同，大多数编程语言仅有其中一种。Pony 两者都有！

## Nominal subtyping

这种 subtyping 被称为 __nominal__ 因为她们全是关于 __names__ 的。

如果你之前使用过面向对象语言，你可能会看到很多关于 _单继承(single inheritance)_，_多继承(multiple inheritance)_，_混入(mixins)_，_traits_ 和一些相似概念的讨论。这些都是 __nominal subtyping__ 的例子。

核心思想是你有一个声明了与一些 category 存在关系的类型。在 Java 中，举个例子，一个 __类(class)__(某种具体的类型) 可以去 __实现(implement)__ 一个 __接口(interface)__(一个 category 类型)。在 Java 里，这意味着此类已经在这个接口所表示的 category 里了。编译器将检查该类是否真正提供了所需的一切。

## Traits: nominal subtyping

Pony 有 nominal subtyping，借助 __traits__。一个 __trait__ 会和 __class__ 看起来有点像，但是她使用 `trait` 关键字并且不能有任何 fields

```pony
trait Named
  fun name(): String => "Bob"

class Bob is Named
```

这里，我们有个叫做 `Named` 的 trait，她有一个返回一个字符串的函数 `name`。同时提供了一个默认实现：返回字符串字面量 "Bob"。

我们也有一个类 `Bob`，她说她是 `is Named` 的。这表明 `Bob` 是在 `Named` 的 category 中。在 Pony 里，我们称 _Bob provides Named_，或者更简洁的说 _Bob is Named_。

因为 `Bob` 没有她自己的 `name` 函数实现，她将使用 trait 中的那个。如果 trait 中的函数也没有默认实现，编译器会对此发出警告。

```pony
class Larry
  fun name(): String => "Larry"
```

我们有一个 `Larry` 类，给出了一个和 `name` 函数有着相同签名的函数实现。但是 `Larry` does __not__ provide `Named`!/*对应上面所说的*/

__为什么？__ 因为 `Larry` 没有说她是 `is Named`。要记住，traits 是 __nominal__：一个类型想要提供一个 trait 必须要显示地声明。`Larry` 没有这样做。

## Structural subtyping

这里有另一种和 name 无关的 subtyping —— __structural subtyping__。她是关于如何构建一个类型的。

一个具体的类型不管她叫做什么，只要拥有所有必要的函数实现，她便是一个 structural category 的成员。

如果你之前使用过 Go，你会意识到 Go 的接口是 structural types。

## Interfaces: structural subtyping

Pony 也有 structural subtyping，借助 __interfaces__。Interfaces 像是 traits，但是使用 `interface` 关键字。

```pony
interface HasName
  fun name(): String
```

这里 `HasName` 和 `Named` 看起来很像，除了她是 interface 而不是 trait。上例表明 `Bob` and `Larry` provide `HasName`!程序员写 `Bob` 和 `Larry` 时甚至不需要知道 `HasName` 的存在。

Pony 的接口也可以有函数的默认实现。某种类型只有在她明确声明她是 `is` 该接口的时候才会选择它们。

__我应该在代码中使用 trait 或者 interface 么？__ Both! interface 更加灵活，所以如果你不清楚你想要什么，那就使用 interface。但是 trait 是一个更强大的工具：他们阻止了 _accidental subtyping_。
