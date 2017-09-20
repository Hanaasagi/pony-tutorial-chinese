# Actors

__actor__ 和 __类(class)__ 相似，但是有一个重要的区别：actor 可以具有 __behaviours__。

## Behaviours

__behaviour__ 和 __function__ 相似，除了函数是 _同步的(synchronous)_ 而行为是 _异步的(asynchronous)_。换句话说，当你调用一个函数，函数体中的代码立即就会执行，调用的结果就是函数体的结果。这就像其他面向对象语言中的方法调用。

但当你调用一个行为，其中的代码不会立刻执行。相反，行为的代码体会在未来的某个不确定的时间执行。

一个行为酷似一个函数，但是不是由 `fun` 关键字引入，而是由 `be` 关键字引入。

就像一个函数，一个行为能有参数。Unlike a function, it doesn't have a receiver capability (a behaviour can be called on a receiver of any capability) and you can't specify a return type./*不是很理解*/

__behaviour 返回什么？__ behaviour 总是返回 `None`，就像一个没有明确返回类型的函数，因为他们不能返回他们所计算的东西(他们还没有执行)

```pony
actor Aardvark
  let name: String
  var _hunger_level: U64 = 0

  new create(name': String) =>
    name = name'

  be eat(amount: U64) =>
    _hunger_level = _hunger_level - amount.min(_hunger_level)
```

这里我们有一个能异步进食的土豚。非常聪明的土豚。

## 并发(Concurrent)

因为 behaviour 是异步的，可以在某一时刻有多个行为在运行。这正是 Pony 所做的。Pony 的运行时(runtime)有着她自己的调度器(scheduler)，默认有着和你机器 CPU 核心数相等的线程。每一个调度器线程能在任何给定的时间执行一个 actor 的 behaviour，所以 Pony 是天然并发的。

## 时序(Sequential)

actors 是有序的。每一个 actor 每次只执行一个行为。这表明在写 actor 中的时，不需要考虑并发问题：不需要锁或者信号量或者任何类似的东西。

当你写 Pony 代码时，能够意识到 actor 不是并行单元而是时序单元是一件很好的事情。也就是说，一个 actor 应该按顺序执行某些事物，其他任何事情都可以分解成另一个 actor，以此自动地完成并行。

## 为什么安全？

因为 Pony 有 __capabilities secure type system__。我们在谈论 function receiver reference capabilities 前有有简短的提到过 reference capabilities。更简短的版本是他们是用于使并行安全的类型注解，运行时没有额外的开销。

之后我们会深入 reference capabilities。

## Actors 是轻量的

如果你之前搞过并发编程，你一定知道线程是昂贵的。上下文切换会造成问题，每一个线程需要一个栈(这会消耗许多内存)，你需要许多的锁和其他机制来保证所写的代码是线程安全的。

而 actor 是轻量的。真的十分轻量。相对于一个对象来说，一个 actor 只会多占大约 256 字节的内存空间。字节，不是千字节！并且不存在锁和上下文切换。一个未执行的 actor 除了少量的内存占用外不会占用任何其他资源。

使用成百上千的 actor 来写 Pony 程序是一件正常的事情。

## Actor finalisers

就像类，actor 具有 finalisers。finaliser 的定义方式相同 (`fun _final()`)。所有的保证和限制对于 actor finaliser 也同样适用。此外，actor 在其 finaliser 被调用后不会接收任何消息。
