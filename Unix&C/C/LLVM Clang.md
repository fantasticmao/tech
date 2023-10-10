# LLVM Clang

## Clang

### 重要参数

- `-E` 只运行预编译阶段，约定后缀名为 `.i`；
- `-S` 只运行预编译 -> 编译阶段，约定后缀名为 `.s`；
- `-c` 只运行预编译 -> 编译 -> 汇编阶段，约定后缀名为 `.o`、`.obj`；
- `-o <file>` 写入输出至文件 `<file>` 中；
- `-g` 生成源码级别的 debug 信息；
- `-D <macro>=<value>` 定义宏 `<macro>` 为 `<value>`；
- `-U <macro>` 取消定义宏 `<macro>`。

## LLDB

### core 文件

`ulimit -c 1024` 允许程序在崩溃时生成 core 文件。
