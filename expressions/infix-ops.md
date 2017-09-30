# 中缀运算符(Infix Operators)

中缀运算符的运算符是以中缀形式处于操作数的中间。最常见的是算数和比较运算符:

```pony
1 + 2
a < b
```

Pony 具有与其他语言相同的中缀运算符。

## 优先级(Precedence)

在复杂的表达式中使用中缀运算符会遇到 __优先级(precedence)__ 问题，即哪个运算符会先执行。比如下面这个表达式：

```pony
1 + 2 * 3
```

如果我们先执行加法，结果将会是 9；如果我们先执行乘法，结果将会是 7。在数学中有着关于运算符执行顺序的规则，大多数编程语言也遵循着这套规则。

这样做的结果是程序员必须记住这些顺序，这并不是一件讨人喜欢的事情。大多数人会记得在加法前应先执行乘法，但左移和位与操作呢？有时人们会猜错导致错误。更糟糕的是这些 bug 通常很难发现。

Pony 采取了不同的方法，并取消了中缀优先级。使用到多个中缀运算符的表达式都必须使用圆括号来消除歧义。如果你不这样做，编译器会警告。

这意味着上面的例子是非法的，在 Pony 中应该向下面这样写：

```pony
1 + (2 * 3)
```

允许在表达式中重复使用同一个运算符而不必使用圆括号：

```pony
1 + 2 + 3
```

## 运算符别名(Operator aliasing)

Pony 的大多数中缀运算符实际上是函数的别名。左操作数是函数调用的接收者，右操作数会作为参数传入。例如，以下表达式是等价的：

```pony
x + y
x.add(y)
```

这意味着 `+` 不是只能应用于 magic types 的特殊符号。任何类型都可以提供自己的 `add` 函数实现。如果程序员想要的话，可以对此类型使用 `+`。

定义自己的 `add` 函数时，对参数或返回值的类型没有限制。`+` 的右侧必须匹配参数类型，而`+`表达式的返回值类型即是 `add` 的返回类型。

下面一个完整的例子：

```pony
// Define a suitable type
class Pair
  var _x: U32 = 0
  var _y: U32 = 0

  new create(x: U32, y: U32) =>
    _x = x
    _y = y

  // Define a + function
  fun add(other: Pair): Pair =>
    Pair(_x + other._x, _y + other._y)

// Now let's use it
class Foo
  fun foo() =>
    var x = Pair(1, 2)
    var y = Pair(3, 4)
    var z = x + y
```

[Case functions](http://tutorial.ponylang.org/pattern-matching/case-functions.html) 可以用来提供不止一个 `add` 函数。也可以使用 union 类型或者 f-bounded 多态来重载中缀运算符，但这超出了本教程的范围。更多信息，请参考 Pony 标准库。

中缀运算符及其对应函数如下表：

---

Operator | Method   | Description
---------|----------|------------
+        | add()    | Addition
-        | sub()    | Subtraction
*        | mul()    | Multiplication
/        | div()    | Division
%        | mod()    | Modulus
<<       | shl()    | Left bit shift
>>       | shr()    | Right bit shift
and      | op_and() | And, both bitwise and logical
or       | op_or()  | Or, both bitwise and logical
xor      | op_xor() | Xor, both bitwise and logical
==       | eq()     | Equality
!=       | ne()     | Non-equality
<        | lt()     | Less than
<=       | le()     | Less than or equal
>=       | ge()     | Greater than or equal
>        | gt()     | Greater than

---

## 短路(Short circuiting)

`and` 和 `or` 运算符具有 __短路(short circuiting)__ 特性。第一个操作数总是会被执行，但第二个的执行取决于它是否能影响结果整体。

对于 `and`，如果第一个操作数是 __false__，那么第二个操作数不会执行，因为它不能影响整体结果。

对于 `or`，如果第一个操作数是 __true__，那么第二个操作数不会执行，因为它不能影响整体结果。

This is a special feature built into the compiler, it cannot be used with operator aliasing for any other type.

## 一元运算符(Unary operators)

一元运算符以相同的方式处理，但是只有一个操作数。比如，下面的表达式是等价的：

```pony
-x
x.neg()
```

所有的一元运算符及其对应函数如下表：

---

Operator | Method   | Description
---------|----------|------------
-        | neg()    | Arithmetic negation
not      | op_not() | Not, both bitwise and logical

---
