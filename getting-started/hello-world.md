# Hello World: 您的第一个 Pony 程序

现在您已经成功地安装了 Pony 编译器，让我们开始编程！我们的第一个程序将沿袭传统，打印 "Hello, world"。首先，创建一个名为 `helloworld` 的目录：

```bash
$ mkdir helloworld
$ cd helloworld
```

__目录的名字重要么？__ 是的，很重要。她是您程序的名字！当您的程序被编译时，生成的可执行二进制文件将具有和您程序所在目录相同的名字。

## 代码

然后，在这个目录中创建一个名为 `main.pony` 的文件。

__文件的名字重要么？__ 对编译器来说并没有。Pony 所关心的是她们是否以 `.pony` 结尾，而不是文件名。但她们对您来说也许很重要！通过给出好的文件名，日后可以更容易找到您要查找的代码。

将下面的代码拷贝至您的文件：

```pony
actor Main
  new create(env: Env) =>
    env.out.print("Hello, world!")
```

## 进行编译

现在，编译她：

```bash
$ ponyc
Building .
Building builtin
Generating
Optimising
Writing ./helloworld.o
Linking ./helloworld
```

(如果您使用 Docker, 您需要这样 `$ docker run -v Some_Absolute_Path/helloworld:/src/main ponylang/ponyc`，当然这取决于您的 `helloworld` 目录的绝对路径)

看！她 build 了当前目录 `.` 和 Pony 内置的 `builtin`，生成了一些代码，进行了优化，创建了一个目标文件(object file)，并和它所需要库一起链接至可执行文件。如果你是一个 C/C++ 程序员，那对你来说都是有意义的，否则可能不会，但是没关系，你可以忽略它。

__等等，她经过链接？__ 是的，您无需一个 build 系统(像 `make`)。她为您处理了这些(包括在链接到C库时处理依赖关系的顺序，稍后将介绍)

## 运行

现在我们可以运行程序了：

```bash
$ ./helloworld
Hello, world!
```

恭喜，您已经完成了第一个 Pony 程序！我们会在下一节解释这些代码做了什么。
