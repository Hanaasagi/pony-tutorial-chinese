# 所需之物

您需要一个文本编辑器和 [ponyc](https://github.com/ponylang/ponyc) 编译器。这边足够了。

## Pony 编译器

在踏上旅途之前，请确保您正确地安装了 Pony 编译器。参考这份指南[installation instructions](https://github.com/ponylang/ponyc/blob/master/README.md#installation)

## 文本编辑器

虽然您可以用任何编辑器来写代码，但下面这些提供了 Pony 语言的支持：

* [Sublime Text](http://www.sublimetext.com/). There's a [Pony Language](https://packagecontrol.io/packages/Pony%20Language) package.
* [Atom](https://atom.io/). There's a [language-pony](https://atom.io/packages/language-pony) package.
* [Emacs](https://www.gnu.org/software/emacs/emacs.html). There's a [ponylang-mode](https://github.com/seantallen/ponylang-mode).
* [Vim](http://www.vim.org). There's a [ pony-vim-syntax](https://github.com/dleonard0/pony-vim-syntax) plugin. Install `pony` with `vim-plug` or `pathogen`.
* [Microsoft Visual Studio](http://www.visualstudio.com/). There's a [VS-pony](https://github.com/CausalityLtd/VS-pony) plugin.
* [BBEdit](http://www.barebones.com/products/bbedit/). There's a [bbedit-pony](https://github.com/TheMue/bbedit-pony) module.

如果您所爱的编辑器并不在上述之中，我们很高兴您能制作一个 Pony 的支持包

## 编译器

Pony 是一门编译型语言，而不是解释型语言。事实上，更进一步地讲， Pony 采用了运行前编译(__ahead-of-time__, AOT)，而不是即时编译(_just-in-time_， JIT)

这便意味着，一旦您 build 好您的程序，您可以一遍又一遍的运行，而不需要依赖编译器或者虚拟机之类的其他东西。她是一个完整的程序。

但这也意味着您需要 build 您的程序后才能运行她。在解释型语言或即时编译(JIT)语言中，您更倾向于这样做来运行程序：

```bash
$ python helloworld.py
```

或者您将 __shebang__ 添加到您的程序之中，类似 `#!/usr/bin/env python`。之后 `chmod` 赋予执行位，便可以这样：

```bash
$ ./helloworld.py
```

使用 Pony 时，这些都是不能做的！

## 编译您的程序

如果您位于和您程序相同的目录，您可以这样做：

```bash
$ ponyc
```

这会告诉 Pony 编译器您当前的工作目录包含您的源代码，请对她进行编译。如果您的源代码位于其他的目录中，你可以告诉 ponyc 她的位置。

```bash
$ ponyc path/to/my/code
```

还有一些其他的选项，我们稍后会进行介绍。
