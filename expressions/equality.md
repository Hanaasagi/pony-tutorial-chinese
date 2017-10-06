# Equality in Pony

Pony 的相等有两种：结构(structure)和标识(identity)。/*值相等和id相等*/

## Identity equality

使用 `is` 关键字来检查标识是否相等。`is` 验证两个对象是否为同一个。

```pony
if None is None then
  // TRUE!
  // There is only 1 None so the identity is the same
end

let a = Foo("hi")
let b = Foo("hi")

if a is b then
  // NOPE. THIS IS FALSE
end

let c = a
if a is c then
  // YUP! TRUE!
end
```

## Structural equality

使用中缀运算符 `==` 来检查结构是否相等。它验证两个对象是否有相同的值。如果两个对象的标识相等，那么按照定义来说，他们有相同的值。

你可以通过实现 `fun eq(that: box->Foo): Bool` 方法来定义如何在你的对象上检查结构相等。记住，因为 `==` 是一个中缀运算符，`eq` 必须在左操作数上定义，右操作数必须是 `Foo` 类型的。

```pony
class Foo
  let _a: String

  new create(a: String) =>
    _a = a

  fun eq(that: box->Foo): Bool =>
    this._a == that._a

actor Main
  new create(e: Env) =>
    let a = Foo("hi")
    let b = Foo("bye")
    let c = Foo("hi")

    if a == b then
      // won't print
      e.out.print("1")
    end

    if a == c then
      // will print
      e.out.print("2")
    end

    if a is c then
      // won't print
      e.out.print("3")
    end
```

如果你没有定义你自己的 `eq`，那么你将继承默认的实现——值相等即标识相等。

```pony
interface Equatable[A: Equatable[A] #read]
  fun eq(that: box->A): Bool => this is that
  fun ne(that: box->A): Bool => not eq(that)
```

## Primitives and equality

你可能记得[第二章](http://tutorial.ponylang.org/types/primitives.html)里说过 primitive 除了下面两点外和类相同：

* primitive 没有 field
* 自定义的 primitive 只能存在一个实例

这意味着，某类型的每一 primitive 都是结构相等并且标识相等的。比如 None 总是 None：

```pony
if None is None then
  // this is always true
end

if None == None then
  // this is also always true
end
```
