# Passing and Sharing References

reference capability 使在 actor 间 __传递(pass)__ 可变数据和在 actor 当中 __共享(share)__ 不可变数据变得安全。不仅如此，它们安全地做到了没有复制，没有锁。事实上，没有任何运行时开销。

## Passing

对于一个可变的对象，我们需要确保没有 _其他_ actor 能够读写此对象。有三种可变的 reference capability(`iso`，`trn` 和 `ref`)可以给予保证。

但是如果我们想要将一个可变对象从一个 actor 传递给另一个呢？为了做到这一点，我们需要确保 actor _发送(sending)_ 可变对象的同时也 _放弃(gives up)_ 读写这个对象的能力。

这正是 `iso` 所做的。它是 _读写唯一的(read and write unique)_，同一时刻只能有一个引用可以用于读写。如果你发送一个 `iso` 对象到另一个 actor，你将会放弃读写这个对象的能力。

__所以当我想在 actor 间传递可变的对象时，我应当使用 `iso`么？__ 是的！如果你不需要传递，可以直接使用 `ref`。

## Sharing

如果你想要在 actor 间  __共享(share)__  一个对象，那么需要做出如下保证之一：

1. 没有 actor 可以写入对象，任何 actor 都可以读取
2. 只有一个 actor 可以写入对象，其他 actor 既不能读取也不能写入

第一种保证是 `val` 所做的。它是 _全局不可变的(globally immutable)_，所以我们知道没有 actor 能写入对象。因此，你可以自由的将 `val` 对象发送至其他的 actor，而不必放弃读取对象的能力。

__所以当我想在 actor 间共享不可变对象时，我应当使用 `val` 么_？__ 是的！如果你不需要共享它，你可以使用 `ref` 来替代，或者 `box`。

第二种保证是 `tag` 所做的。Not the part about only one actor writing (that's guaranteed by any mutable reference capability), but the part about not being able to read from or write to an object. 这意味着您可以自由地将 `tag` 对象传递给其他 actor，而不必放弃读取或写入的能力

__发送一个不能读写 field 的 tag reference 到另一个 actor，又有什么意义呢？__ 因为 `tag` 可以用来 __identify__ 对象，有时这正是你所要的。当然如果对象是一个 actor，即使是 `tag` 你也可以调用 behaviour。

__所以当我想在 actor 间共享可变对象的 identity ，我应该使用 `tag`么？__ 是的！ 或者说，任何东西的 identity，无论是可变的，不变的，甚至是 actor。

## Reference capabilities that can't be sent

你可能已经注意到了我们没有提到向其他 actor 传递 `trn`，`ref` 或者 `box`。这是因为你不能这么做。他们不做出我们需要的保证以确保安全。

那么什么时候该使用这些 reference capabilities 呢？

* 当你需要随时随地修改对象时，可以使用 `ref`。另一方面如果你的程序在你使用不可变类型后不会变慢的话，你可能想要使用 `val`。On the other hand, if your program wouldn't be any slower if you used an immutable type instead, you may want to use a `val` anyway./*不理解*/
* 当你不在乎对象是可变还是不可变的时候，可以使用 `box`。换句话说，你希望能够读取，但你不需要写入，也不需要和其他 actor 分享。
* 当你希望能够在一段时间内可以改变对象，但你也希望之后能够使其成为 _全局不可变(globally immutable)_ 的时候，可以使用 `trn`。
