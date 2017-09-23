# Type Expressions

目前为止，我们已讨论过的类型可以在 __type expressions__ 中进行组合。如果你之前使用的是面向对象编程，那么这些你可能没有见过，但他们在函数式编程中十分常见。__type expression__ 也被称为 __algebraic data type__。

有三种 type expression： __tuples__，__unions__ 和 __intersections__。

## Tuples

__tuple__ 类型是一系列的类型。举个例子，如果我们想要表达某个东西是一个 `String` 后跟着 `U64` 的，我们可以这样写：

```pony
var x: (String, U64)
x = ("hi", 3)
x = ("bye", 7)
```

所有的 type expressions 都要被圆括号包围，并且元素间使用逗号分隔。可以通过赋值来重构 tuple：

```pony
(var y, var z) = x
```

或者我们可以直接访问 tuple 中的元素：

```pony
var y = x._1
var z = x._2
```

记住，没有办法去给 tuple 的某个元素赋值。作为替代的是可以重新赋值整个 tuple，像这样：

```pony
x = ("wombat", x._2)
```

__为什么使用 tuple 而不是类？__ tuple 是表示不具有任何相关代码或预期行为的值集合的一种方式。基本上，如果你只需要快速地构造一个集合，例如想从函数中返回多个值，可以使用 tuple。

## Unions

__union__ 类型和 __tuple__ 写法相似，但在元素间使用 `|` (读作 "or") 替代 `,`。tuple 表示一个值集合， union 表示 _一个_ 值，可以为任何指定的类型。

union 可以用于表达其他语言中出现的多种概念，比如 optional value, 枚举， marker values 等。

```pony
var x: (String | None)
```

上面的例子使用 union 来表示一个可选的类型， `x` 可以是 `String`，也可以是 `None`。

## Intersections

__intersection__ 的元素间使用 `&` (读作 "and")。她和 union 表示的正相反：她是 _一个_ 值同时满足所有的指定类型。

这对于组合 traits 和 interfaces 十分有用，比如这个来自于标准库的例子：

```pony
type Map[K: (Hashable box & Comparable[K] box), V] is HashMap[K, V, HashEq[K]]
```

这是一个相当复杂的类型别名，但是我们来看看 `K` 的约束(constraint)。她是 `(Hashable box & Comparable[K] box)`，这表示 `K` 是 `Hashable` _且_ `Comparable[K]` 的。

## Combining type expressions

type expressions 可以组合成更复杂的类型。这是标准库的一个例子：

```pony
var _array: Array[((K, V) | _MapEmpty | _MapDeleted)]
```

这里我们有一个 array，每一个元素是 `(K, V)` 或者 `_MapEmpty` 或者 `_MapDeleted`。

因为每一个 type expression 都有圆括号，所以当你掌握窍门后他们实际上阅读起来很容易。但如果你经常使用复杂的 type expression，那么为它提供一个类型别名是很好的做法。

```pony
type Number is (Signed | Unsigned | Float)

type Signed is (I8 | I16 | I32 | I64 | I128)

type Unsigned is (U8 | U16 | U32 | U64 | U128)

type Float is (F32 | F64)
```

这些都是标准库用到的类型别名。

__`Number` 是一个包含其他类型别名的 type expression 的类型别名么？__ 是的。
