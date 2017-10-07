# 语法糖(Sugar)

Pony 允许你在代码中去省略一些小细节，它会进行还原。这样做是为了让你的代码变得整洁更加易读。使用语法糖是可选的，如果你愿意你可以写完整的版本。

## Apply

Pony 的许多类都有一个叫做 `apply` 的函数，用来执行那个类型最常见的操作。Pony 允许省略 `apply`，直接从对象上进行调用。

```pony
var foo = Foo.create()
foo()
```

becomes:

```pony
var foo = Foo.create()
foo.apply()
```

任何所需的参数可以像方法调用那样添加。

```pony
var foo = Foo.create()
foo(x, 37 where crash = false)
```

becomes:

```pony
var foo = Foo.create()
foo.apply(x, 37 where crash = false)
```

__我仍需为 apply 提供参数么？__ 是的，Pony 只会帮你添加所省略的 `apply`。必须提供数量正确，类型正确的参数。可以正常使用默认参数和命名参数。

__如果我调用一个名为 foo 的函数会怎么样？__ `apply` 语法糖只会在调用一个对象时生效，而不是调用一个方法。编译器了解这些区别，只会在恰当的时候添加 `apply`。

## Create

为了创建一个对象你需要指定类型并调用构造函数。Pony 允许你省略构造函数并会自动帮你调用 `create()`

```pony
var foo = Foo
```

becomes:

```pony
var foo = Foo.create()
```

通常类型出现在表达式中不是一件合法的事情，所以省略构造函数不会造成歧义。记住，你可以轻松的认识到某个名字是一个类型因为他们以大写字母开头。

如果 `create` 需要参数，则可以像调用类型一样提供参数。可以正常使用默认参数和命名参数。

```pony
var foo = Foo(x, 37 where crash = false)
```

becomes:

```pony
var foo = Foo.create(x, 37 where crash = false)
```

__如果我想使用不叫 create 的构造函数呢？__ 很遗憾这个语法糖不能帮你这样做，你需要在代码里写明。

__如果我想调用的 create 没有参数，我可以留着圆括号么？__ 不。`Type()` 形式的调用是 combined create-apply 语法糖。要 `Type.create()` 请使用 `Type`。

## Combined create-apply

如果一个类型有一个没有参数的 create 构造函数，那么 create 和 apply 语法糖可以一起使用。调用此类型会调用 create 构造函数并且会添加 apply。create 构造函数不会接收任何参数，apply 会接收你提供的所有参数。

```pony
var foo = Foo()
var bar = Bar(x, 37 where crash = false)
```

becomes:

```pony
var foo = Foo.create().apply()
var bar = Bar.create().apply(x, 37 where crash = false)
```

__如果 create 有默认参数会怎样？combined create-apply 语法糖会使用此默认值么？__ combined create-apply 语法糖只能用于 `create` 构造函数没有参数的时候。如果有默认参数，那么此语法糖不能使用。

## Update

`update` 语法糖允许任何类使用赋值的方式去接收值。许多语言允许在集合上使用，比如 C 的数组 `a[3] = x;`。

任何左侧为函数调用的赋值，Pony 都会将此转换成 update，右侧的值作为参数传入：

```pony
foo(37) = x
```

becomes:

```pony
foo.update(37 where value = x)
```

赋值右侧的值总是会当做名为 `value` 的参数传入。通过提供恰当的带有 `value` 参数的 `update` 函数，任何对象都可以用这种语法。

__update 函数唯一的参数必须是整数么？__ 不是，你可以通过定义来 update 任何你想要的值，只要有一个参数是 `value`。下面这些都可以：

```pony
foo1(2, 3) = x
foo2() = x
foo3(37, "Hello", 3.5 where a = 2, b = 3) = x
```

__Does it matter where `value` appears in my parameter list?__ Whilst it doesn't strictly matter it is good practice to put `value` as the last parameter. That way all of the others can be specified by position.

__`value` 在参数中出现的位置是特定的么？__ 尽管没有严格的要求，但是将 `value` 作为最后一个参数是很好的做法。这样所有其他的参数可以通过位置来指定。
