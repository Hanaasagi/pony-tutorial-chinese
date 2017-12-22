# Linking to C Libraries

如果 Pony 代码调用 FFI 函数，那么这些函数或者包含他们的库必须链接到 Pony 程序中。

## Use for external libraries

要将一个外部的库链接到 Pony 代码中，需要使用另一种 `use` 明林。`lib` 指示符用于告诉编译器你想要链接一个库：

```pony
use "lib:foo"
```

像其他 `use` 命令一样可以指定条件。当库在不同平台有不同名字时这将非常有用。

这里有一个来自标准库的真实例子：

```pony
use "path:/usr/local/opt/libressl/lib" if osx
use "lib:ssl" if not windows
use "lib:crypto" if not windows
use "lib:libssl-32" if windows
use "lib:libcrypto-32" if windows

primitive _SSLInit
  """
  This initialises SSL when the program begins.
  """
  fun _init() =>
    @SSL_load_error_strings[None]()
    @SSL_library_init[I32]()
```

Windows 上，我们使用 `libssl-32` 和 `libcrypto-32` 库，其他平台上我们使用 `ssl` 和 `crypto`。他们包含了 FFI 函数 `SSL_library_init` 和 `SSL_load_error_strings`

默认情况下 Pony 编译器会在 standard places 查找并进行连接，但这是由 build 的平台定义的。有必要去在一个额外的地方查找。`use "path:..."` 命令允许你将指定的路径添加到搜索路径中。上面的例子在 OSX 平台上添加了 `/usr/local/opt/libressl/lib` 路径。这是必须的，因为这个库是由 `brew` 提供的，它在标准库搜索路径之外安装了一些东西。

如果你正在继承现有的库，那么这是必须做的。
