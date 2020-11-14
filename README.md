# Makefile 简明教程

>  英文原文地址：https://www.cs.colby.edu/maxwell/courses/tutorials/maketutor/

Makefiles 是组织代码编译的一种方式。通过这篇简明教程，虽然你不能完整学会 `make` 指令，但是你可以使用 makefile 来组织小到中型的项目啦。

## 一个 简单的例子

我们来从下面的三个文件开始吧：`hellomake.c`，`hellofunc.c`，`hellomake.h`。这是一个经典 C 语言程序，代码根据功能组织在不同的文件中。

_hellomake.c_

```c
#include <hellomake.h>

int main() {
  // 调用另一个文件里的函数
  myPrintHelloMake();
  
  return (0);
}
```

_hellofunc.c_

```c
#include <stdio.h>
#include <hellomake.h>

void myPrintHelloMake() {
  printf("Hello makefiles!\n");
  
  return;
}
```

_hellomake.h_

```
/* example include file */
void myPrintHelloMake(void);
```



一般情况下，我们通过下面的指令来编译代码：

```bash
gcc -o hellomake hellomake.c hellofunc.c -I.
```

我们来说明下这个指令：

1. 我们编译两个 `.c` 文件
2. 命名了编译后的可执行文件为 `hellomake`
3. `-I.` 告诉 gcc 在当前目录中寻找 `hellomake.h`

如果没有使用 makefile，我们在调试开发的时候，可以在终端上输入 `向上方向键` 来快速显示上次的指令（尤其是你有多个 `.c` 文件需要编译的时候）。

然而，通过上面的直接输入编译指令的方式存在两个弊端：

- **弊端一：**不方便呀！当你换了电脑之后，你要重新再输入上面的指令。
- **弊端二：**编译效率低下！即使你只是修改了项目中的一个 `.c` 文件，每次编译时，还是需要编译所有的文件，这无疑是效率低下，浪费时间。

所以接下来，请出本文的主角 —— makefile。

## Makefile1

```makefile
hellomake: hellomake.c hellofunc.c
	gcc -o hellomake hellomake.c hellofunc.c -I.
```

把上述的内容，放入到 `Makefile` 或者 `makefile` 文件，然后在命令行输入 `make` 命令，就能够直接执行编译了。有以下几点我们需要关注下：

1. 如果 `make` 后面没有跟任何参数，那么他就会执行 makefile 的第一条规则。
2. 把命令依赖的文件放在第一行的 `:` 后面，这样 `make` 就能知道，当依赖文件变化时， `hellomake` 规则需要重新执行。
3. 注意，第二行 `gcc` 前面，是一个 `tab` 制表符！不要使用空格！

通过这样简单的 Makefile，我们已经解决了弊端一的问题，即：我们不需要每次都输入编译指令了。

然而，现在还不够高效，即使只修改了一个文件，还是需要全量编译（即编译所有的源文件）。为了使编译更加高效，让我们继续往下看。

## Makefile2

```makefile
CC=gcc
CFLAGS=-I.

hellomake: hellomake.o hellofunc.o
	$(CC) -o hellomake hellomake.o hellofunc.o
```

我们定义了两个常量 `CC`、 `CFLAGS`，这两个常量告诉 `make` 怎么去编译 `hellomake.c` 和 `hellofunc.c`。其中 `CC` 告诉 make 使用哪个 C 编译器，`CFLAGS` 说明了编译指令的参数列表。通过把 `hellomake.o` 和 `hellofunc.o` 放到依赖列表中， `make` 指令就知道每次需要分别编译 `.c` 文件，然后再把他们编译为可行性文件 `hellomake`。

终端执行效果如下：

```bash
➜  makefile-tourial git:(master) ✗ make
gcc -I.   -c -o hellomake.o hellomake.c
gcc -I.   -c -o hellofunc.o hellofunc.c
gcc -o hellomake hellomake.o hellofunc.o
➜  makefile-tourial git:(master) ✗ 
```

这种形式的 makefile 对小型的项目还是比较方便的。然而，还是有个问题，那就是依赖文件的更新。设想下，即使你修改了`hellomake.h` 文件，`make` 指令不会重新编译文件。

为了解决这个问题，我们需要告诉 `make` 一件事情：即`.c` 文件和 `.h` 文件间的依赖关系。好，我们继续往下看。

## Makefile3

```makefile
CC=gcc
CFLAGS=-I.
DEPS = hellomake.h

%.o: %.c $(DEPS)
	$(CC) -c -o $@ $< $(CFLAGS)

hellomake: hellomake.o hellofunc.o
	$(CC) -o hellomake hellomake.o hellofunc.o
```

相较于上个版本，我们先是增加了一个 `DEPS`：这里列出了 `.c` 文件所依赖的 `.h` 文件集合。

接着，我们定义了一个了规则 `%.o: %.c $(DEPS)`：它说明了 `.o` 文件是取决于 `.c` 文件和 `DEPS` 里的 `.h` 文件。

接下来我们看下规则 `$(CC) -c -o $@ $< $(CFLAGS)`，意思是说，为了生成这些 `.o` 文件，`make` 指令使用了 `CC` 定义的编译器来编译 `.c` 文件：

- `-c` 说明了是为了生成目标文件（object files）
- `$@` 代表 `:` 左边的内容，即：`%.o`
- `$<` 是依赖列表里的第一项，即：`%.c`
- `CFLAGS` 和之前的说明一样，就是编译的指令参数了(flag）

执行效果如下：

```bash
➜  makefile-tourial git:(master) ✗ make
gcc -c -o hellomake.o hellomake.c -I.
gcc -c -o hellofunc.o hellofunc.c -I.
gcc -o hellomake hellomake.o hellofunc.o
➜  makefile-tourial git:(master) ✗ 
```

最后，我们再来做下简化，使编译更具通用性。我们使用 `$@` 和 `$^` 来分别表示 `:`  的左侧和右侧。在下面的例子里，所有 include 文件会作为 `DEPS` 的一部分，所有目标文件（object files）会作为 `OBJ` 的一部分。

## Makefile4

```makefile
CC=gcc
CFLAGS=-I.
DEPS = hellomake.h
OBJ = hellomake.o hellofunc.o

%.o: %.c $(DEPS)
	$(CC) -c -o $@ $< $(CFLAGS)
	
hellomake: $(OBJ)
	$(CC) -o $@ $^ $(CFLAGS)
```

执行效果如下：

```bash
➜  makefile-tourial git:(master) ✗ make
gcc -c -o hellomake.o hellomake.c -I.
gcc -c -o hellofunc.o hellofunc.c -I.
gcc -o hellomake hellomake.o hellofunc.o -I.
```

让我们来进一步思考下：

- 我们能不能把 `.h` 的文件都放到一个专门的 `inlcude` 目录，把 `.c` 文件都放到一个专门的 `src`目录？
- 我们能不能把这些烦人的 `.o` 文件都隐藏起来？

当然是可以的！我们会在下一个 makefile 中把对应的文件放到 `include` 和  `lib`文件夹中，并且把生成的目标文件都放到 `src` 的 `obj` 子目录中。除此之外，我们还可以定义任何我们想包含的库文件，比如常用的 math library `-lm`。这个 makefile 放在 `src` 目录里。

需要注意的是，我们还定义了一个 `clean` 规则，用来把生成的目标文件清除（使用 `make clean` 命令）。`.PHONY` 防止 `make` 清除名为 `clean` 的文件。

文件路径为

```bash
➜  src git:(master) ✗ tree          
.
├── hellofunc.c
├── hellomake
├── hellomake.c
├── makefile
└── obj
    ├── hellofunc.o
    └── hellomake.o

1 directory, 6 files
```

## Makefile5

```makefile
IDIR = ../include
CC=gcc
CFLAGS=-I$(DIR)

ODIR=obj
LDIR=../lib

LIBS=-lm

_DEPS = hellomake.h
DEPS=$(patsubst %,$(IDIR)/%,$(_DEPS))

_OBJ = hellomake.o hellofunc.o
OBJ=$(patsubst %,$(ODIR)/%,$(_OBJ))

$(ODIR)/%.o: %.c $(DEPS)(
	$(CC) -c -o $@ $< $(CFLAGS)
	
hellomake: $(OBJ)
	$(CC) -o $@ $^ $(CFLAGS) $(LIBS)
	
.PHONY: clean

clean:
	rm -f $(ODIR)/*.o *~ core $(INCDIR)/*~
```

运行结果

```bash
➜  src git:(master) ✗ make
gcc -c -o obj/hellomake.o hellomake.c -I../include
gcc -c -o obj/hellofunc.o hellofunc.c -I../include
gcc -o hellomake obj/hellomake.o obj/hellofunc.o -I../include
```

> 注意要在 `src` 目录下运行，并且要把 `.h` 文件放到 `include` 目录里

好了，到目前为止，你已经有了一个不错的 makefile 了，现在你能 hold 住一个中型的项目了。你也可以增加更多的规则到 makefile 里，你甚至可以在一个规则中调用另一个规则。

想知道更多关于 makefile 和 make 的信息，就去查阅 [GNU Make Manual](https://www.gnu.org/software/make/manual/make.html) 吧！
