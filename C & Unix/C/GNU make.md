# GNU make

## make 命令概览

make 命令可以自动确认一个大型工程中需要重新编译的源码部分，然后执行命令来进行重新编译。

make 命令通常被用于 C 语言程序，但实际上它可以被用于任何支持使用 shell 命令来编译的编程语言。事实上，make 命令甚至不被受限于编程领域，它还可以被用于来执行这样类型的任务：当一个文件被更改时，需要自动更新其它文件。

## makefile 介绍

make 命令通过一个 makefile 文件来得知具体的执行步骤，通常 makefile 文件会告知 make 命令如何具体地执行编译和链接一个程序。

### rule 介绍

一个简单的 makefile 文件中会包含一些 rule，格式如下：

```makefile
target ... : prerequisites ...
  [tab] recipe
  [tab] ...
  [tab] ...
```

- `target` 可以是由程序生成的文件名称，例如一个 executable 文件或者 object 文件的名称。`target` 也可以是一个待执行的动作名称，例如 clean；
- `prerequisite` 是在创建 `target` 时的输入文件，一个 `target` 通常会依赖多个文件；
- `recipe` 是 make 命令要具体执行的动作，一个 `recipe` 可以有多个命令，可以位于一行或者多行。**注意**，每个 `recipe` 前面需要加上一个制表符（可以通过 .RECIPEPREFIX 变量来修改这个特殊符号）。

通常来说，`recipe` 会位于带有 `prerequisite` 的 rule 中，在 `prerequisite` 更改时会被用于创建 `target`。然而，指定了 `recipe` 的 rule 其实不必非得需要配置 `prerequisite`。例如，一个用于删除操作的名为 clean 的 `target` 就不需要配置 `prerequisite`。

### 使用 make 命令来处理 makefile

默认情况下，make 命令会读取当前目录中的 makefile 文件，然后处理 makefile 文件中的第一个 rule。

在 make 命令可以完全处理某个 rule 之前，会先处理这个 rule 所依赖的 `prerequisite`。

## 编写 makefile

### makefile 文件内容

makefile 文件中包含了五种类型的内容：显式 rule、隐式 rule、变量定义、指令和注释。

- 显式 rule 说明了何时以及如何重新 make 一个或多个文件，这些文件被称为 rule 的 `target`。显式 rule 会列出 `target` 所依赖的一些输入文件，这些文件被称为 `prerequisite`，同时也会给定一些用于创建或者更新 `target` 的 `recipe`；
- 隐式 rule 说明了何时以及如何基于文件名称来重新 make 一个或多个文件；
- 变量定义是将一个文本字符串值指定为一个变量的一行，变量在后续内容中可以被替换为文本字符串；
- 指令是使 make 命令在读取 makefile 时所执行的一些特殊操作的指令，包括了：
  - 读取其它 makefile 文件；
  - 根据变量的值来决定使用或者忽略部分 makefile 内容；
  - 将包含多行的字符串指定为一个变量；
- 在 makefile 中的注释以 `#` 开头。

### makefile 文件名称

默认情况下，make 命令会按以下名称顺序来查找 makefile 文件：`GNUmakefile`、`makefile`、`Makefile`。

通常会将 makefile 文件命名为 `makefile` 或者 `Makefile`。（更加推荐后者的方式，因为在列出目录内容时，`Makefile` 会像 `README` 一样出现在列表中更靠前的位置。）

可以通过 `-f` 或者 `--file` 选项来指定具体的 makefile 文件。

## 编写 rule

### rule 语法

通常来说，rule 的格式如下：

```makefile
targets : prerequisites
  [tab] recipe
  [tab] ...
```

或者如下：

```makefile
targets : prerequisites ; recipe
  [tab] recipe
  [tab] ...
```

`targets` 是文件名称，以空格分隔，可以使用通配符。通常只会有一个 `target`，但偶尔也会因为某些原因而有多个 `target`。

`recipe` 需要以一个制表符为开头。第一个 `recipe` 可以位于 `prerequisite` 的下一行，也可以位于 `prerequisite` 的同一行（需要使用分号来分隔）。

由于 `$` 符号用于标记变量引用的开始，因此需要使用 `$$` 来代表实际的 `$` 符号。

一个 rule 会告知 make 命令两件事情：`target` 何时会过期，以及如何在必要时候更新 `target`：

- `target` 的过期是根据 `prerequisite` 来判断的，`prerequisite` 是一组以空格分隔的文件名称。当 `target` 不存在或者早于任何一个 `prerequisite` 文件时（通过比较文件的 `last-modification` 时间），这个 `target` 就是已经过期了的；
- `target` 的更新是通过 `recipe` 来实现的，`recipe` 是一行或者多行可以执行的 shell 脚本。

### 在文件名称中使用通配符

一个文件名可以使用通配符来指定多个文件。

在 make 命令中可以使用的通配符是 `*`、`?`、`[…]`，`~` 则代表用户目录，这些符号的用法和在 Bourne shell 中的用法类似。

### phony target

phony `target` 并不是一个真实文件的名称，它只是通过明确的 make 命令（例如 make clean）来代表一组待执行的 `recipe` 的名称。

以如下的 rule 为例，这个 rule 只会简单地执行 `rm *.o temp` 命令，不会创建 `target` 对应的名为 clean 的文件，同时由于在工程中是大概率不会存在名为 clean 的文件，因此在每次调用 make clean 命令时，rule 中的 `recipe` （即 `rm *.o temp` 命令）都会被执行。

```makefile
clean:
  [tab] rm *.o temp
```

但是如果工程中恰好存在名为 clean 的文件的话，此时由于这个 rule 不存在 `prerequisite`，同时 `target` 对应的名为 clean 的文件始终会被认为是最新的（即没有被更改过），因此 make clean 命令便不会再被执行。

为了避免上述的问题，可以显式地将这个 `target` 声明为 phony 的。

```makefile
.PHONY: clean
clean:
  [tab] rm *.o temp
```

## 编写 rule 中的 recipe

### recipe 语法

makefile 文件中的大部分内容是使用 make 命令的语法，然而 `recipe` 是由 shell 来解释执行的，因此在 `recipe` 中使用的是 shell 语法。make 命令不会尝试去解释 shell 语法，它仅会在将 `recipe` 传递给 shell 执行之前，执行一些少量的特定翻译工作。

除了和 `target` 和 `prerequisite` 位于同一行的特殊 `recipe`，其它的每行 `recipe` 必须以制表符开头（可以通过 .RECIPEPREFIX 变量来修改这个特殊符号）。

### recipe 回声

通常来说，make 命令在执行每行 `recipe` 之前会打印它，这被称为回声（echoing）。

当行是以 `@` 为开头时，该行的 `recipe` 的回声功能会被禁用。make 命令将 `recipe` 传递给 shell 之前，会删除 `@`。

可以通过 `-n` 或者 `--just-print` 选项来使 make 命令仅打印而不会执行 `recipe`。

可以通过 `-s` 或者 `--silent` 选项来使 make 命令禁用所有 `recipe` 的回声功能。

### recipe 执行

除了在 .ONESHELL 这个特殊的 `target` 生效的时候，每行 `recipe` 都会在一个子 shell 环境中执行。

**注意**，这也意味着定义 shell 变量和调用类似 `cd` 之类的 shell 命令——这种为每个进程设置本地上下文的操作不会对下一行 `recipe` 生效。如果期望 `cd` 命令对下一行 `recipe` 生效，则可以将两行 `recipe` 的 shell 语句合并到同一行中。

## 参考资料

- [Make 命令教程 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2015/02/make.html)
- [GNU make](https://www.gnu.org/software/make/manual/make.html)
