# Chapter 4: Capabilities

/*我真的不知道这该翻译成什么东西*/

我们已经涵盖了 Pony 类型系统的和表达式的基础知识，这章节会介绍 Pony 类型系统的另一个特性 capabilities。目前还没有主流的编程语言有这个特性。capability 是什么呢？

capability 是做"某事"的能力。通常"某事"涉及一个你想访问的外部资源；像是文件系统或者网络。这被称为 object capability。Object capabilities 已经在包含 [E](https://en.wikipedia.org/wiki/E_%28programming_language%29) 在内的一些语言中出现过。

除了 object capabilities 外，Pony 还有另一种 capability，叫做 "reference capability"。object capabilities 是关于赋予对对象进行操作的能力，reference capabilities 是关于拒绝对内存引用进行操作的能力。举个例子，你可以访问内存，但是只能进行读取。你不能进行写入。这是一个 reference capability，它拒绝了你去做某些事情。

capabilities 是 Pony 的核心，它令 Pony 非常别致。你可能记得在这份 tutorial 的介绍处我们这样描述 Pony：

* 她是类型安全(type safe)的，真正的类型安全。存在着[数学证明](http://www.ponylang.org/media/papers/opsla237-clebsch.pdf)
* 她是内存安全(memory safe)的。好吧，这是类型安全所带来的，但她仍然很有趣。不存在迷途指针，没有缓冲区溢出。这语言甚至没有 _null_ 的概念！
* 她是异常安全(exception safe)的。没有运行时异常。所有异常都具有明确的语义，并且它们 _总是_ 被处理。
* 她没有竞态问题(data-race free)。Pony 没有锁和原子操作或者与之类似的东西。类型系统在 _编译期_ 确保了您的并发程序不会存在竞态问题。所以你可以无忧地编写高度并发的程序。
* 她没有死锁(deadlock free)。这点很容易做到，因为 Pony 根本没有锁！所以他们绝对不会死锁，因为他们不存在。

object capability 和 reference capability 使上述这些成为可能

当你读完本章节后，你应当对 capability 是什么以及如何使用有了一个初步的掌握。如果你一开始难以接受，请不要担心。对于大多数人来说，这是一种新的思考代码的方法，需要过一段时间才能掌握。如果你能坚持下去，肯定会有所收获。一旦你使用它们几个星期，capability 上的问题会慢慢减少，不过在那之前可能会很煎熬。别担心，我们都经历了这场斗争，Pony 社区会对你有所帮助。你可以在 Freenode 的 #ponylang IRC 或者 [邮件列表](https://groups.io/g/pony+user) 上找到我们。

Scared? Don't be. Ready? Good. Let's get started with object capabilities.
