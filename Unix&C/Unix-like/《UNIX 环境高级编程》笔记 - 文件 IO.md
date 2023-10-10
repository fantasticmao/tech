# 《UNIX 环境高级编程》笔记 - 文件 IO

本章描述的函数经常被称为 **不带缓冲** 的 I/O（unbuffered I/O）。术语 **不带缓冲** 指的是每个 `read` 和 `write` 函数都会调用内核中的一个系统调用。

## 文件描述符

对于内核而言，所有打开的文件都通过 **文件描述符** 引用。文件描述符是一个非负整数，当打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。当读、写一个文件时，使用 `open` 或者 `create` 返回的文件描述符用于标识该文件，将其作为参数传给 `read` 或者 `write`。

UNIX 系统把文件描述符 0 与进程的标准输入关联，文件描述符 1 与标准输出关联，文件描述符 2 与标准错误关联。

在符合 POSIX 的应用程序中，魔法值 0、1、2 虽然已经被标准化，但应当把它们替换为符号常量 `STDIN_FILENO`、`STDOUT_FILENO`、`STDERR_FILENO` 以提高可读性。这些常量都在头文件 `<unistd.h>` 中定义。

## 函数 open 和 openat

调用 `open` 函数或 `openat` 函数可以打开或创建一个文件。

```c
#include <sys/stat.h>
#include <fcntl.h>

int open(const char *path, int oflag, ...);
int openat(int fd, const char *path, int oflag, ...);
```

path 参数是要打开或创建文件的名字，oflag 参数用来说明此函数的多个选项。用下列一个或多个常量进行「或」运算来构成 oflag 参数：

- `O_RDONLY` 只读打开；
- `O_WRONLY` 只写打开；
- `O_RDWR` 读写打开；
- `O_EXEC` 只执行打开；
- `O_APPEND` 每次 `write` 时都追加内容到文件的尾端；
- `O_CREAT` 若文件不存在，则会创建；
- `O_TRUNC` 如果文件存在，而且为只写或者读写打开时，则会将其长度截断为 0；
- `O_NONBLOCK` 当以只写或者读写打开 FIFO（进程间通信）时，或者当打开支持以非阻塞方式打开的块特殊文件和字符特殊文件时，后续的 I/O 操作将会被设置为非阻塞的；
- `O_SYNC` 每次 `write` 等待物理 I/O 操作完成，包括由该写操作引起的更新文件属性所需的 I/O；
- `O_DSYNC` 每次 `write` 等待物理 I/O 操作完成，忽略由该写操作引起的更新文件属性所需的 I/O；
- ……

`openat` 函数时 POSIX 最新版本中新增的函数，支持让线程可以使用相对路径名打开目录中的文件，而不再只能打开当前工作目录中的文件。

## 函数 creat

调用 `creat` 函数可以创建一个新的文件。

```c
#include <sys/stat.h>
#include <fcntl.h>

int creat(const char *path, mode_t mode);
```

`creat` 函数等同于调用 `open(path, O_WRONLY|O_CREAT|O_TRUNC, mode)`。

## 函数 close

调用 `close` 函数可以关闭一个打开的文件。

```c
#include <unistd.h>

int close(int fildes);
```

关闭一个文件时，还会释放该进程加在该文件上的所有记录锁。当一个进程终止时，内核自动关闭它所有的打开文件。

## 函数 lseek

调用 `lseek` 函数可以显式地为一个打开的文件设置偏移量。

```c
#include <unistd.h>

off_t lseek(int fildes, off_t offset, int whence);
```

每个打开的文件都有一个与其关联的「当前文件偏移量（current file offset）」。它通常是一个非负整数，用以度量文件开始处计算的字节数。通常，读、写操作都从当前文件偏移量处开始，并使偏移量增加所读写的字节数。

当打开一个文件时，除非指定 `O_APPEND` 选项，否则该偏移量被设置为 0。

如果文件描述符指向的是管道、FIFO、网络套接字，则 `lseek` 函数返回 -1，并将 errno 设置为 `ESPIPE`。

## 函数 read

调用 `read` 函数可以从打开的文件中读取数据。

```c
#include <unistd.h>

ssize_t pread(int fildes, void *buf, size_t nbyte, off_t offset);
ssize_t read(int fildes, void *buf, size_t nbyte);
```

如果 `read` 函数执行成功，则返回实际读取的字节数，如果已到达文件的尾端，则返回 0。

## 函数 write

调用 `write` 函数可以向打开的文件中写入数据。

```c
#include <unistd.h>

ssize_t pwrite(int fildes, const void *buf, size_t nbyte,
           off_t offset);
ssize_t write(int fildes, const void *buf, size_t nbyte);
```

## I/O 的效率

大多数文件系统为改善性能，都会采用某种预读（read ahead）技术。当检测到正进行顺序读取时，系统就会试图读入比应用所需的更多数据，并假设应用很快就会读取到这些数据。

## 原子操作

### 追加文件内容

早期版本的 UNIX 系统不支持 `open` 函数的 `O_APPEND` 选项，因此追加文件内容的逻辑会被写成：

```c
if (lseek(fd, OL, 2) < 0) {
  err_sys("lseek error");
}
if (write(fd, buf, 100) != 100) {
  err_sys("write_error");
}
```

被拆分成两个函数调用的「先定位到文件尾端，然后写入内容」逻辑代码是非原子性的，因为在两个函数调用之间，内核有可能会临时挂起进程。

UNIX 系统为这样的操作提供了一种保证原子性的方法，即在打开文件时设置 `O_APPEND` 选项。

### 函数 pread 和 pwrite

调用 `pread` 函数和 `pwrite` 函数可以原子性地定位并执行 I/O 操作。

```c
#include <unistd.h>

ssize_t pread(int fildes, void *buf, size_t nbyte, off_t offset);
ssize_t pwrite(int fildes, const void *buf, size_t nbyte,
           off_t offset);
```

调用 `pread` 函数或 `pwrite` 函数相当于先调用 `lseek` 函数再调用 `read` 函数或 `write` 函数，但又有一些区别：

- 调用 `pread` 函数或 `pwrite` 函数时，无非中断其定位和读写操作；
- 不会更新当前文件的偏移量。

### 创建一个文件

`open` 函数的 `O_CREAT` 选项支持原子性的「检查文件是否存在，当文件不存在时则创建文件」的逻辑操作。

## 函数 dup 和 dup2

调用 `dup` 函数或 `dup2` 函数可以复制一个现有的文件描述符。

```c
#include <unistd.h>

int dup(int fildes);
int dup2(int fildes, int fildes2);
```

## 函数 sync、fsync 和 fdatasync

传统的 UNIX 系统实现，在内核中设有缓冲区高速缓存或页高速缓存，大多数磁盘 I/O 都通过缓冲区进行。当我们向文件写入数据时，内核通常先将数据复制到缓冲区中，然后排入队列，晚些时候再写入磁盘。这种方式被称为「延迟写（delayed write）」。

通常，当内核需要重用缓冲区来存放其它磁盘的数据块时，它会把所有延迟写的数据块写入磁盘。为了保证磁盘上实际文件系统与缓冲区中的内容一致，UNIX 系统提供了 `sync`、`fsync`、`fdatasync` 三个函数。

```c
#include <unistd.h>

void sync(void);
int fsync(int fildes);
int fdatasync(int fildes);
```

`sync` 函数只是将所有修改过的在缓冲区中的数据块排入写队列，然后就返回，它并不等待实际写磁盘的操作结束。通常，称为 update 的系统守护进程会周期性地调用（一般间隔 30s）`sync` 函数。

`fsync` 函数只对由文件描述符 fd 指定的一个文件起作用，并且等待写磁盘的操作结束时才返回。`fsync` 可用于数据库这样的应用程序，这类应用程序需要确保修改过的数据块立即写到磁盘上。

`fdatasync` 函数类似于 `fsync` 函数，但它只影响文件的数据部分。而 `fsync` 函数除了对数据部分，还会同步更新文件的属性。

## 函数 fcntl

调用 `fcntl` 函数可以修改已经打开文件描述符/文件状态的标志。

```c
#include <fcntl.h>

int fcntl(int fildes, int cmd, ...);
```

`fcntl` 函数支持以下 5 种功能：

1. 复制一个已有的描述符；
2. 获取/设置文件描述符的标志；
3. 获取/设置文件状态的标志；
4. 获取/设置异步 I/O 所有权；
5. 获取/设置记录锁。

在 UNIX 系统中，通常 write 只是将数据排入队列，而实际的写磁盘操作则可能在之后的某个时刻进行。数据库系统则会需要使用 `O_SYNC` 选项，当它从调用 `write` 函数返回时就可以确保数据已经的确写入到磁盘上了，以系统在异常时导致数据丢失。

程序在运行时，设置 `O_SYNC` 标志会增加系统时间和时钟时间，其原因是内核需要从进程中复制数据，并将数据排入队列以便由磁盘驱动器将其写到磁盘上。当使用同步写入时，系统时间和时钟时间便会显著增加。

## 函数 ioctl

`ioctl` 函数一直是 I/O 操作的杂物箱，不能用本章中其它函数表示的 I/O 操作通常都能用 `ioctl` 函数来完成。终端 I/O （例如树莓派的 GPIO）是使用 `ioctl` 最多的地方。

```c
#include <stropts.h>

int ioctl(int fildes, int request, ... /* arg */);
```
