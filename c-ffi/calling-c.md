# Calling C from Pony

FFI 内置于 Pony 中，并且可以直接引用。不需要进行额外的编码，配置，包装，调用接口。

下面是一个 Pony 标准库中的 FFI 调用例子。它像正常的方法调用，不过有一些区别：

```pony
@fwrite[U64](data.cstring(), U64(1), data.size(), _handle)
```

主要区别是函数名前有 `@` 符号，这表明是一个 FFI 调用。任何情况下 Pony 代码中出现 `@` 都是一个 FFI 调用。

另一个关键区别是函数的返回类型是在函数名后使用方括号指定的。这是由于编译器需要知道返回的值(如果有)是什么类型，但是无法进行推断，所以需要显式地指定。

FFI 调用在参数上也有一些不同。对于第二个参数，传递了 `1`，我们需要指定它是 `U64`。和刚才相同的原因，编译器需要知道参数的类型大小。

## Safely does it

__调用 FFI 时参数和返回类型必须正确，这点是极为重要的么。__ 编译器无从知晓 native code 的信息，它会按你说的做。类型错误会导致不合法的数据传入 FFI 函数或者返回给 Pony，这会令程序崩溃。

为了帮助避免 bug，Pony 允许你提前指定 FFI 函数的类型签名。虽然你必须在 FFI 调用处保证参数类型正确，且符合声明的签名。这意味着如果类型错误，则至少存在两处 bug。这并不会在和 native code 参数类型不一致时起到帮助，但是将使你免受琐碎的错误和拼写错误。

FFI 签名使用 `use` 命令声明。下面是一个来此于标准库的例子：

```pony
use @SSL_CTX_ctrl[I32](ctx: Pointer[_SSLContext] tag, op: I32, arg: I32,
  parg: Pointer[U8] tag) if windows

use @SSL_CTX_ctrl[I64](ctx: Pointer[_SSLContext] tag, op: I32, arg: I64,
  parg: Pointer[U8] tag) if not windows

class SSLContext val
  new create() =>
    // set SSL_OP_NO_SSLv2
    @SSL_CTX_ctrl(_ctx, 32, 0x01000000, Pointer[U8])
```

`@` 符号告诉我们 `use` 命令是一个 FFI 签名声明。这里所指定的类型会被认为是权威的，任何和签名不同的 FFI 调用都将会抛出异常。

注意我们不需要在调用处指定返回类型，因为签名声明已经指定过一次了。然而，如果你想要再次指定也是完全可以接受的。

`use @` 命令也可以像 `use` 命令那样使用条件。这在一些场景下十分有用，比如 Windows 下的 `SSL_CTX_ctrl` 和其他平台具有不同签名。

## C types

许多 C 函数需要一些与 Pony 不完全相同的类型。为此提供了许多特性。

对于没有返回值的 FFI 函数(C 中的 `void`)，返回值类型应当被指定为 `[None]`。

Pony 的字符串是一个具有 header 和 field 的对象，但在 C 是 `char*`。String 的 `.cstring()` 函数可以提供一个合法的指针。上面 `fwrite` 示例的第一个参数就用到了这个。

Pony classes correspond directly to pointers to the class in C.

对于指向简单类型的 C 指针，比如 `U64`，可以使用 Pony 中 reference capability 为 `tag` 的多态类型 `Pointer[]`。`void*` 应当使用 `Pointer[U8] tag`。这可以在上面 `SSL_CTX_ctrl` 的示例中看到。

为了传递值的指针，可以使用 `addressof` 运算符，就像 C 中的取址一样。下面是一个标准库中传递 `U64` 的地址至参数为 `uint64_t*`的 FFI 函数的示例：

```pony
var len = U64(0)
@pcre2_substring_length_bynumber_8[I32](_match, i.u32(), addressof len)
```

### Get and Pass Pointers to FFI

接收 C 数据结构的指针，你需要声明 primitives 的指针

```pony
primitive _XDisplayHandle
primitive _EGLDisplayHandle

let x_dpy = @XOpenDisplay[Pointer[_XDisplayHandle]](U32(0))
if x_dpy.is_null() then
  env.out.print("XOpenDisplay failed")
end

let e_dpy = @eglGetDisplay[Pointer[_EGLDisplayHandle]](x_dpy)
if e_dpy.is_null() then
  env.out.print("eglGetDisplay failed")
end
```

### Read Struct Values from FFI

C 语言一个常见的模式是将结构体指针传到函数，然后函数会填充结构体。在 Pony 中你需要创建一个结构体，然后使用 `MaybePointer`：

```pony
struct Winsize
  var height: U16 = 0
  var width: U16 = 0

  new create() => None

let size = Winsize

@ioctl(0, 21523, MaybePointer[Winsize](size))

env.out.print(size.height.string())
```

## FFI functions raising errors

FFI 函数可能会抛出 Pony 的异常。现有的 C 库中的函数不太可能会这样做，但为 Pony 专门编写的库可能会这样做。

可能会抛出错误的 FFI 调用必须使用 `?` 标记：

```pony
@os_send[U64](_event, data.cstring(), data.size()) ? // May raise an error
```

如果使用签名声明，那么必须使用同样的方式去标记。FFI 调用处也需要标记。

```pony
use @os_send[U64](ev: Event, buf: Pointer[U8] tag, len: U64) ?

@os_send(_event, data.cstring(), data.size())? // May raise an error
```
