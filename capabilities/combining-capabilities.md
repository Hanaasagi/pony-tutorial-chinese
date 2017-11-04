# Combining Capabilities

当一个对象的 field 被读取时，它的 reference capability 同时依赖于 field 自身的 reference capability 和 __origin__ 的 reference capability，__origin__ 是被读取字段所在的对象。

这是因为 __origin__ reference capability 所做的所有保证都必须保留在其字段当中。

# Viewpoint adaptation

将 origin 和 field 的 capabilities 进行组合的过程称为 __viewpoint adaptation__。也就是说，__origin__ 有 __viewpoint__，并且只能从那个 __viewpoint__ “观察”它的 field。

下表展现了对于不同 capability 的 __origin__ 来说，每种 capability 的 __field__ 会呈现出什么姿态。

---

| &#x25B7;        | iso field | trn field | ref field | val field | box field | tag field |
|-----------------|-----------|-----------|-----------|-----------|-----------|-----------|
| __iso origin__  | iso       | tag       | tag       | val       | tag       | tag       |
| __trn origin__  | iso       | trn       | box       | val       | box       | tag       |
| __ref origin__  | iso       | trn       | ref       | val       | box       | tag       |
| __val origin__  | val       | val       | val       | val       | val       | tag       |
| __box origin__  | tag       | box       | box       | val       | box       | tag       |
| __tag origin__  | n/a       | n/a       | n/a       | n/a       | n/a       | n/a       |

---

举个例子，如果你有一个 `trn` 的 origin 然后你读取一个 `ref` 的 field，你会得到一个 `box` 的结果：

```pony
class Foo
  var x: String ref

class Bar
  fun f() =>
    var y: Foo trn = get_foo_trn()
    var z: String box = y.x
```

## Explaining why

可能不是现在，但是最终你对觉得这张表非常自然。为了让你更加熟悉，我们会遍历表中的每个格子，然后解释为什么要这样做。

### Reading from an `iso` variable

任何从 `iso` origin 读取的东西必须保持 origin 所拥有的 isolation guarantee。关键是要记住 `iso` 可以发送至另一个 actor，也可以成为其他的 reference capability。所以当我们读取一个 field 时，我们需要得到一个不会破坏 `iso` 保证的结果，也就是说不能破坏 __读写唯一(read and write uniqueness)__。

`iso` field 可以做出和 `iso` origin 相同的保证，所以没问题。`val` field 是 _全局不可变的(globally immutable)_，这意味着无论 origin 是什么(除了`tag`)，它总是可以被读取。这也没问题。

其他的都会打破 `iso` 保证。这就是为什么其他的 reference capabilities 被视为 `tag`：它是唯一一个既不可读也不可写的。

### Reading from a `trn` variable

就像 `iso` 一样，但有着更弱一些的保证(_写唯一(write uniqueness_)而不是 _读写唯一(read and write uniqueness)_)。这是一个
很大的区别，因为我们现在可以返回一些可读的东西。

`iso` field 做出了比 `trn` origin 更强的保证，`trn` field 做出了相同的保证，所以他们都没问题。`val` field 是 _全局不可变的(globally immutable)_，所以也没问题。`box` field 是可读的，而我们只需要保证 _写唯一(write uniqueness_，所以也没有问题。

对于允许写操作的 `ref` field，我们返回 `box` 作为替代。

### Reading from a `ref` variable

`ref` origin 不会对 field 进行任何修改。这是因为 `ref` origin 没有做出和其 field 有任何冲突的保证。

### Reading from a `val` variable

`val` origin 是全局不可变的(globally immutable)，所以它的所有 field 也是 `val` 的。唯一的例外是 `tag` field。因为我们不能读取，我们也不能保证没有人可以写入，所以它依然是 `tag`。

### Reading from a `box` variable

`box` 变量是局部不可变的。这意味着它可能会通过其他变量(`trn`或`ref`)成为可变的，但是 `box` 变量也可能是一些 `val` 变量的别名。

当我们读取一个 field 时，我们需要返回一个与 field 兼容的 reference capability，但需要是局部不可变的。

`iso` field 会作为 `tag` 返回，因为没有局部不可变的 reference capability 可以维持 `iso` 保证。`val` field 会作为 `val` 返回，因为全局不可变强于局部不可变。`box` field 和 origin 做出了相同的保证，所以没有问题。

对于 `trn` 和 `ref` 我们需要返回一个局部不可变且不会违反 field 自身的保证的 reference capability。在这两种情况下，我们都返回 `box`。

### Reading from a `tag` variable

这个很简单：`tag` 变量是不透明的(opaque)！他们不能被读取。
