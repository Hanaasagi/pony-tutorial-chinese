# 字面量(Literals)

***我们想要什么？***

Values!

***哪里需要他们？***

在我们的 Pony 程序中！

**什么都不要说了**

每一种编程语言都有字面量来编码某些具体类型的值，Pony 亦如此。

在 Pony 中你可以通过字面量表示布尔类型，数值类型，字符类型，字符串类型和数组类型。

## 布尔字面量(Bool Literals)

`true` 和 `false`，就是这样。

## 数值字面量(Numeric Literals)

数值字面量可以被用于编码有符号或无符号整数或者浮点数。

在大多数情况下 Pony 能够从上下文中推断字面量的具体类型(例如赋值给 field 或局部变量时或作为 方法/behaviour 的参数时)

可以使用以下某个数值类型的构造函数来帮助编译器来决定字面量的具体类型：

 * U8, U16, U32, U64, U128, USize, ULong
 * I8, I16, I32, I64, I128, ISize, ILong
 * F32, F64

```pony
let my_explicit_unsigned: U32 = 42_000
let my_constructor_unsigned = U8(1)
let my_constructor_float = F64(1.234)
```

整数字面量有十进制，十六进制，二进制

```pony
let my_decimal_int: I32 = 1024
let my_hexadecimal_int: I32 = 0x400
let my_binary_int: I32 = 0b10000000000
```

浮点字面量可以通过标准浮点数或者科学计数法来表示：

```pony
let my_double_precision_float: F64 = 0.009999999776482582092285156250
let my_scientific_float: F32 = 42.12e-4
```

## 字符字面量(Character Literals)

字符字面量需要被单引号(`'`)包围。

字符字面量与字符串字面量不同，会被编码成一个数值。通常是一个字节，`U8`。但是他们可以被强制转成任何整数类型：

```pony
let big_a: U8 = 'A'                 // 65
let hex_escaped_big_a: U8 = '\x41'  // 65
let newline: U32 = '\n'             // 10
```

支持下面的转义序列：

* `\x4F` hex escape sequence with 2 hex digits (up to 0xFF)
* `\a`, `\b`, `\e`, `\f`, `\n`, `\r`, `\t`, `\v`, `\\`, `\0`, `\"`

### 多字节字符字面量(Multibyte Character literals)

字符字面量可以包含多个字符。生成的整数值由一个接一个的字符所代表的单个字节组成，最后一个字符是最低有效字节：

```pony
let multiByte: U64 = 'ABCD' // 0x41424344
```

## 字符串字面量(String Literals)

字符串字面量由双引号(`"`)或者三引号(`"""`)所包围。他们能包含任意字节或者转义序列：

* `\u00FE` unicode escape sequence with 4 hex digits encoding one code point
* `\u10FFFE` unicode escape sequence with 6 hex digits encoding one code point
* `\x4F` hex escape sequence for unicode letters with 2 hex digits (up to 0xFF)
* `\a`, `\b`, `\e`, `\f`, `\n`, `\r`, `\t`, `\v`, `\\`, `\0`, `\"`

Each escape sequence encodes a full character, not byte.

```pony

    let pony = "🐎"
    let pony_hex_escaped = "p\xF6n\xFF"
    let pony_unicode_escape = "\U01F40E"

    env.out.print(pony + " " + pony_hex_escaped + " " + pony_unicode_escape)
    for b in pony.values() do
      env.out.print(Format.int[U8](b, FormatHex))
    end

```

所有的字符串字面量支持多行字符串：

```pony

let stacked_ponies = "
🐎
🐎
🐎
"
```

### 字符串字面量和编码(String Literals and Encodings)

字符串字面量包含我们从源代码文件中读取的字节。他们实际的值依赖于源代码的编码。

考虑下面的例子：

```pony
let u_umlaut = "ü"
```

如果包含这代码的文件是 *UTF-8* 编码的，`u_umlaut` 的字节值会是 `\xc3\xbc`。如果是 *ISO-8559-1* (Latin-1) 则其值为 `\xfc`。

### 三引号字符串(Triple quoted Strings)

对于要在字符串字面量中嵌入多行文本，可以使用三引号字符串。

```pony
let triple_quoted_string_docs =
  """
  Triple quoted strings are the way to go for long multiline text.
  They are extensively used as docstrings which are turned into api documentation.

  They get some special treatment, in order to keep pony code readable:

  * The string literal starts on the line after the opening triple quote.
  * Common indentation is removed from the string literal
    so it can be conveniently aligned with the enclosing indentation
    e.g. each line of this literal will get its first two whitespaces removed
  * Whitespace after the opening and before the closing triple quote will be
    removed as well. The first line will be completely removed if it only
    contains whitespace. e.g. this strings first character is `T` not `\n`.
  """
```

## 数组字面量(Array Literals)

数组字面量需要被方括号包围。数组字面量的元素可以是任意类型的表达式。他们使用分号或者换行分隔：

```pony
let my_literal_array =
  [
    "first"; "second"
    "third one on a new line"
  ]
```

### Type inference

如果数组的类型不确定，数组字面量的类型是 `Array[T] ref`， `T`(元素的类型)被推断为所有元素类型的 union：

```pony
let my_heterogenous_array =
  [
    U64(42)
    "42"
    U64.min_value()
  ]
```

上面的例子数组类型将会是 `Array[(U64|String)] ref` 因为数组包含 `String` 和 `U64` 元素。

如果数组字面量所赋给的变量或者调用参数具有类型，字面量会被强制类型转换：

```pony
let my_stringable_array: Array[Stringable] ref =
  [
    U64(0xA)
    "0xA"
  ]
```

这里 `my_stringable_array` 会被强制转换为 `Array[Stringable] ref`。这之所以能够工作是因为 `Stringable` 是一个 trait 同时被 `String` 和 `U64` 实现。

It is also possible to return an array with a different [Reference Capability](../capabilities/index.md) than `ref` just by specifying it on the type:

也可以通过在类型上指定，返回一个不同于 `ref` 的 [Reference Capability](../capabilities/index.md) 的数组。/*待定*/


```pony
let my_immutable_array: Array[Stringable] val =
  [
    U64(0xBEEF)
    "0xBEEF"
  ]
```

This way array literals can be used for creating arrays of any [Reference Capability](../capabilities/index.md).

### `As` 表达式(`As` Expression)

通过 `as` 表达式可以在数组元素强制转换成某种类型时给予提示。紧接着在左方括号后添加期望的数组元素类型表达式，用冒号分隔：

```pony
let my_as_array: Array[Stringable] ref =
  [ as Stringable:
    U64(0xFFEF)
    "0xFFEF"
    U64(1 + 1)
  ]
```

根据 `as` 表达式，数组字面量被强制转为 `Array[Stringable] ref`。

如果在左侧指定了类型，则需要与 as 表达式中的类型完全匹配。

### Arrays and References

使用字面量构造一个数组会创建对其元素的新的引用。因此，为了达到技术上百分之百的正确，数组元素会被推断为实际元素类型的别名。如果所有元素的类型都是 `T`，数组字面量将会被推断为 `Array[T!] ref`，别名 `T` 的一个数组。

It is thus necessary to use elements that can have more than one reference of the same type (e.g. types with `val` or `ref` capability) or use ephemeral types for other capabilities (as returned from [constructors](../types/classes.md#constructors) or the [consume expression](../capabilities/consume-and-destructive-read.md)).
