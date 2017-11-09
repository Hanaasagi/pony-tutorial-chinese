# Arrow Types aka Viewpoints

当我们谈论 __reference capability composition__ 和 __viewpoint adaptation__ 时，我们处理了那些已知 origin 的 reference capability 的情况。然而，有时我们并不知道 origin 准确的 reference capability。

当这种情况发生时，我们可以编写一个 __viewpoint adapted type__，我们称其为 __arrow type__，因为我们使用 `->` 符号。

## Using `this->` as a viewpoint

接收者为 `box` 的函数可以被 `ref` 接收者或者 `val` 接收者调用，因为他们都是 `box` 的子类型。有时，我们希望使用某个特别的东西来解决这种问题。例如：

```pony
class Wombat
  var _friend: Wombat

  fun friend(): this->Wombat => _friend
```

这里，我们有一个 `Wombat`，并且每一个 `Wombat` 都有一个同为 `Wombat` 类型的 "friend"。实际上，它是一个 `Wombat ref`，因为 `ref` 是 `Wombat` 的默认 reference capability(由于我们没有显式指定)。我们有一个函数来返回 "friend"。它有一个 `box` 接受者(因为 `box` 是函数默认的 receiver reference capability)。

所以返回类型应当是一个 `Wombat box`。为什么？因为正如我们之前所见，我们从 `box` origin 读取一个 `ref` field，我们获得 `box`。在这种情况下，origin 是接收者，它是 `box`。

但是等等！如果我们想要一个函数当接收者是 `ref` 时返回 `Wombat ref`，当接收者是 `val` 返回 `Wombat val`，当接收者是 `box` 返回 `Wombat box`呢？我们可不想重复写三次。

我们需要使用 `this->`！这样写 `this->Wombat`。这意味着 `Wombat ref` 被视为接收者。

在 _调用处(call site)_ 我们便知道接收者真正的 reference capability。所以当这个函数被调用时，编译器就会知道它所需要的一切。

## Using a type parameter as a viewpoint

目前我们还没有涉及泛型(generics)，所以这可能看起来有点奇怪。当谈论泛型时我们会再次涵盖这个话题(参数化类型(parameterised types))，但是为了使章节完整，我们会先提一下。

另一种是如果我们使用一个类型参数，但我们不知道明确的 reference capability。下面是一个来自标准库的例子：

```pony
class ListValues[A, N: ListNode[A] box] is Iterator[N->A]
```

上面的例子中，我们有一个有两个类型参数 `A` 和 `N` 的 `ListValues` 类型。另外，`N` 有一个约束：它必须是 `ListNode[A] box` 的子类型。但是我们也说 `ListValues[A, N]` 提供了 `Iterator[N->A]`。这一点很有趣：我们提供了一个接口让我们去迭代 `N->A` 类型的值。

这意味着我们会返回类型为 `A` 的对象，但是 reference capability 会和从 `N` 类型对象视角所看到的 `A` 类型对象相同。

## Using `box->` as a viewpoint

还有一种我们需要使用 arrow types 的场景，它也与泛型相关。有时候我们想表达一个类型参数就像就像被未知类型所视，_只要此类型可以读取类型参数_。(There's one more way we use arrow types, and it's also related to generics. Sometimes we want to talk about a type parameter as it is seen by some unknown type, _as long as that type can read the type parameter_.)

换句话说，未知类型将是 `box` 的子类型，但是这是我们知道的。下面是一个来自标准库的例子：

```pony
interface Comparable[A: Comparable[A] box]
  fun eq(that: box->A): Bool => this is that
  fun ne(that: box->A): Bool => not eq(that)
```

在这里，我们说某事物是 `Comparable[A]` 当且仅当如果它有函数 `eq` 和 `ne` 且这些函数都有一个 `box-A` 类型的参数并返回 `Bool`。也就说无论 `A` 绑定什么，我们只需要能够读取它。
