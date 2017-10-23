# Reference Capabilities

So if the object _is_ the capability, what controls what we can do with the object? How do we express our _access rights_ on that object?

In Pony, we do it with _reference capabilities_.

## Rights are part of a capability

如果你在 UNIX 下打开一个文件，你会得到一个文件描述符(file descriptor)，这个文件描述符是指定对象的 token，但它不是 capability。要成为 capability，我们需要指定权限并打开文件。比如：

```C
int fd = open("/etc/passwd", O_RDWR);
```

现在我们有一个 token 和一些权限：

在 Pony 中，每个 reference 都同时具有一个类型和一个 reference capability。实际上，reference capability 是其类型的一部分。他们允许你指定哪些对象可以和其他 actor 共享，并允许编译器来检查你在做的是否并发安全。

## 基本概念(Basic concepts)

有一些你学要在 reference capabilities 之间理解的简单概念。我们之前已经讨论过一些了，有些可能对你来说已经显而易见了，但是值得在这里重新提起。

__共享可变数据是困难的__

共享可变数据会带来并发问题。如果两个不同的线程能够访问同一条数据，那么他们可能会尝试同时更新数据。最好的情况是这两个线程有不同版本的数据。最坏的情况是更新可能会互相影响导致被垃圾数据覆盖。避免这些问题的标准做法是使用锁来防止数据同时更新。这会造成性能缺陷，并会造成许多 bug。

__不可变数据可以被安全的共享__

任何不可变数据(即不能更改)可以安全地在并发中使用。数据的更新会造成并发问题，由于它是不可变的，所以永远不会被更新。

__隔离数据(Isolated data)是安全的__

如果一个数据块只有一个对其的引用，那么我们称之为 __隔离的(isolated)__。因为只有一个引用，隔离数据不能被多个线程共享，所以不存在并发问题。隔离数据可以在多个线程间传递。只要某一个时刻只有一个线程持有对其的引用，那么仍可以避免并发问题。

__隔离数据可以是复合的__

隔离数据可以是单个字节，但它也可以是内部各个对象间有着多个引用的大型数据结构。隔离数据关键在于对结构整体是否只有一个引用。我们会讨论数据结构的 __隔离边界(isolation boundary)__:

1. 边界外只能有一个指向对象的引用。
2. 边界内部可以有任何数量的引用，但是它们中的任何一个都不能指向外部的对象。

__每个 actor 都是一个线程__

actor 中的代码不会并发执行。这意味着在 actor 内，数据更新不会造成并发问题。只有当我们想要在 actor 间共享数据时才会有问题。

__好的，在并发中安全地共享数据是比较棘手的事情。reference capabilities 能帮到什么？__

通过共享不可变数据和只交换隔离数据，我们可以不借助锁来安全地并发编程。问题在于很难正确地做到这一点。如果你意外地又引用了已经传出去的隔离数据或者更改了共享的不可变变量，那么一切都会出错。你所需要的是编译器来强制你履行承诺。Pony 的 reference capabilities 允许编译器做到这一点。

## 类型修饰符

如果你使用过 C/C++，你可能会对 `const` 比较熟悉，它是一个 _类型修饰符(type qualifier)_ 告诉编译器不允许程序员进行改变。

reference capability 是 _类型修饰符_ 的一种形式，提供了比 `const` 更多的保证(guarantees)！

在 Pony 中，每次使用类型都会有一个 reference capability。这些 capabilities 适用于变量而不是整个类型。换言之，当你定义 `class Wombat`，你不会给予其 reference capability。相反每个 `Wombat` 变量都有自己的 reference capability。

举个例子，某些语言中，你必须定义一个类型表示可变的 `String` 和另一个表示不可变 `String` 的类型。Java 中，有 `String` 和 `StringBuilder`。在 Pony 中，你只定义一个 `class String` 即可，有一些变量是 `String ref`(可变的)，另一些变量是 `String val`(不可变)的。

## reference capabilities 列表

Pony 有六种 reference capabilities，他们都有着严格的定义和使用规则。稍后我们会介绍全部，现在列出他们的名字和用法：

__Isolated__，`iso`。用于隔离数据结构的引用。如果你有一个 `iso` 变量，你会知道没有其他的变量能够访问这份数据。所以你可以按你想要修改，并传递给另一个 actor。

__Value__，`val`。用于不可变数据结构的引用。如果你有一个 `val` 变量，那么你会知道没有人可以改变这份数据。所以你可以读取并与其他 actor 进行共享。

__Reference__，`ref`。用于非隔离可变数据结构的引用，换句话说，"normal" 数据。如果你有一个 `ref` 变量，那么你可以对其进行读取写入数据，可以有多个能够访问这份数据的变量。但是你不能和其他 actor 共享。

__Box__。用于对你来说只读的数据。数据可能是不可变的，并与其他 actor 共享或者在你的 actor 中有其他变量可以改变其数据。无论那种方式，`box` 变量能够被用于安全地读取数据。可能听起来没有意义，但是他允许你编写可以同时工作于 `val` 和 `ref` 变量的代码，只要不写入对象。

__Transition__，`trn`。用于你想要写入并作为只读(`box`)变量传出的数据结构。如果你想要的话，你可以将 `trn` 变量转换成 `val` 变量，这会阻止所有人改变数据，并且令其可以与其他 actor 进行共享。

__Tag__。用于标识符引用。您不能使用 `tag` 变量读取或写入数据。但是你可以存储、比较 `tag` 来检查对象的标识，并且能够和其他 actor 共享。

请注意，如果你有一个变量引用一个 actor，那么你可以向此 actor 发送消息而不管这个变量有什么样的 reference capability。

## How to write a reference capability

reference capability 写在类型的后面：

```pony
String iso // An isolated string
String trn // A transition string
String ref // A string reference
String val // A string value
String box // A string box
String tag // A string tag
```

__当类型不指定 reference capability 会如何？__ 这意味着你将使用该类型的默认 reference capability，这是在类型定义时声明的。下面是一个来自标准库的例子：

```pony
class val String
```

当我们使用 `String` 时，我们通常意味着一个字符串值，所以我们把　`val`　作为　`String`　的默认 reference capability。

__我必须在类型定义时指定 reference capability 么__ 只要你想要。大多数类型会使用合理的默认值。class 使用 `ref`， primitives(不可变引用)使用 `val`，actor 使用`tag`。
