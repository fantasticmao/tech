# 《鳥哥的 Linux 私房菜》笔记 - 认识与学习 bash

## 认识 bash 这个 shell

### 系统的合法 shell 与 /etc/shells 功能

由于早年的 Unix 发展者众多，所以依据不同的 shell 发展者，就有许多不同的版本，例如常听到的 Bourne Shell（sh）、在 Sun 里头预设的 C Shell、商业上常用的 K Shell、还有 TCSH 等等。每一种 shell 都各有其特点，Linux 使用的 shell 版本称为「 Bourne Again Shell（简称 bash）」，这个 shell 是 Bourne Shell 的增强版，也是基准于 GUN 的架构下发展出来的。

可以通过检查 /etc/shells 文件来查询当前系统可以使用的 shell。

### bash shell 的功能

- 命令编修能力：history
- 命令与档案补全功能：[tab] 按键
- 命令别名设定功能：alias
- 工作控制、前景背景控制：job control、foreground、background
- 程序化脚本：shell script
- 通配符：wildcard
- ......

### 命令的下达与快速编辑

```bash
~$command [-options] parameter1 parameter2 ...
```

下达命令的详细说明：

1. 一行命令中第一个输入的部分是「命令（command）」或「可执行文件」（例如 shell script）。
2. command 是命令的名称，例如变换工作目录的命令为 cd。
3. [-options] 是命令的选项，通常选项前会带 - 符号，例如 -h，有时候会使用选项的完整全名，此时选项前会带 -- 符号，例如 --help。
4. parameter1 parameter2 ... 是属于 options 或属于 command 的参数。
5. command、options、parameter 以空格分隔，不论空几格，shell 都视为一格。
6. 在按下 [enter] 按键后，该命令就立即执行。[enter] 按键代表着一行命令的开始启动。
7. 命令太长的时候，可以使用「反斜杠（\）」来转义 [enter] 符号，使命令连续到下一行。

| 组合键     | 功能与示范               |
| ---------- | ------------------------ |
| [ctrl] + u | 从光标向前删除命令串     |
| [ctrl] + k | 从光标向后删除命令串     |
| [ctrl] + a | 移动光标到命令串的最前面 |
| [ctrl] + e | 移动光标到命令串的最后面 |

## shell 的变量功能

### 变量的获取 & 设置 & 修改 & 取消

```bash
~$echo $PATH
~$echo ${PATH}
~$myname=VBird
~$myname=maomao
~$unset myname
```

使用 `echo $variable` 或 `echo ${variable}` 获取一个变量的值。

使用「等号（=）」设置和修改一个变量。变量的设置规则：

1. 变量和变量值以一个等号连接。
2. 等号两边不能有空格字符。
3. 变量名称只能是英文字母和数字，并且开头不能是数字。
4. 可以使用「反斜杠（\）」转义特殊字符，例如 `var=hello\ world && echo $var` 输出 hello world。
5. 变量值中若有空格字符，可以使用「"」或「'」将变量内容结合起来
    - 双引号内的特殊字符如 $ 等，可以保持原本的特性，例如 `var="shell is $SHELL" && echo $var` 输出 shell is /bin/bash。
    - 单引号内的特殊字符则仅为一般字符，例如 `var='shell is $SHELL' && echo $var` 输出 shell is $SHELL。
6. 在一串命令的执行中，若需要使用其它额外的命令所提供的资料时，可以使用「command」或「$(command)」，例如 ``var=`pwd` && echo $var`` 输出 /Users/maomao。
7. 若该变量需要在其它程序执行，则可以使用 `export variable` 使变量成为环境变量。
8. 通常大写字符为系统预设变量。

使用 `unset variable` 取消一个变量的定义。

### 环境变量的功能

使用 `env` 查看环境变量，常见的环境变量说明：

- HOME 代表用户的家目录
- USER 代表当前用户
- PATH 代表可执行文件的搜索路径
- SHELL 代表当前 shell 环境的版本
- LANG 代表当前 shell 环境的语言

使用 `set` 查看所有变量，常见的变量说明：

- PS1 代表提示字符
- $ 代表当前 shell 的 PID
- ? 代表上一个执行命令的返回结果

### 变量的键盘读取 & 声明

使用 `read [-pt] variable` 读取键盘输入。

使用 `declare [-aixr] variable` 声明变量的类型。

### 数组变量

使用 `var[index]=content` 定义数组变量。

使用 `echo "var[index]"` 或 `echo ${var[index]}` 获取数组变量值。

## 命令别名和历史命令

### 命令别名 alias

使用 alias 设置命令别名，例如 `alias ll='ls -al'` 。

使用 unalias 取消设置命令别名，例如 `unalias ll`。

### 历史命令 history

在正常情况下，历史命令的读取和记录是这样的：

- 在 bash 登陆 Linux 主机时，系统会主动地从 ~/.bash_history 文件中读取曾经下达过的命令。
- 在 bash 退出 Linux 主机时，系统会将最近的 HISTFILESIZE 笔命令写入到 ~/.bash_history 文件中。

使用 `history n` 列出最近记忆的 n 个命令，使用 `history -w` 强制写入近期的命令，使用 `history -c` 清空当前 bash 中的所有历史命令。

使用 `!number` 执行最近的第 number 个命令，使用 `!command` 执行最近的 command 命令，使用 `!!` 执行上一个命令。

## bash 的操作环境

### PATH 和指令搜索顺序

系统执行命令的顺序：

1. 执行相对或绝对路径的命令，例如「/bin/ls」、「./ls」。
2. 执行 alias 匹配的命令。
3. 执行 bash 的内建（builtin）命令。
4. 搜索 $PATH 参数的顺序，并执行第一个匹配的命令。

### bash 的环境配置文件

在开始介绍 bash 的配置文件之前，需要先介绍 login shell 与 non-login shell。因为 login shell 与 non-login shell 读取的配置文件是不相同的。

- login shell：获取 bash 时需要完整的登录流程，称为 login shell，它会读取 /etc/profile 系统级别的配置文件和 ~/.bash_profile 或 ~/.bash_login 或 ~/.profile 三个用户级别的配置文件中的一个。
- non-login shell：获取 bash 时不需要重复的登录流程，称为 non-login shell，它仅会读取 ~/.bashrc 配置文件。

下面开始逐个介绍 bash 的环境配置文件：

- /etc/profile：在 login shell 情况下读取，它可以通过当前用户的 UID 来定义很多重要的变量，例如：PATH、MAIL、USER、HOSTNAME 等等，还可以读取一些指定的外部资料，例如 /etc/profile.d/\*.sh、/etc/locale.conf 等等。
- ~/.bash_profile：在读取 /etc/profile 之后，再读取按 ~/.bash_profile、~/.bash_login、~/.profile 顺序匹配的三个配置文件中的第一个。这也意味着，如果 ~/.bash_profile 配置文件存在，则其它两个配置文件都不会被读取。
- ~/.bashrc：在 non-login shell 情况下读取。
- ~/.bash_history：在登录 bash 时读取，用于初始化历史命令。
- ~/.bash_logout：在退出 bash 时读取，用于执行在退出系统之前的相关命令。

### 终端的环境设置

使用 `stty [-a]` 查询当前终端的环境设置。在输出结果中，`^` 表示 [ctrl] 按键的意思，例如 `^C` 代表 [ctrl] + c。常见的终端环境设置如下：

| 设置选项 | 代表含义                                                 |
| -------- | -------------------------------------------------------- |
| into     | 发送一个中断信号（interrupt signal）给当前正在执行的程序 |
| quit     | 发送一个退出信号（quit signal）给当前正在执行的程序      |
| susp     | 发送一个终端结束信号（terminal stop signal）             |
| stop     | 暂停终端输出                                             |
| start    | 在 stop 之后，恢复终端输出                               |
| kill     | 清除当前行的命令                                         |
| erase    | 向后删除字符                                             |
| eof      | 在终端输入时，输入结束（end of file）                    |

bash 预设的按键组合如下：

| 组合按键   | 执行结果               |
| ---------- | ---------------------- |
| [ctrl] + c | 中断目前正在执行的程序 |
| [ctrl] + z | 暂停目前正在执行的命令 |
| [ctrl] + s | 暂停屏幕输出           |
| [ctrl] + q | 恢复屏幕输出           |
| [ctrl] + u | 清除当前行的命令       |
| [ctrl] + d | 输入结束               |
| [ctrl] + m | 同 [enter]             |

### 通配符和特殊符号

在 bash 中常用的通配符：

| 符号 | 含义                                                                       |
| ---- | -------------------------------------------------------------------------- |
| \*   | 零个或多个字符                                                             |
| \?   | 一个任意字符                                                               |
| []   | 一个在 [] 内的任意字符，例如 [abc] 表示 a 或 b 或 c 的一个字符             |
| [-]  | 一个在编码顺序区间内的任意字符，例如 [a-c] 表示 a 或 b 或 c 的一个字符     |
| [^]  | 一个不在 [] 内的任意字符，例如 [^abc] 表示 非 a 且 非 b 且 非 c 的一个字符 |

在 bash 中常用的特殊符号：

| 符号  | 含义                           |
| ----- | ------------------------------ |
| #     | 注释符号                       |
| \\    | 转义符号                       |
| /     | 目录符号：分隔目录路径         |
| \|    | 管线符号：分隔管线命令         |
| ;     | 分隔符号：分隔连续下达的命令   |
| ~     | 使用者的家目录                 |
| $     | 获取变量的前置符号             |
| &     | 工作控制：将命令变成背景下执行 |
| !     | 逻辑非                         |
| >, >> | 输出重定向                     |
| <, << | 输入重定向                     |
| ' '   | 单引号：不具有变量置换功能     |
| " "   | 双引号：具有变量置换功能       |
| \` \` | 重音号：优先执行 shell         |
| ()    | 括号：包含子 shell             |
| {}    | 花括号：组合 shell 语句块      |

## 资料流重定向

standard output（stdout）标准输出是指「命令执行成功所返回的信息」，standard error output（stderr）标准错误输出是指「命令执行失败所返回的信息」。不论是正确或者错误的信息，bash 预设都是输出到终端屏幕，可以通过输出重定向将 stdout 和 stderr 输出到其它档案或设备中。

standard input（stdin）标准输入是指「命令执行所需要读取的信息」。可以通过输入重定向切换 stdin 方式，例如键盘输入和文件输入。

- stdin：使用 < 或 <<，前者表示「将原本需要由键盘输入的资料，切换到由文件输入」，后者表示「输入结束（eof）」。
- stdout：使用 > 或 >>，前者表示以覆盖的方式输出「正确的资料」，后者表示以追加的方式输出「正确的资料」。
- stderr：使用 2> 或 2>>，前者表述以覆盖的方式输出「错误的资料」，后者表示以追加的方式输出「错误的资料」。

使用 `command 2> /dev/null` 忽略命令执行过程中的错误信息。

### 命令执行的判断依据

使用 `command; command` 执行多个命令。

使用 `command1 && command2` 在 command1 执行成功的情况下，执行 command2。

使用 `command1 || command2` 在 command1 执行失败的情况下，执行 command2。

## 管道命令

使用「管道符号（|）」，将前一个命令的 stdout 当作第二个命令的 stdin。例如 `ls -al | less` 命令以 `less` 命令查看 `ls -al` 的输出结果。

常用的管线命令：`cut`、`grep`、`sort`、`uniq`、`wc`、`tee`、`tr`、`col`、`join`、`paste`、`expand`、`split`、`xargs`。

### 减号的用途

使用「减号（-）」将前一个命令的 stdout 当作第二个命令的指定参数。例如 `ls -al ~ | grep '^-' -` 命令列出用户家目录下的所有文件。

## 参考资料

- [《鸟哥的 Linux 私房菜》 - 认识与学习 Bash](https://linux.vbird.org/linux_basic/centos7/0320bash.php)
