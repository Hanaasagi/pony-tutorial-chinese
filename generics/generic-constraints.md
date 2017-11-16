# 约束(Constraints)

## Capability Constraints

泛型类或者方法的类型参数约束可以归约为一个特别的 capability，就像之前所见：

```pony
class Foo[A: Any val]
```

没有约束，泛型必须能在所有 capabilities 下工作。有时你不想要被限制在一个特定的 capability 下，但同时你也无法支持所有的 capabilities。解决的方法是泛型约束修饰符(generic constraint qualifiers)。这些用来表示泛型可以接受的 capability。合法的修饰符如下：


| &#x25B7;        | Capabilities allowed         | Description
|-----------------|------------------------------|-------------
| #read           | ref, val, box                | Anything you can read from
| #send           | iso, val, tag                | Anything you can send to an actor
| #share          | val, tag                     | Anything you can send to more than one actor
| #any            | iso, trn, ref, val, box, tag | Default of a constraint
| #alias          | ref, val, box, tag           | Set of capabilities that alias as themselves (used by compiler)


上一节中，我们花了额外的功夫来支持 `iso`。如果不需要支持 `iso`，我们可以使用 `#read`，其可以支持 `ref`，`val` 和 `box`：

```pony
class Foo[A: Any #read]
  var _c: A

  new create(c: A) =>
    _c = c

  fun ref get(): A => _c

  fun ref set(c: A) => _c = c

actor Main
  new create(env:Env) =>
    let a = Foo[String ref](recover ref "hello".clone() end)
    env.out.print(a.get().string())

    let b = Foo[String val]("World")
    env.out.print(b.get().string())
```
