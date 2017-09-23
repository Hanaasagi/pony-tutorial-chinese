# 类型别名(Type Aliases)

__类型别名(type alias)__ 可以给与类型另一个名字。可能会听起来有点蠢：毕竟，类型都已经有名字了！然而使用它来表示一些复杂的类型是很方便的。

我们将给出一组使用类型别名的例子，来亲身感受她们吧。

## 枚举(Enumerations)

表示枚举是使用类型别名的用途之一。举个例子，想象我们想说的是红色，蓝色或绿色。我们可能会这样写：

```pony
primitive Red
primitive Blue
primitive Green

type Colour is (Red | Blue | Green)
```

这里有两个概念。第一个是类型别名，使用 `type` 关键字引入。她表明 `type` 之后的名字会被编译器翻译成 `is` 之后的类型。

第二个陌生的概念是 `is` 之后的那些类型。她们不是单一类型而是 __union__ 类型。您可以在这个上下文中将 `|` 符号视为 __or__，所以这个类型是 "Red or Blue or Green"。

union 类型是一种形式的 _closed world_ 类型。每一种类型都可能成为她的成员。相比之下，面向对象的 subtyping 通常是 _open world_ 的，比如 Java 中，一个接口可以被任意数量的类实现。

你可以像 C 或者 Go 那样去声明常量，
```pony
primitive Red    fun apply(): U32 => 0xFF0000FF
primitive Green  fun apply(): U32 => 0x00FF00FF
primitive Blue   fun apply(): U32 => 0x0000FFFF

type Colour is (Red | Blue | Green)
```

或者像这样的命名空间

```pony
primitive Colours
  fun red(): U32 => 0xFF0000FF
  fun green(): U32 => 0x00FF00FF
```

你可能会想要像这样迭代枚举类型，输出名称用于调试

```pony
primitive ColourList
  fun tag apply(): Array[Colour] =>
    [Red; Green; Blue]

for colour in ColourList().values() do
end
```

## 复杂类型(Complex types)

如果一个类型较复杂，给她一个助记符可能会好一些。举个例子，如果我们想要一个类型实现多个接口，我们会这样：

```pony
interface HasName
  fun name(): String

interface HasAge
  fun age(): U32

interface HasAddress
  fun address(): String

type Person is (HasName & HasAge & HasAddress)
```

这对 trait 也同样适用：

```pony
trait HasName
  fun name(): String => "Bob"

trait HasAge
  fun age(): U32 => 42

trait HasAddress
  fun address(): String => "3 Abbey Road"

type Person is (HasName & HasAge & HasAddress)
```

这里有另一个新的概念：类型中有一个 `&` 符号。这和 __union__ 类型中的 `|` 相似：她表明这是一个 __intersection__ 类型。意味着 `HasName`， `HasAge` 和 `HasAddress` 三者共有的部分。

在这里 `type` 的使用和上面的枚举例子是相同的。它只是为类型提供一个名字，否则你要一遍一遍的去敲。

另一个例子来自于标准库，她是 `SetIs`。这里给出实际的定义：

```pony
type SetIs[A] is HashSet[A, HashIs[A!]]
```

再一次出现了一些新东西。`SetIs` 之后有一个在方括号内的 `A`。这是因为 `SetIs` 是一个 __泛型(generic type)__。你可以给 `SetIs` 一个类型作为参数，来制造一些具体类型的集合。如果你有使用过 Java 或 C#，应该会比较熟悉。如果你以前用的是 C++，模板是与之相似的概念，但工作机制不同。

`type` 提供了一种方便的途径去引用我们起别名的类型：

```pony
HashSet[A, HashIs[A!]]
```

这是另一个 __泛型(generic type)__。`SetIs` 是一种 `HashSet`。又有一个新的概念 `!` 类型。这是一种另一种类型的 __alias__ 类型。这个会比较棘手，你仅在编写复杂的泛型类型时才会用到，我们之后在讲解。

另一个例子 `Map` 依旧是标准库中的。它实际上是一个类型别名，下面是 `Map` 的真正定义：

```pony
type Map[K: (Hashable box & Comparable[K] box), V] is HashMap[K, V, HashEq[K]]
```

与之前的例子不同，第一个类型参数 `K` 有一个相关联的类型。这是 __约束(constraint)__，这意味着当你参数化一个 `Map` 时，你传递的必须是约束的子类型。

另外，请留意类型里出现的 `box`。这是一个 __reference capability__。表明有一个对 `K` 进行操作的具体类。稍后会详细介绍。

就像我们的其他例子，这一切表明 `Map` 是一种 `HashMap`。

## 其他

上面介绍了类型别名的一般应用，但她还可以做很多事情。记住：类型别名能带来方便，你可以在每个使用类型别名的地方用完整类型来替换。

实际上，这正是编译器所做的。
