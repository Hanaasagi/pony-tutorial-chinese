# Capability Subtyping

子类型是关于可替代性的概念。也就是说，如果我们需要提供某种类型的东西，我们可以用其他什么类型来替代？reference capabilities 是一种影响因素。

## Simple substitution

首先，让我们来说一下非临时类型(`^`)，非别名类型(`!`)的替换。`<:` 符号表示“是xxx的子类型”或者“可以替换xxx”

* `iso <：trn`。`iso` 是 _读写唯一(read and write unique)_ 的，`trn` 只是 _写唯一(write unique)_ 的，所以用 `iso` 替换 `trn` 是安全的。

* `trn <: ref`。`trn` 是可变的也是 _写唯一(write unique)_ 的。`ref` 是可变的但是并没有做出唯一性保证。所以用 `trn` 去替换 `ref` 是安全的。

* `trn <: val`。这个很有意思。`trn` 是 _写唯一(write unique)_，`val` 是 _全局不可变的(globally immutable)_，那么为什么可以用 `trn` 来替换 `val` 呢？关键在于，为了这样做你必须 _放弃(give up)_ `trn`。如果你放弃了 _仅有(only)_ 的可以写入对象的变量，则不会有变量可以对其进行写入。这意味着可以安全地认为它是 _全局不可变的(globally immutable)_。

* `ref <: box`。`ref` 保证了没有 _其他(other)_ actor 可以读取写入对象。`box` 仅保证了没有 _其他(other)_ actor 可以写入对象，所以可以安全地用 `ref` 替换 `box`。

* `val <: box`。`val` 保证 _没有(no)_ actor 能写入对象(甚至当前的这个)。`box` 只保证了没有 _其他(other)_ 能够写入对象，所以可以安全地使用 `val` 替换 `box`。

* `box <: tag`。`box` 保证没有其他 actor 可以写入对象，`tag` 根本不做保证。所以可以安全地使用 `box` 来替换 `tag`。

子类型是可以传递的(_transitive_)。这就意味着因为 `iso <: trn` 且 `trn <: ref` 且 `ref <: box`，我们便能得出 `iso <: box`。

## Aliased substitution

现在让我们考虑一下当我们有一个 reference capability 的别名时会发生什么。举个例子，如果我们有 `iso`，然后我们创建别名(不做 `consume` 或者 destructive read)，我们得到的类型是 `iso!`，而不是 `iso`。

* `iso! <: tag`。这是一个非常大的变化。`iso!` 仅仅是 `tag` 的子类型，而不是一些类似 `iso` 的东西的子类型。这是因为 `iso` 仍然存在，仍然是 _读写唯一(read and write unique)_ 的。任何别名都不能读取或者吸入对象。这就意味着 `iso!` 只能是 `tag` 的子类型。

* `trn! <: box`。因为 `trn` 是 _写唯一(write unique)_ 的，允许别名去读取对象，但是不允许别名写入对象。这就意味着我们能够有 `box` 或者 `val` 的别名，但 `val` 做出了保证没有别名可以写入对象！因为我们的 `trn` 仍然存在并且可以继续写入对象，`val` 别名会打破这项保证。所以 `trn!` 只能是 `box` 的子类型。

* `ref! <: ref`。因为 `ref` 只能保证 _其他(other)_ actor 既不能读取也不能写入对象，所以可以在同一个 actor 中创建 `ref` 别名。

* `val! <: val`。因为 `val` 只能保证 _没有(no)_ actor 能够写入对象，所以可以创建 `val` 别名因为他们也不能写入对象。

* `tag! <: tag`. A `tag` doesn't make any guarantees at all. Just like with a `box`, we can't make more guarantees when we make a new alias, so a `tag` can only alias as a `tag`.

* `box! <: box`。`box` 只能保证 _other(其他)_ actor 不能写入对象。`val` 和 `ref` 都做出了这样的保证，那么为什么 `box` 只能作为 `box` 的别名？这是因为当我们对某个东西别名化时我们不能做出 _更多(more)_ 的保证。这就导致 `box` 只能作为 `box` 的别名。

* `tag! <: tag`。`tag` 根本没有任何保证。就像 `box` 一样，当我们创建一个新的别名时，我们不能做更多的保证，所以 `tag` 只能作为`tag`的别名。

## Ephemeral substitution

最后一个要考虑的情况是临时 reference capability。举个例子，如果我们有 `iso` 然后我们 `consume` 它或者进行 destructive read，我们得到的类型是 `iso^` 而不是 `iso`。

* `iso^ <: iso`。这很简单。当我们给 `iso^` 一个名字时，通过将它赋值给某个东西或者作为参数传递至某个方法，它会丢失 `^` 然后成为一个普通的 `iso`。因为之前的已经弃掉了，所以有一个新的是安全的。

* `trn^ <: trn`。这和 `iso^` 完全一样。保证更弱一些(_写唯一(write uniqueness)_，而不是 _读写唯一(read and write uniqueness)_，但它的工作原理是一样的。

* `ref^ <: ref^` 和 `ref^ <: ref` 和 `ref <: ref^`。在这里，我们有另一个案例。`ref^` 不仅是 `ref` 的子类型，而且也是 `ref^` 的子类型。这里发生了什么？原因在于 临时 reference capability 是一种表述 "a reference capability that, when aliased, results in the base reference capability" 的概念。由于 `ref` 可以被别名为 `ref` ，这意味着 `ref` 和 `ref^` 是可以互换的。

* `val^`，`box^`，`tag^`。这些工作方式与 `ref` 相同，也就是说，它们可以与自己的 base reference capability 互换。这也是出于同样的原因：所有这些 reference capability 都可以作为自己的别名。

__Why do `ref^`, `val^`, `box^`, and `tag^` exist if they are interchangeable with their base reference capabilities?__ It's for two reasons: __reference capability recovery__ and __generics__. We'll cover both of those later.


__如果他们可以与自己的 base reference capabilities 互换，为什么需要 `ref^`，`val^`，`box^` 和 `tag^` 存在？__ 有两个原因： __reference capability recovery__ 和 __generics__。我们之后会进行介绍。
