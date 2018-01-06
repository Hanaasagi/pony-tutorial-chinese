# Callbacks

一些 C API 允许程序员指定回调函数去完成一些工作。比如， SQLite API 有一个函数叫做 `sqlite3_exec`，它执行一条 SQL 语句然后在返回结果的每一行上调用一个程序员指定的函数。这样的函数被称为回调函数。一些 Pony 的函数可以被作为回调函数传入。


## Bare functions

一般的 Pony 函数有接收者，会作为函数的隐式参数传入。因为这个，一般函数不能作为 C API 的回调函数。为此，你可以使用 _bare functions_，这种函数没有接收者。

你可以在函数名前使用 `@` 符号来定义 bare function。

```pony
class C
  fun @callback() =>
    ...
```

该函数可以通过 `addressof` 运算符作为回调传递给 C API。

```pony
@setup_callback(addressof C.callback)
```

应注意，除类型外可以使用对象引用作为方法访问运算符(`.`)的左侧

因为 bare method 没有接收者，他们不能在方法体中使用 `this` 标识符(显式或者隐式的通过 field 方法)，不能使用 `this` viewpoint 来适配类型，也不能指定 receiver capability。

## Bare lambdas

Bare lambda 是特殊的 lambda，可以定义 bare function。除了需要使用 `@` 作为前缀这点外，bare lambda 或者 bare lambda type 和普通的 lambda 具有相同的语法。bare lambda 的底层值相当于一个 C 函数指针，这意味着 bare lambda 可以被直接作为 C 函数的回调传入。对 bare method 进行部分应用会得到 bare lambda。

```pony
let callback = @{() => ... }
@setup_callback(callback)
```

Bare lambdas 可以用于定义含有函数指针的结构体，比如：

```pony
struct S
  var fun_ptr: @{()}
```

这个 Pony 结构体等于下面的 C 结构体：

```c
struct S
{
  void(*fun_ptr)();
};
```

与 bare function 一样，bare lambda 不同捕获自由变量，不能使用 `this`，也不能指定 receiver capability。另外一个 bare lambda 对象总是具有 `val` 的 capability。

一般的 lambda 类型和 bare lambda 类型彼此间不是子类型。

## An example

考虑之前提及的 SQLite，当客户端代码调用 `sqlite3_exec`，一个 SQL 查询在数据库中执行，在 SQL 语句返回的每一行上调用回调函数。下面是 `sqlite3_exec` 的函数签名：

```c
typedef int (*sqlite3_callback)(void*,int,char**, char**);

...

SQLITE_API int SQLITE_STDCALL sqlite3_exec(
sqlite3 *db,                /* The database on which the SQL executes */
const char *zSql,           /* The SQL to be executed */
sqlite3_callback xCallback, /* Invoke this callback routine */
void *pArg,                 /* First argument to xCallback() */
char **pzErrMsg             /* Write error messages here */
)
{
  ...
  xCallback(pArg, nCol, azVals, azCols)
  ...
}
```

`sqlite3_callback` 是回调函数的类型。第一个参数是传入 `sqlite3_exec` 函数的 `pArg`，第二个参数是待处理行的列的数量，第三个参数是每列的数据，第四个三叔是每列的名字。

下面是一些使用 `sqlite3_exec` 去查询 SQLite 数据库的 Pony 代码的基本骨架，一个使用了 bare method，另一个使用了 bare lambda：

```pony
class SQLiteClient
  fun client_code() =>
    ...
    @sqlite3_exec[I32](db, sql.cstring(), addressof this.method_callback,
                       this, addressof zErrMsg)
    ...

  fun @method_callback(client: SQLiteClient, argc: I32,
    argv: Pointer[Pointer[U8]], azColName: Pointer[Pointer[U8]]): I32
  =>
    ...
```

```pony
class SQLiteClient
  fun client_code() =>
    ...
    let lambda_callback =
      @{(client: SQLiteClient, argc: I32, argv: Pointer[Pointer[U8]],
        azColName: Pointer[Pointer[U8]]): I32
      =>
        ...
      }

    @sqlite3_exec[I32](db, sql.cstring(), lambda_callback, this,
                       addressof zErrMsg)
    ...
```

将目光放于回调相关的部分，回调函数使用 `addressof this.method_callback`(或者直接为 bare lambda)作为第三个参数传入 `sqlite3_exec`。第四个参数是 `this`，它是调用回调函数时的第一个参数。回调函数在 `sqlite3_exec` 中通过调用 `xCallback` 来调用。
