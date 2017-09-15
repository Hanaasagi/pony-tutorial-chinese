# 类

就像其他面向对象语言， Pony 有 __classes__。类由 `class` 关键字声明，并且需要有一个首字母大写的名字，像这样：

```pony
class Wombat
```

__所有类型的名字都是首字母大写的么？__ 没错！除此之外的东西都不是首字母大写的，所以当您在 Pony 代码中发现一个名称时，您马上就会知道她是否为一种类型。

## class 中有什么？

一个 class 由以下几部分构成：

1. 域(Fields)
2. 构造函数(Constructors)
3. 函数(Functions)

### 域(Fields)

这些就像是 C 结构体中的域或者 C++, C#, Java, Python, Ruby 等语言的类中的域。有三种类型的域：`var`， `let` 和 `embed`。一个 `var` 域可以被重复赋值，而 `let` 域只能在构造函数中赋值。`embed` 域将在 [variables](../expressions/variables.md) 有更详尽的介绍。

```pony
class Wombat
  let name: String
  var _hunger_level: U64
```

这里， `Wombat` 含有 `String` 类型的 `name` 和 `U64` 类型(无符号64位整数)的 `_hunger_level`

__前置下划线起什么作用？__ 她意味着这些东西是 __私有__ 的。__私有__ 域仅能被同类型的代码所访问。一个 __私有__ 的构造函数，函数，或者行为(behaviour) 仅能被同一个包(package)中的代码所访问。关于 package 我们之后会讲解。

### Constructors

Pony 的构造函数具有名字。除此之外和其他语言的构造函数没什么不同。她们有参数，并且总是返回一个类型的新实例。由于她们具有名字，所以您可以为某个类型创建多个构造函数。

构造函数使用 __new__ 关键字引入。

```pony
class Wombat
  let name: String
  var _hunger_level: U64

  new create(name': String) =>
    name = name'
    _hunger_level = 0

  new hungry(name': String, hunger': U64) =>
    name = name'
    _hunger_level = hunger'
```

这里，我们有两个构造函数。一个创建并不饥饿的 `Wombat`，另一个创建可能饥饿的 `Wombat`。与使用方法重载来区分不同构造函数的语言不同，Pony 不会根据参数的数目和类型来推测要调用哪个候补的构造函数。需要使用 `.` 语法调用方法来选择构造函数。

```pony
let defaultWombat = Wombat("Fantastibat") // Invokes the create method by default
let hungryWombat = Wombat.hungry("Nomsbat", 12) // Invokes the hungry method
```

__单引号是……？譬如 name'__ 您可以在参数和局部变量的名字中使用单引号。在数学上她被成为 _prime_，用来表示另一个。这只是为了方便。

构造函数需要设置对象中的每一个域。如果不这样做，编译器会报错。因为 Pony 中没有 `null`，我们不能像 Java， C# 和其他语言那样在构造函数执行前将 `null` 或零赋给每一个域。由于我们不想随机的崩溃，所以我们不能让域未定义(不像 C 和 C++)

若所有构造函数以相同的方式设置字段，可以简单地这样做

```pony
class Wombat
  let name: String
  var _hunger_level: U64
  var _thirst_level: U64 = 1

  new create(name': String) =>
    name = name'
    _hunger_level = 0

  new hungry(name': String, hunger': U64) =>
    name = name'
    _hunger_level = hunger'
```

现在不管哪个构造函数被调用，每个 `Wombat` 都有点口渴。

### 函数

Pony 中的函数就像是 Java，C＃，C ++，Ruby，Python 或者几乎任何面向对象语言中的方法(method)。她们使用 `fun` 关键字引入。可以像构造函数那样有参数，也可以有返回类型(如果没有给返回类型，则默认是 `None`)。

```pony
class Wombat
  let name: String
  var _hunger_level: U64
  var _thirst_level: U64 = 1

  new create(name': String) =>
    name = name'
    _hunger_level = 0

  new hungry(name': String, hunger': U64) =>
    name = name'
    _hunger_level = hunger'

  fun hunger(): U64 => _hunger_level

  fun ref set_hunger(to: U64 = 0): U64 => _hunger_level = to
```

第一个函数 `hunger` 简单明了。她的返回类型是 `U64`，并返回 `_hunger_level`，她是 `U64` 类型的。唯一有点不同的是没有 `return` 关键字。这是因为函数的返回值即是函数中最后一个表达式的返回值，上例中是 `_hunger_level` 的值。

__Pony 中有 `return` 关键字么？__ 是的，她用于在函数中提前返回某个值。

第二个函数 `set_hunger` 引入了一堆新的概念，让我们来逐一分析。

* `fun` 后的 `ref` 关键字

This is a __reference capability__. In this case, it means the _receiver_, i.e. the object on which the `set_hunger` function is being called, has to be a `ref` type. A `ref` type is a __reference type__, meaning that the object is __mutable__. We need this because we are writing a new value to the `_hunger_level` field.

__What's the receiver reference capability of the `hunger` method?__ The default receiver reference capability if none is specified is `box`, which means "I need to be able to read from this, but I won't write to it".

__What would happen if we left the `ref` keyword off the `set_hunger` method?__ The compiler would give you an error. It would see you were trying to modify a field and complain about it.

* 参数 0 后面的 `= 0`

这是 __默认参数__。她意味着您在调用时没有包含该参数，则会获得默认参数。上例中，`to` 会是 0，如果您不指定。

* 函数返回了什么?

她返回了 `_hunger_level` 的 _旧值_

__等等，认真的吗？ _旧_ 值？__ 没错。在 Pony 中，赋值是一个表达式而不是语句。这就意味着她有返回值。这点对于多数语言都成立，但她们返回 _新_ 值。换句话说，
给定 `a = b`，在大多数语言中，她的值是 `b` 的值，、。但在 Pony 中，她的值是 `a` 的 _旧_ 值。

__...为什么？__ 这被称为 "destructive read"，结合 capabilities-secure 类型系统能够让您做一些有用的事情。我们稍后会讲。现在告诉您，您可以用她来实现交换(swap)操作。在多数语言中，交换 `a` 和 `b` 的值，您需要这样做：

```pony
var temp = a
a = b
b = temp
```

在 Pony 中，你这样做便可：

```pony
a = b = a
```

### Finalisers

finaliser 是特殊的函数。她们必须被命名为 `_final`，take no parameters and have a receiver reference capability of `box`.换句话说， finaliser 必须被这么定义 `fun _final()`。

对象的 finaliser 在被 GC 回收前调用。函数在对象 finalisation 仍可被调用，但只能从其他 finaliser 内。finaliser 中不能发送消息。

finaliser 常被用于清理 C 代码分配的资源，像是文件描述符，网络套接字等。

## 那么继承呢？

在一些面向对象的语言中，一个类型可以从另一个类型 _继承(inherit)_，就像 Java 中的某些东西可以 __扩展(extend)__ 另外一些东西。Pony 不这样做。相反，Pony 比起 _继承(inheritance)_ 更喜欢 _组合(composition)_。换言之，利用 __is a__ 的关系而不是 __has a__ 的关系来实现代码重用。

另一方面，Pony 具有强大的 __trait__ 系统(类似于 Java 8 接口的默认方法)和强大的 __接口(interface)__ 系统(和 Go 语言的接口相似，structurally typed)

以后我们还会讨论。

## 命名规范

目前为止，告诉您 Pony 源代码是使用 __ASCII__ 写的，您应该不会感到惊讶。[ASCII](https://en.wikipedia.org/wiki/ASCII) 是一种标准文本编码用于英文和符号，几乎所有的编程语言的源代码都是其子集。

无论是 class, actor, trait, interface, primitive, 还是类型别名，都必须以大写字母开头。下划线开头的是私有或者特殊 _方法_(behaviors, constructors, and functions)，任何的方法或变量，参数，域都必须以小写字母开头。所有情况下，连续的下划线或者名称结尾处出现下划线都是不被允许的。使用下划线连接字母和数字是合法的。

事实上，数字可能会使用单独的下划线作为分隔符！但只有有效的变量名可以以 prime 结束。
