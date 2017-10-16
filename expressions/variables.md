# 变量(Variables)

像大多数的编程语言一样，Pony 允许你将数据存储在变量中。有几种拥有不同生命周期和不同用途的变量

## 局部变量(Local variables)

Pony 中的局部变量(local variables)工作方式和其他语言相同，允许你存储在计算时产生的临时值。局部变量的生命期是一个代码块，并且每次执行代码块时都会创建，执行完成后销毁。

要定义一个局部变量需要使用 `var` 关键字(也可以使用 `let`，不过这将是之后介绍的内容)。`var` 后面紧跟着变量名，然后可以选择使用 `:` 说明其类型。就像这个样子：

```pony
var x: String = "Hello"
```

上例中我们将 __字符串字面量__ `Hello` 赋值给 `x`。

定义变量时可以不必赋值：你可以在你想要的时候再赋值，如果你想要在赋值前读取变量的值，编译器将会警告而不是让一个可怕的 _uninitialised variable__ bug 从眼皮下溜走。

每个变量都有其类型，但是如果你可以提供一个初始值的话，便可以不在声明中具体指定。编译器会自动使用变量初始值的类型。

下面的 `x`，`y`，`z` 的定义是等价的。

```pony
var x: String = "Hello"

var y = "Hello"

var z: String
z = "Hello"
```

__我可以同时省略变量的类型和初始值么？__ 不能。编译器警告：无法找出该变量的类型。

所有局部变量名均以小写字母开头。当你想要一个与第一个变量几乎相同意义的第二个变量时，你可以以 _prime_ `'` 作为结尾(或者不止一个)，这非常有用。举个例子，你可以有一个变量叫做 `time`，一个变量叫做 `time'`。

变量所生存的代码块被称为 __作用域(scope)__。确切的说作用域依赖于它被定义的位置。比如，在 `if` 语句的 `then` 表达式中定义的变量的作用域就是 `then` 表达式。我们还没有看过`if`语句，但是它们与其他语言非常相似。

```pony
if a > b then
  var x = "a is bigger"
  env.out.print(x)  // OK
end

env.out.print(x)  // Illegal
```

变量只会存在于定义开始处至当前作用域结束处。对于变量 `x`，结束是在 then 表达式的结束处：在这之后便不能使用了。

## Var vs. let

`var` 或者 `let` 都可以声明局部变量。使用 `var` 意味着变量可以被重复赋值多次。使用 `let` 意味着变量只能被赋值一次。

```pony
var x: U32 = 3
let y: U32 = 4
x = 5  // OK
y = 6  // Error, y is let
```

你不必非要使用 `let` 来声明变量，但如果你知道某个变量的值将永远不会改变，那么使用`let`将会是一个很好的方式来捕捉错误。它也可以作为一种有用的注释，表明值不会改变。

## Fields

在 Pony 中，field 是和所处对象生存期相同的变量。他们和其他面向对象语言的 field 相似。他们始于对象的构造函数并随对象一起被销毁。

如果 field 的名字以 `_` 开头，它是 __私有的(private)__。这说明只有该字段所在的类型能够对 field 进行读写。否则，该字段是 __公有的(public)__ 能够在任何地方读写。

就像局部变量一样，field 也可以是 `var` 或者 `let` 的。他们也能在定义时赋一个初始值，或者可以在构造函数中给出它们的初始值。

__field 可以在构造函数后出现么？__ 不能。为了使 Pony 的语法不产生歧义，只允许类型别名可以出现在 `actor Name`, `object is Trait`等和 field 的定义之间。在任何情况下，将变量放到程序员容易看到的地方是一种良好的习惯，因为对于此类型的任何方法 field 都是可以访问的。

Unlike local variables, some types of fields can be declared using `embed`. Specifically, only classes or structs can be embedded - interfaces, traits, primitives and numeric types cannot. A field declared using `embed` is similar to one declared using `let`, but at the implementation level, the memory for the embedded class is laid out directly within the outer class. Contrast this with `let` or `var`, where the implementation uses pointers to reference the field class. Embedded fields can be passed to other functions in exactly the same way as `let` or `var` fields. Embedded fields must be initialised from a constructor expression.

与局部变量不同，某些类型的 field 可以使用 `embed` 声明。具体来说，只有类或者结构体可以是嵌入的，接口，traits，primitives 和数值类型则不能。使用 `embed` 声明的字段与使用 `let` 声明的字段相似。但在实现上，根据内存布局来说，嵌入类是直接位于外部类中的。与使用指针去引用 field class 的 `let` 或 `var` 不同。嵌入 field 可以被传入其他函数。嵌入 field 必须在构造函数中进行初始化。/*待定*/

__为什么使用 `embed`？__ 当访问一个 field 和创建 field 分配内存时，`embed` 能避免指针间接寻址。默认情况下，建议尽可能使用 `embed`。然而由于嵌入字段与其父对象一起分配，因此若外部持有该字段的引用将阻止父对象的垃圾回收，可能导致更高的内存占用。如果你对此非常关心，请使用 `let`

## 全局(Globals)

有些编程语言具有允许从任何地方访问的 __全局变量(global variables)__。这是一个坏主意！Pony 没有全局变量。

## 隐藏(Shadowing)

一些编程语言允许你声明一个和已存在的某个变量名称相同的变量，并有一套访问规则。这被称为 _隐藏(shadowing)_，它是一些 bug 的源头。如果你意外的重复声明了一个变量，Pony 编译器会发出警告。

如果你想要一个相近的变量名，你可以使用 prime `'`。
