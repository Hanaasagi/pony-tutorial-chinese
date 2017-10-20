# Object Capabilities

Pony 的 capabilities-secure 类型系统是基于 object-capability 模型的。这听起来复杂，但实际上优雅且简单。核心思想是：

> capability 是一个委派到对象(a)的不可伪造的令牌(token)，为对对象执行一组特定操作的程序进行授权(b)。

那么 token 是什么？它是一个地址。一个指针，一个引用。它只是...一个对象。

## 如何做到不可伪造的？

由于 Pony 没有指针运算，同时是类型安全的，内存安全的，对象引用不能被程序 "创造"(i.e. 伪造)。你只能通过构造一个对象或者传递对象的方式来获取。

__C FFI 呢？__ 使用 C FFI 会打破这项保证。稍后我们会讨论 __C FFI trust boundary__ 以及如何去控制。

## 全局变量呢？

这真的不是什么好东西！因为你可以不通过构造对象或者传递对象的方式来获得。

全局变量是 _ambient authority_ 的一种形式。另一种形式的 ambient authority 是不受约束的访问文件系统。

Pony 没有全局变量和全局函数。这不意味着所有的 ambient authority 都被消除了——我们仍需要小心的对待文件系统。没有全局变量是必要的，但不足以消除 ambient authority。

__是否和引用透明相似(referential transparency)？__ 是的。As long as you think of the receiver as being passed to a method as well (which it is).

## 这有什么帮助？

object-capabilities 模型没有权限列表，访问控制列表，或其他形式的安全措施。它意味着如果你有一个对象的引用，你可以对该对象进行操作。简单有效。

对象能力模型不是具有权限列表，访问控制列表或其他形式的安全性，它意味着如果您有对象的引用，则可以使用该对象执行操作。简单有效。

有一个关于 object-capability 模型如何工作的论文：

[Capability Myths Demolished](http://srl.cs.jhu.edu/pubs/SRL2003-02.pdf)

## Capabilities and concurrency

object-capability 模型本身并不能解决并发问题。它使得同时访问对象的情况变得清晰，但是并没有规定单一的控制方法。

capabilities 和 actor 模型组合使用是一个好的开端，在像 [E](http://erights.org/)，Joule 之类的编程语言中已早已使用。

Pony 也有这个并在类型系统中使用了 _reference capabilities_ 系统。
