# Package System

Pony 代码可以被组织到 __packages__ 中。每个程序和库都是一个 package，也可能有使用到其他的 package

## The package structure

package 是 Pony 的基本代码单元。它与文件系统的目录直接对应，所有目录中的 Pony 源码文件都在 package 之中。注意这不包含子目录中的文件。

每个源文件都在一个 package 中。因此，所有的 Pony 代码都是在 package 中。

一个 package 通常会被分割成几个源文件，尽管不必这样做。这纯粹是为了更好的组织代码，编译器会像处理单个文件般处理 package 内的所有代码

package 是类型和方法的私有边界：

1. 私有类型(名字以下划线开头)只能在定义它们的 package 中使用
2. 私有方法(名称以下划线开头)只能在定义它们的 package 中调用

它遵循 package 中的所有代码都互相信任的原则。

Pony 中没有 sub-package 的概念。比如 package "foo/bar" 和 "foo/bar/wombat" 可能是相关的，但是它们是两个独立的 package。"foo/bar" 不包含 "foo/bar/wombat" 亦不能访问其私有元素。
