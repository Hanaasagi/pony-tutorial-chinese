# Testing with PonyTest

PonyTest 是 Pony 的单元测试框架。它在编写测试用例和运行测试上设计得尽可能简单。

每个单元测试是一个类，有一个测试函数。默认情况下，所有的测试并发执行。

会为每个测试的运行提供一个辅助对象。它提供了日志记录和断言函数。默认情况下，日志信息只在测试失败时显示。

任何断言失败时，整个测试都会被认为失败。然而，也可以在测试函数中通过抛出异常来标记失败。

## Example program

想使用 PonyTest 只需要简单地为每个测试写一个 `TestList` 类型的类，告诉 PonyTest 对象测试的信息。通常 `TestList` 会是 package 的 `Main`。

下面是一个完整的程序，包含两个测试。

```pony
use "ponytest"

actor Main is TestList
  new create(env: Env) =>
    PonyTest(env, this)

  new make() =>
    None

  fun tag tests(test: PonyTest) =>
    test(_TestAdd)
    test(_TestSub)

class iso _TestAdd is UnitTest
  fun name(): String => "addition"

  fun apply(h: TestHelper) =>
    h.assert_eq[U32](4, 2 + 2)

class iso _TestSub is UnitTest
  fun name(): String => "subtraction"

  fun apply(h: TestHelper) =>
    h.assert_eq[U32](2, 4 - 2)
```

上例中的 `make()` 构造函数是不需要的。不过，它允许去做一些测试的简单聚合。所以建议所有测试的 `Main` 都提供它。

`Main.create()` 只在当前 package 的程序调用时调用。`Main.make()` 在聚合时被调用。如果需要，可以将额外的代码添加到这些构造函数中来执行额外的任务。

## Test names

测试通过名称来标识，用在测试结果输出和命令行中指定具体要运行的测试。这些名称独立于 Pony 源码中测试类的名称。

名称可以使用任意的字符串，但是对于大型项目，强烈建议使用分级命名方案来方便选择测试。

## Aggregation

有时需要运行一组来自多个不同文件的单元测试。例如，如果几个 package 都有属于自己的单元测试，那么一起运行所有的测试是很有用处的。

这可以通过编写一个会调用列出的每个函数的聚合测试列表来实现。下面是一个聚合 package `foo` 和 `bar` 中测试的示例：

```pony
use "ponytest"
use foo = "foo"
use bar = "bar"

actor Main is TestList
  new create(env: Env) =>
    PonyTest(env, this)

  new make() =>
    None

  fun tag tests(test: PonyTest) =>
    foo.Main.make().tests(test)
    bar.Main.make().tests(test)
```

聚合测试类本身可以被聚合。每个测试列表类可以包含它自己的测试和聚合列表的任意组合。

## Long tests

简单的测试可以运行在单个函数中。当该函数退出时，无论是返回还是抛出异常，测试都会完成。这对需要使用 actors 的测试是不可行的。

Long tests 允许延时完成。任何测试可以在 `TestHelper` 上调用 `long_test()` 来指示它需要保持运行。当测试最终完成时，它调用 `TestHelper` 的 `complete()`。

`complete()` 函数接收一个 `Bool` 参数用来指示测试是否成功。如果有任何断言失败，测试会被认为失败，不管此参数的值。然而 `complete()` 仍需被调用。

由于失败的测试可能会被挂起，所以每个 long test 必须设定 `timeout`。当测试函数退出时，定时器以指定的 `timeout` 启动。如果这个定时器在 `complete()` 调用前出发，测试会被标记为失败，并且报告超时。

超时时，单元测试对象上的 `timed_out()` 函数会被调用。这应当执行任何指定的 tidy up，以允许程序退出。如果发生超时不需要调用 `complete()`。

请注意 timeout 只在测试挂起并且要阻止程序结束时有用。在一个不应当挂起的测试上设置一个非常长的 timeout 是完全可以接受的，如果成功的话，不会让测试延迟。

超时不应当被用作检测测试是否失败的标准方法。

## Exclusion groups

默认情况下，所有的测试会并发执行。对于某些测试可能会有问题，比如，如果他们操作外部文件或者使用系统资源。要解决这个问题需要将测试放在 exclusion group 中。

同一个 exclusion group 中的测试不会并发执行。

exclusion group 按名称区别，可以使用任意的字符串。可以使用多个 exclusion group，并且不同 group 中的测试可以并发执行。没有放在 exclusion group 中测试会与其他测试并发执行。

命令行选项 `--sequential` 可以阻止测试并发执行，无论是否有 exclusion groups。这是为了调试而不是正常使用。

## Tear down

每个单元测试对象可以定义 `tear_down()` 函数。这会在测试结束后调用，用来重置复杂的环境。

不管通过还是失败，每个测试都会调用 `tear_down()` 函数。如果测试超时，`tear_down()` 会在 `timed_out()` 之后调用。

当一个测试位于 exclusion group 时，`tear_down()` 会被认为测试的一部分。在当前测试的 `tear_down()` 返回之后，exclusion group 的下一个测试才会开始。

测试的 `TestHelper` 会被传递至 `tear_down()` 中，并允许消息记录和调用断言函数。

## Additional resources

You can learn more about PonyTest specifics by checking out the [API documentation](http://stdlib.ponylang.org/ponytest--index/). There's also a [testing section](http://patterns.ponylang.org/testing/index.html) in the [Pony Patterns](http://patterns.ponylang.org/) book.
