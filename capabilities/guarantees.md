# Reference Capability Guarantees

由于类型是保证，所以谈论 reference capability 所做的保证是有用处的

## 什么会被拒绝(What is denied)

我们将就 what's _denied_ 来谈论 reference capability 的保证。对此，我们说：当你有一个具有某些 reference capability 的变量时，其他变量不能够做什么？

我们需要区分包含上述问题中提到变量的 actor 和包含其他变量的 actor。

这很重要，因为来自其他 actor 的数据读写可能会并行发生。如果两个 actor 能够同时读取同一份数据，若其中一个改变了数据，那么对另一个 actor 也会受影响。这将导致数据竞争，需要使用锁。Pony 通过确保这种情形永远不会发生，消除了使用锁的必要。

同一个 actor 中的代码是顺序执行的。这说明同一个 actor 中多个变量的数据访问不会遭受数据竞争的困扰。

## mutable reference capabilities

__可变(mutable)__ reference capabilities 有 `iso`，`trn` 和 `ref`。这些 reference capabilities 是 __可变的(mutable)__ 因为他们可以被用于对象的读写。

* 如果某个 actor 有一个 `iso` 变量，那么 _任何_ actor 都不能使用其他变量来读写此对象。这意味着一个 `iso` 变量是整个程序中唯的可以读写此对象的变量。它是 _读写唯一的(read and write unique)_。

* 如果某个 actor 有一个 `trn` 变量，那么任何 actor 都不能使用其他变量来对此对象进行写入，并且除此之外的 actor 都不能使用其他变量来读写此对象。这意味着一个 `trn` 变量是程序中唯一能够写入此对象的变量，此 actor 的其他变量可以进行读取。它是 _写唯一的(write unique)_。

* 如果某个 actor 有一个 `ref` 变量，那么 _其他_ actor 都不能使用变量读写此对象。这意味着同一个 actor 中其他的变量可以用于对象的读写。

__为什么它们能够被写入？__ 因为它们都会阻止 _其他_ actor 读取或者写入对象。由于我们知道没有其他的 actor 会去读取，那么我们可以安全地对对象进行写入，不必担心数据竞争(data-races)。同样地，由于我们知道没有其他的 actor 会进行吸入，那么我们可以安全的读取对象。

## Immutable reference capabilities

__不可变(immutable)__ reference capabilities 有 `val` 和 `box`。这些 reference capabilities 是 __不可变的(immutable)__ 因为他们只被用于读取对象，不能写入对象。

* 如果某个 actor 有一个 `val` 变量，那么 _任何_ actor 都不能使用其他变量来对此对象进行写入。这意味着对象不会发生改变。它是 _全局不可变的(globally immutable)_。

* 如果某个 actor 有一个 `box` 变量，那么 _其他_ actor 都不能使用变量来写入此对象。这意味着其他的 actor 可以读取对象并且同一个 actor 中的其他变量能够对其进行写入。无论那种情况，我们都可以安全的读取。这个对象是 _局部不可变的(locally immutable)_。

__为什么他们能够被读取，但却不能写入？__ 因为这些 reference capabilities 只能阻止 _其他_ actor 对对象进行写入。这意味着不能保证 _其他_ actor 不会去读取对象，也就是说写入是不安全的。但同一时间可以有多个 actor 安全地读取对象，所以我们允许去这样做。

## Opaque reference capabilities

只存在一个 __opaque__ reference capability，它是 `tag`。`tag` 变量根本不做任何保证。因此，他不能用于读取对象或者写入对象。

它仍然很有用处：你可以通过它进行标识比较，你可以在它上面调用 behaviours，你可以调用一个只需要 `tag` 作为接受者的函数

__为什么 `tag` 不能读写？__ 因为 `tag` 不会阻止 _其他_ actor 写入对象。这意味着如果我们试图读取，我们不能保证没有其他 actor 在写入对象，所以我们可能会得到一个竞态条件。
