# C ABI

FFI 支持使用 C 应用二进制接口(ABI)来与 native code 交互。C ABI 是众多调用惯例中的一种，允许使用来自不同编程语言的对象(The C ABI is a calling convention, one of many, that allow objects from different programming languages to be used together.)

## Writing a C library for Pony

编写 Pony 可以使用的 C 库几乎和使用现有库一样简单。

让我们看一个完整的例子，它提供给 Pony 一个可用的 C 函数。Jump Consistent Hash 可以使用如下的 Pony 代码实现：

```pony
// Jump consistent hashing in pony, with an inline pseudo random generator

fun jch(key: U64, buckets: I64): I32 =>
  var k = key
  var b = I64(0)
  var j = I64(0)

  while j < buckets do
    b = j
    k = (k * 2862933555777941757) + 1
    j = ((b + 1).f64() * (U32(1 << 31).f64() / ((key >> 33) + 1).f64())).i64()
  end

  b.i32()
```

比如我们想要对纯 Pony 实现和下面头文件中现有 C 实现的性能进行比较

```C
#ifndef __JCH_H_
#define __JCH_H_

extern "C"
{
  int32_t jch_chash(uint64_t key, uint32_t num_buckets);
}

#endif
```

注意使用 `extern "C"`。如果库是由 C++ 构建，那么我们需要告诉编译器不要修改函数名(这里应该指的是 name mangling)，否则 Pony 将无法找到。对于 C 构建的库则不需要。

实现可能像下面这样：

```C
#include <stdint.h>
#include <limits.h>
#include "math.h"

// A reasonably fast, good period, low memory use, xorshift64* based prng
double lcg_next(uint64_t* x)
{
  *x ^= *x >> 12;
  *x ^= *x << 25;
  *x ^= *x >> 27;
  return (double)(*x * 2685821657736338717LL) / ULONG_MAX;
}

// Jump consistent hash
int32_t jch_chash(uint64_t key, uint32_t num_buckets)
{
  uint64_t seed = key;
  int b = -1;
  int32_t j = 0;

  do {
    b = j;
    double r = lcg_next(&seed);
    j = floor((b + 1)/r);
  } while(j < num_buckets);

  return (int32_t)b;
}
```

我们需要将 native code 编译进 shared library。这个例子是针对 OSX 的。其他平台上的具体细节可能会有所不同。

```
clang -fPIC -Wall -Wextra -O3 -g -MM jch.c >jch.d
clang -fPIC -Wall -Wextra -O3 -g   -c -o jch.o jch.c
clang -shared -lm -o libjch.dylib jch.o
```

使用这个新的 C 库的 Pony 代码就像我们先前所见使用 C 库的代码一样。

```pony
"""
This is an example of pony integrating with native code via the built-in FFI
support
"""

use "lib:jch"
use "collections"
use "random"
use @jch_chash[I32](hash: U64, bucket_size: U32)

actor Main
  var _env: Env

  new create(env: Env) =>
    _env = env

    let bucket_size: U32 = 1000000
    var random = MT

    for i in Range[U64](1, 20) do
      let r: U64 = random.next()
      let hash = @jch_chash(i, bucket_size)
      _env.out.print(i.string() + ": " + hash.string())
    end
```

We can now use ponyc to compile a native executable integrating pony and our C library. And that's all we need to do.

现在我们可以使用 `ponyc` 来编译集成 pony 代码和我们 C 库的可执行文件了。
