# Pony Tutorial

欢迎来到 Pony 的世界！您所阅读的这个将为您打开新世界的大门。这正是我们想要做的。

这份 tutorial 的受众群体是那些已经有一些编程经验的人。这和您是否知道一些 Python，或者一些 Ruby，或者您是一个 JavaScript hacker，亦或是 Java， Scala， C/C ++， Haskell， OCaml 都没什么关系。只要您过去写过一些代码，就 Ok。

## What's Pony, anyway?

Pony 是一门面向对象(object-oriented)，actor-model，capabilities-secure 的编程语言。她是 __面向对象__ 的因为她具有类和对象，就像 Python， Java， C++ 和其他语言那样。它是 __actor-model__ 因为她有着和 Erlang 与 Akka 相似的 _actors_ 概念。这些表现的与对象相似，但是她们能异步地执行代码。Actors make Pony awesome.

当我们提到 Pony 是 capabilities-secure 时，我们的意识：

* 她是类型安全(type safe)的，真正的类型安全。存在着[数学证明](http://www.ponylang.org/media/papers/opsla237-clebsch.pdf)
* 她是内存安全(memory safe)的。好吧，这是类型安全所带来的，但她仍然很有趣。不存在迷途指针，没有缓冲区溢出。这语言甚至没有 _null_ 的概念！
* 她是异常安全(exception safe)的。没有运行时异常。所有异常都具有明确的语义，并且它们 _总是_ 被处理。
* 她没有竞态问题(data-race free)。Pony 没有锁和原子操作或者与之类似的东西。类型系统在 _编译期_ 确保了您的并发程序不会存在竞态问题。所以你可以无忧地编写高度并发的程序。
* 她没有死锁(deadlock free)。这点很容易做到，因为 Pony 根本没有锁！所以他们绝对不会死锁，因为他们不存在。

我们将讨论更多的关于 capabilities-security 的话题，包括 __object capabilities__ 和 __reference capabilities__。

## The Pony Philosophy: Get Stuff Done

In the spirit of [Richard Gabriel](http://www.jwz.org/doc/worse-is-better.html), the Pony philosophy is neither "the-right-thing" nor "worse-is-better". It is "get-stuff-done".

* Correctness. Incorrectness is simply not allowed. _It's pointless to try to get stuff done if you can't guarantee the result is correct._

* Performance. Runtime speed is more important than everything except correctness. If performance must be sacrificed for correctness, try to come up with a new way to do things. _The faster the program can get stuff done, the better. This is more important than anything except a correct result._

* Simplicity. Simplicity can be sacrificed for performance. It is more important for the interface to be simple than the implementation. _The faster the programmer can get stuff done, the better. It's ok to make things a bit harder on the programmer to improve performance, but it's more important to make things easier on the programmer than it is to make things easier on the language/runtime._

* Consistency. Consistency can be sacrificed for simplicity or performance.
_Don't let excessive consistency get in the way of getting stuff done._

* Completeness. It's nice to cover as many things as possible, but completeness can be sacrificed for anything else. _It's better to get some stuff done now than wait until everything can get done later._

The "get-stuff-done" approach has the same attitude towards correctness and simplicity as "the-right-thing", but the same attitude towards consistency and completeness as "worse-is-better". It also adds performance as a new principle, treating it as the second most important thing (after correctness).

## Guiding Principles

Throughout the design and development of the language the following principles should be adhered to.

* Use the get-stuff-done approach.

* Simple grammar. Language must be trivial to parse for both humans and computers.

* No loadable code. Everything is known to the compiler.

* Fully type safe. There is no "trust me, I know what I'm doing" coercion.

* Fully memory safe. There is no "this random number is really a pointer, honest."

* No crashes. A program that compiles should never crash (although it may hang or do something unintended).

* Sensible error messages. Where possible use simple error messages for specific error cases. It is fine to assume the programmer knows the definitions of words in our lexicon, but avoid compiler or other computer science jargon.

* Inherent build system. No separate applications required to configure or build.

* Aim to reduce common programming bugs through the use of restrictive syntax.

* Provide a single, clean and clear way to do things rather than catering to every programmer's preferred prejudices.

* Make upgrades clean. Do not try to merge new features with the ones they are replacing, if something is broken remove it and replace it in one go. Where possible provide rewrite utilities to upgrade source between language versions.

* Reasonable build time. Keeping down build time is important, but less important than runtime performance and correctness.

* Allowing the programmer to omit some things from the code (default arguments, type inference, etc) is fine, but fully specifying should always be allowed.

* No ambiguity. The programmer should never have to guess what the compiler will do, or vice-versa.

* Document required complexity. Not all language features have to be trivial to understand, but complex features must have full explanations in the docs to be allowed in the language.

* Language features should be minimally intrusive when not used.

* Fully defined semantics. The semantics of all language features must be available in the standard language docs. It is not acceptable to leave behaviour undefined or "implementation dependent".

* Efficient hardware access must be available, but this does not have to pervade the whole language.

* The standard library should be implemented in Pony.

* Interoperability. Must be interoperable with other languages, but this may require a shim layer if non primitive types are used.

* Avoid library pain. Use of 3rd party Pony libraries should be as easy as possible, with no surprises. This includes writing and distributing libraries and using multiple versions of a library in a single program.

## More help

Working your way through the tutorial but in need of more help? Not to worry, we have you covered.

If you are looking for an answer "right now", we suggest you give our IRC channel a try. It's #ponylang on Freenode. If you ask a question, be sure to hang around until you get an answer. If you don't get one, or IRC isn't your thing, we have a friendly [mailing list](https://groups.io/g/pony+user) you can try. Whatever your question is, it isn't dumb, and we won't get annoyed.

Think you've found a bug? Check your understanding first by writing the mailing list. Once you know it's a bug, [open an issue](https://github.com/ponylang/ponyc/issues).

# Help us

Found a typo in this tutorial? Perhaps something isn't clear? We welcome pull requests against the tutorial: [https://github.com/ponylang/pony-tutorial](https://github.com/ponylang/pony-tutorial).

Be sure to check out the [contribution guidelines](https://github.com/ponylang/pony-tutorial/blob/master/CONTRIBUTING.md) before opening your PR.
