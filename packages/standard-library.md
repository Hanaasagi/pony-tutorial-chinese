# 标准库(Standard library)

Pony 标准库是 package 的集合，可以根据需要来提供各式各样的功能。比如，__files__ package 提供了文件访问；__collections__ package 提供了 generic lists, maps, sets 等。

标准库中也有一个特殊的 package 名为 __builtin__。这包含编译器必须特殊对待的各式类型，并且所有的 Pony 代码都需要它们。Pony 源文件隐式地包含了 `use "builtin"` 语句。这意味着所有定义在 builtin package 中的类型都会自动的导入至所有的 Pony 源文件的命名空间中。

标准库文档的 [在线版本](http://stdlib.ponylang.org/)
