# 《鳥哥的 Linux 私房菜》笔记 - 任务管理

## 什么是任务管理

在 bash 中执行任务管理（job control）需要注意的限制条件是：

1. 这些任务必须是当前 shell 进程的子进程；
2. 前景：用户可以控制与下达指令的环境被称为前景；
3. 背景：可以自己运行的任务，用户无法用 `Ctrl + c` 来终止，但是可以使用 `fg` 和 `bg` 来切换任务；
4. 「背景」中执行的任务不能等待 terminal 和 shell 的输入。

## job control 的管理

bash 只能管理自己的任务，而不能管理其它 bash 的任务，即使是 root 用户也不能管理其它的 bash 底下的任务。

用户可以控制与下达指令的环境被称为「前景」，反之则是「背景」。在背景中的任务的执行状态分为暂停（Stopped）和运行中（Running）。

### 将待执行的任务丢到背景中执行：`&`

在只有一个 bash 的环境下，如果想要同时进行多个工作，那么可以将某些任务直接丢到「背景」中执行。使用 `&` 可以达成这个目的。

```bash
host:~$ sleep 5 &
[1]  1234             # 1 表示 job number，1234 表示进程 pid
host:~$
[1]-  Done  sleep 5   # 1 表示 job number
```

主要注意的是，在「背景」中执行的任务的 stdout 和 stderr 是依旧显示在屏幕上的，解决办法是使用「重定向」。

### 将当前的任务丢到背景中执行：`Ctrl + z`

假设用户正在 Vim 中编辑文件时，如果需要回到 bash 来执行命令的话，可以使用 `Ctrl + z` 命令。

```bash
host:~$ vim ~/.bashrc        # 按下 Ctrl + z
[1]+  Stopped  vim .bashrc
host:~$                      # 回到 bash 来执行命令
```

### 查看背景中的任务的运行状态：`jobs`

`jobs` 命令可以查询当前「背景」当中有多少个任务。

```bash
host:~$ jobs
[1]-  Stopped  vim .bashrc   # - 表示最近最后第二个被放入背景中的任务
[2]+  Stopped  vim .vimrc    # + 表示最近被放入背景中的任务，
```

### 将背景中的任务切换到前景中执行：`fg`

`fg` 命令（fg 是 foreground 的缩写）可以将背景中的任务切换到前景中执行。

```bash
host:~$ fg %1   # 重新进入 Vim，符号 % 是可选的
```

### 将背景中的任务切换到执行状态：`bg`

使用 `Ctrl + z` 命令放入「背景」中执行的任务，默认是暂停状态，可以使用 `bg` 命令（bg 是 background 的缩写）将它切换为运行中状态。

```bash
host:~$ git clone https://github.com/xxx/xxx           # 按下 Ctrl + z
[1]+  Stopped   git clone https://github.com/xxx/xxx
host:~$ bg %1                                          # 运行 git clone 命令，符号 % 是可选的
```

### 管理背景中的人物：kill

`kill` 命令可以通过信号来中断、终止、重启任务，具体的信号可以参考 `kill -l` 和 `man 7 signal`。

```bash
host:~$ kill -15 %1        # 符号 % 是必选的
host:~$ kill -SIGTERM %1   # 符号 % 是必选的
```

## 离线管理

在任务管理中提及的「背景」是指在 terminal 模式下可以避免被 `ctrl + c` 中断的一种场景，但是任务还是属于当前 shell 进程的子进程，不能将这种行为理解成把任务放入到了操作系统的背景中来执行。因此任务管理的「背景」中的任务依旧是会与 terminal 有所关联。

假设用户是以 `ssh` 命令连接至远程的 Linux 主机，并且将任务以 `&` 的方式执行，那么在用户退出 `ssh` 命令的时候，该任务也会因为中断而停止。

`nohup` 命令可以使任务在当用户退出之后还能继续执行，可以搭配使用 `nohup` 和 `&` 命令来执行需要长期在后台运行的任务。

```bash
host:~$ nohup [命令和参数]     # 在 terminal 前景中执行
host:~$ nohup [命令和参数] &   # 在 terminal 背景中执行
```
