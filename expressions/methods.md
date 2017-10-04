# 方法(Methods)

所有描述行为而不是定义类型的 Pony 代码都需要放在被称为方法的命名快中。有三种类型的方法：函数，构造函数，和行为(behaviour)。所有的方法都关联着类型定义(如 类)。没有全局函数。

行为用于处理发送至 actor 的异步消息。我们之后会提到。

__我可以像 Python 那样在方法外写一些代码么？__ 不。所有的 Pony 代码必须在方法中。

## 函数(Functions)

Pony 的函数有点像其他语言的函数(或方法)。他们可以有 0 个或更多形参和 0 或者 1 个返回值。如果返回类型是省略的，那么函数会返回 `None`

```pony
class C
  fun add(x: U32, y: U32): U32 =>
    x + y

  fun nop() =>
    add(1, 2)  // Pointless, we ignore the result
```

函数的形参在函数名后的圆括号中指定。没有形参的函数也需要有圆括号。

需为每一个形参指明形参名和类型。在上例中函数 `add` 有两个形参，`x` 和 `y`，两者的类型都是 `U32`。调用函数时传入的值(上例中是 `1` 和 `2`)被称为实参。函数调用时他们会被求值并赋值给形参。形参不能在函数内再次被赋值，他们相当于使用 `let` 声明。

形参后跟着返回类型。如果没有则可以省略。

返回值后面有一个 `=>` 然后是函数体。返回值就是函数体最后一条命令的值(记住一切皆为表达式)。

如果你想提前退出一个函数可以使用 `return`。如果函数有返回类型，那么你需要提供一个返回值。如果函数没有返回类型，那么 `return` 应当不跟任何值。

Pony 拥有尾递归优化，所以可以这样实现阶乘函数：

```pony
fun factorial(x: I32): I32 ? =>
  if x < 0 then error end
  if x == 0 then
    1
  else
    x * factorial(x - 1)?
  end
```

能否进行优化取决于 LLVM 编译器的版本。

__我能通过参数类型来重载函数么？__ [Case functions](http://tutorial.ponylang.org/pattern-matching/case-functions.html) 提供了一种机制去根据参数类型选择实现不同的同名函数。

## 构造函数(Constructors)

如同许多语言，Pony 的构造函数用于初始化新创建的对象。然而与很多语言不同，Pony 的构造函数是有名的，所以你可以有任意数量，采用你想要形参的构造函数。按照惯例，每个类型的主构造函数被称为 `create`。

```pony
class Foo
  var _x: U32

  new create() =>
    _x = 0

  new from_int(x: U32) =>
    _x = x
```

构造函数的目的是设置正在创建的对象的内部状态。为了确保这一点，构造函数必须初始化对象的所有 field。

__我可以提前退出构造函数么？__ 可以。使用没有值的 `return`。这个对象不许处于合法的状态。

## Calling

与其他语言一样，Pony 通过在方法名后的圆括号中提供参数的方式来调用方法。即使没有传递参数，也需要圆括号。

```pony
class Foo
  fun hello(name: String): String =>
    "hello " + name

  fun f() =>
    let a = hello("Fred")
```

构造函数通常从一个类型上调用，这个类型是要创建的类型。要做到这一点，只需指定一个类型然后跟一个点，接着是要调用的构造函数名称。

```pony
class Foo
  var _x: U32

  new create() =>
    _x = 0

  new from_int(x: U32) =>
    _x = x

class Bar
  fun f() =>
    var a: Foo = Foo.create()
    var b: Foo = Foo.from_int(3)
```

函数总是在对象上调用。需要指定对象，然后跟一个点，接着跟要调用的函数名称。如果对象被省略则使用当前对象(`this`)。

```pony
class Foo
  var _x: U32

  new create() =>
    _x = 0

  new from_int(x: U32) =>
    _x = x

  fun get(): U32 =>
    _x

class Bar
  fun f() =>
    var a: Foo = Foo.from_int(3)
    var b: U32 = a.get()
    var c: U32 = g(b)

  fun g(x: U32): U32 =>
    x + 1

```

也可以在表达式上调用构造函数。这里创建一个与指定表达式相同类型的对象 - 这相当于直接指定类型。

```pony
class Foo
  var _x: U32

  new create() =>
    _x = 0

  new from_int(x: U32) =>
    _x = x

class Bar
  fun f() =>
    var a: Foo = Foo.create()
    var b: Foo = a.from_int(3)
```

## 默认参数(Default arguments)

定义方法时你可以为任意参数提供默认值。调用者可以选择去使用你提供的默认值还是使用自己的值。默认参数的值在参数名后使用 `=` 指定。

```pony
class Coord
  var _x: U32
  var _y: U32

  new create(x: U32 = 0, y: U32 = 0) =>
    _x = x
    _y = y

class Bar
  fun f() =>
    var a: Coord = Coord.create()     // Contains (0, 0)
    var b: Coord = Coord.create(3)    // Contains (3, 0)
    var b: Coord = Coord.create(3, 4) // Contains (3, 4)
```

__我必须为所有的参数提供默认值么？__ 不，根据您的需要来定。

## 命名参数(Named arguments)

到目前为止，当调用方法时我们需要按顺序给出所有的参数。这种被称为 __位置(positional)__ 参数。但是你还可以通过指定名称来以任意的顺序传入参数。这种被称为 __命名(named)__ 参数。

要使用命名参数调用方法，要使用 `where `关键字，后跟命名参数及其值。

```pony
class Coord
  var _x: U32
  var _y: U32

  new create(x: U32 = 0, y: U32 = 0) =>
    _x = x
    _y = y

class Bar
  fun f() =>
    var a: Coord = Coord.create(where y = 4, x = 3)
```

__我应该为每个命名参数指定 `where`？__ 不，每个方法调用中只能有一个 `where`。

命名参数和位置参数可以在调用时一起使用。先指定你想要的位置参数，然后使用 `where`，最后是命名参数。要小心，每个参数只能指定一次。

默认参数也可以与位置和命名参数组合使用 - 只需要跳过任何要使用默认参数的参数。

```pony
class Foo
  fun f(a:U32 = 1, b: U32 = 2, c: U32 = 3, d: U32 = 4, e: U32 = 5): U32  =>
    0

  fun g() =>
    f(6, 7 where d = 8)
    // Equivalent to:
    f(6, 7, 3, 8, 5)
```

__我可以在使用位置参数时跳过第一个么？__ 不行。如果使用位置参数，它们必须是调用中的第一个参数。

## 链式调用(Chaining)

链式方法调用允许你不需要在方法中返回一个接收者就可以在对象上进行链式调用。调用方法并链接接收者的语法是 `object.>method()`，大致相当于 `(object.method() ; object)`。链式调用方法会丢弃其正常的返回值。/*接受者(receiver) 指调用方法时点左边的东西*/

```pony
primitive Printer
  fun print_two_strings(out: StdStream, s1: String, s2: String) =>
    out.>print(s1).>print(s2)
    // Equivalent to:
    out.print(s1)
    out.print(s2)
    out
```

注意链式调用的最后一个 `.>` 可以是 `.` 如果你需要最后一次调用的返回值的话。

```pony
interface Factory
  fun add_option(o: Option)
  fun make_object(): Object

primitive Foo
  fun object_wrong(f: Factory, o1: Option, o2: Option): Object =>
    f.>add_option(o1).>add_option(o2).>make_object() // Error! The expression returns a Factory

  fun object_right(f: Factory, o1: Option, o2: Option): Object =>
    f.>add_option(o1).>add_option(o2).make_object() // Works. The expression returns an Object
```

## Privacy

在 Pony 中方法名以小写字母或者下划线后跟一个小写字母开头。以下划线起始的方法是私有的。这意味着他们只能被相同包(package)内的代码调用。不以下划线起始的方法是公有的，任何人都可以调用。

__方法名可以使用双下划线(或更多)开头么？__ 不可以。如果第一个字符是一个下划线，那么第二个字符必须为一个小写字母。
