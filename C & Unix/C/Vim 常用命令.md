# Vim 常用命令

在本文中，blank 字符指空格符、制表符、回车符、换行符等等，`$` 表示美元符号，`${}` 表示占位符。

## Normal 模式

### 移动光标

|命令     |描述|
| --- |--- |
| h、← | 向左移动光标 |
| j、↓ | 向下移动光标 |
| k、↑ | 向上移动光标 |
| l、→ | 向右移动光标 |
| Ctrl + b、PgUp | 向上翻页 |
| Ctrl + f、PgDn | 向下翻页 |
| Ctrl + u | 向上翻半页 |
| Ctrl + d | 向下翻半页 |
| 0、Home | 移动光标至当前行的第一个位置 |
| \$、End | 移动光标至当前行的最后一个位置 |
| H | 移动光标至当前页的第一列的第一个字符位置 |
| M | 移动光标至当前页的中间列的第一个字符位置 |
| L | 移动光标至当前页的最后一列的第一个字符位置 |
| ^ | 移动光标至当前行的第一个非 blank 字符位置 |
| g\_ | 移动光标至当前行的最后一个非 blank 字符位置 |
| \${n}G | 相当于 `:${n}` |
| gg | 相当于 1G 和 `:${1}` |
| G | 移动光标至当前文件的最后一行 |
| ${n} + Space | 向右移动光标至之后 n 个字符的位置 |
| f\${char} | 向右移动光标至下个 char 字符的位置 |
| F\${char} | 向左移动光标至上个 char 字符的位置 |
| t\${char} | 向右移动光标至下个 char 字符的前一个位置 |
| T\${char} | 向左移动光标至上个 char 字符的前一个位置 |

### 复制、粘贴、删除

| 命令 | 描述 |
| ---- | ---- |
| yy | 复制当前行 |
| y + [移动光标](#移动光标) | 复制光标移动过程中的内容 |
| [选中内容](#选中内容) + y | 复制光标选中的内容 |
| p | 执行 a / 向下，再粘贴剪贴板内容 |
| P | 执行 i / 向上，再粘贴剪贴板内容 |
| x | 向右删除一个字符，相当于插入模式中的 fn + Delete |
| X | 向左删除一个字符，相当于插入模式中的 Delete |
| dd | 删除当前行，**并把删除的内容保存至剪贴板中**，相当于「剪切」功能 |
| d + [移动光标](#移动光标) | 删除光标移动过程中的内容 |
| [选中内容](#选中内容) + d | 删除光标选中的内容 |

### 搜索

| 命令 | 描述 |
| ---- | ---- |
| / \${word} | 向下搜索 word 关键字 |
| ? \${word} | 向上搜索 word 关键字 |
| n | 重复前一个搜索动作 |
| N | 反向重复前一个搜索动作 |

### 普通命令

| 命令 | 描述 |
| ---- | ---- |
| = | 缩进当前行 |
| [选中内容](#选中内容) + = | 缩进光标选中的内容 |
| guu | 转换当前行为小写 |
| [选中内容](#选中内容) + gu | 转换选中内容为小写 |
| gUU | 转换当前行为大写 |
| [选中内容](#选中内容) + gU | 转换选中内容为大写 |
| u | undo，撤销上一次操作 |
| Ctrl + r | redo，撤销 u 操作 |
| . | 重复执行上一次命令 |
| \${n}`command` | 重复执行 n 次 `command` 命令 |
| **ZZ**、:wq | 相当于 `:wq` |

## Visual 模式

### 选中内容

| 命令 | 描述 |
| ---- | ---- |
| v | 选择字 |
| V | 选择行 |
| Ctrl + v | 选择块 |
| **Ctrl + z** | 挂起 Vim（相当于执行 `bg` 命令，使用 `fg` 命令返回 Vim） |

## Insert 模式

| 命令 | 描述 |
| ---- | ---- |
| i | 从光标所在的位置进入 INSERT 模式 |
| I | 从光标所在的第一个非 blank 字符位置进入 INSERT 模式 |
| a | 从光标所在的下一个位置进入 INSERT 模式 |
| A | 从光标所在的当前行的最后一个位置进入 INSERT 模式 |
| o | 从光标所在的下一行位置进入 INSERT 模式 |
| O | 从光标所在的上一行位置进入 INSERT 模式 |
| r | 替换光标所在的位置的第一个字符，并进入 INSERT 模式 |
| R | 从光标所在的位置进入 REPLACE 模式 |
| Esc | 退出 Insert 模式，回到 Normal 模式 |
| Ctrl + n | 开启自动补全 |
| Ctrl + p | 开启自动补全，并跳至最后一个选项 |

## Command-line 模式

| 命令 | 描述 |
| ---- | ---- |
| :\${n} | 移动光标至当前文件的第 n 行 |
| :! + \${shell} | 执行一条 `shell` 命令 |
| :sh、:shell | 挂起 Vim，执行多条 `shell` 命令（退出当前 `shell` 即返回 vim） |
| :pwd | 打印当前目录 |
| :cd \${dir} | 改变当前目录 |
| :e \${fileName} | 打开新的文件 |
| :r \${fileName} | 读取文件内容至当前文件中 |
| :r! \${shell} | 读取 `shell` 命令的输出内容至当前文件中 |
| :w | 保存当前文件内容 |
| :w \${fileName} | 保存当前文件内容为新的文件，类似于「文件另存为」功能 |
| :q | 退出 Vim |
| :q! | 强制退出 Vim |
| :wq、ZZ | 保存并退出 Vim |
| :n | 编辑下一个文件 |
| :N | 编辑上一个文件 |
| :files | 列出打开的所有文件 |
| :sp、:split | 创建水平分屏 |
| :vsp、:vsplit | 创建垂直分屏 |
| :Ctrl + w + [h, j, k, l] | 在分屏窗口中移动光标 |
| :Ctrl + w + [H, J, K, L] | 移动分屏窗口 |

## 参考资料

- Vim :help —— VIM main help file
- [Vim (text editor)](https://en.wikipedia.org/wiki/Vim_(text_editor)) —— Wikipedia
- [Vim 程式編輯器](http://linux.vbird.org/linux_basic/0310vi.php) —— 鳥哥的 Linux 私房菜
- [Vim 的分屏功能](https://coolshell.cn/articles/1679.html) —— 酷 壳 - CoolShell
- [简明 Vim 练级攻略](https://coolshell.cn/articles/5426.html) —— 酷 壳 - CoolShell
- [无插件 Vim 编程技巧](https://coolshell.cn/articles/11312.html) —— 酷 壳 - CoolShell
