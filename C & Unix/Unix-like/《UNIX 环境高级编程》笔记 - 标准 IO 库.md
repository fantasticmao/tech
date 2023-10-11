# 《UNIX 环境高级编程》笔记 - 标准 IO 库

不仅是 UNIX，很多其它操作系统都实现了标准 I/O 库，这个库是由 ISO C 标准定义的。

## 流和 FILE 对象

文件 I/O 函数都是围绕着文件描述符进行的。当打开一个文件时，即返回一个文件描述符，然后该文件描述符用于后续的 I/O 操作。

对于标准 I/O 库而言，它们的操作是围绕流（stream）进行的。当用标准 I/O 库打开或者创建一个文件时，我们使用一个流与一个文件相关联。

对于 ASCII 字符集，一个字符用一个字节表示。对于国际字符集（例如 UTF-8），一个字符可用多个字节（宽字节）表示。标准 I/O 文件流即可以用于单字节的，也可用于多字节的字符集。流的方向（orientation of stream）决定了它所读写的字符是单字节还是多字节的。

调用 `fwide` 函数可以设置流的方向。需要注意的是，`fwide` 函数并不会改变已经定向的流的方向。

```c
#include <wchar.h>

int fwide(FILE *stream, int mode);
```

当打开一个流时，标准 I/O 函数 `fopen` 返回一个指向 FILE 对象的指针。FILE 对象通常是一个结构，它包含了标准 I/O 库为管理该流需要的所有信息，包括用于实际 I/O 的文件描述符、指向用于该流缓冲区的指针、缓冲区的长度、当前在缓冲区中的字符数以及出错标志。

## 标准输入、标准输出和标准错误

每个进程都预定义了 3 个流，并且这 3 个流可以自动地被进程所使用，它们是：标准输入、标准输出、标准错误。

这 3 个标准 I/O 流通过预定义的文件指针 stdin、stdout、stderr 加以引用，文件指针定义在头文件 `<stdio.h>` 中。

## 缓冲

标准 I/O 库提供缓冲的目的是尽可能减少使用 `read` 函数和 `write` 函数的调用次数。标准 I/O 库提供了以下 3 种类型的缓冲：

- 全缓冲：标准 I/O 库将会在缓冲区填满之后，再进行实际的 I/O 操作。术语 flush 表示标准 I/O 缓冲区的写操作；
- 行缓冲：标准 I/O 库将会在输入和输出过程中遇到换行符时，再进行实际的 I/O 操作。
- 不带缓冲：标准 I/O 库不会对字符进行缓冲。标准错误流 stderr 通常是不带缓冲的。

调用 `setbuf` 函数或 `setlinebuf` 函数可以更改流的缓冲类型。

```c
#include <stdio.h>

int setvbuf(FILE *restrict stream, char *restrict buf,
            int mode, size_t size);

void setbuf(FILE *restrict stream, char *restrict buf);
void setbuffer(FILE *restrict stream, char *restrict buf,
            size_t size);
void setlinebuf(FILE *stream);
```

## 打开流

调用 `fopen` 函数、`fdopen` 函数或 `freopen` 函数可以打开一个标准 I/O 流。

```c
#include <stdio.h>

FILE *fopen(const char *restrict pathname, const char *restrict mode);
FILE *fdopen(int fd, const char *mode);
FILE *freopen(const char *restrict pathname, const char *restrict mode,
              FILE *restrict stream);
```

`fopen` 函数打开指定的路径名为 pathname 的文件。

`fdopen` 函数打开指定的文件描述符 fd 上的文件。这个函数通常用于创建管道函数和网络通信函数返回的文件描述符。因为这些特殊类型的文件不能用标准 I/O 库中的 `fopen` 函数打开，因此必须先调用设备专用函数用以获取一个文件描述符，然后用 `fdopen` 函数将标准 I/O 流与该文件描述符相关联。

`freopen` 函数打开指定的路径名为 pathname 的文件，并且关联到指定的流 stream 上。如果指定的流 stream 已经打开，则会先关闭它。

`fopen` 函数和 `freopen` 函数是 ISO C 的一部分，因为 ISO C 并不涉及文件描述符，因此只有 POSIX 才有 `fdopen` 函数。

mode 参数指定对该 I/O 流的读写方式，ISO C 规定 mode 参数可以用 15 种不同的值。

| mode             | 说明                                                 | 对应的 open(2) 标志             |
| ---------------- | ---------------------------------------------------- | ------------------------------- |
| r 或 rb          | 只读打开                                             | O_RDONLY                        |
| r+ 或 r+b 或 rb+ | 读写打开                                             | O_RDWR                          |
| w 或 wb          | 只写打开，当文件不存在时则会创建，将文件长度截断为 0 | O_WRONLY \| O_CREAT \| O_TRUNC  |
| w+ 或 w+b 或 wb+ | 读写打开，将文件长度截断为 0                         | O_RDWR \| O_CREAT \| O_TRUNC    |
| a 或 ab          | 只写打开，当文件不存在时则会创建，追加内容           | O_WRONLY \| O_CREAT \| O_APPEND |
| a+ 或 a+b 或 ab+ | 读写打开，追加内容                                   | O_RDWR \| O_CREAT \| O_APPEND   |

## 读和写流

一旦打开了流，则可以使用 3 种不同类型的非格式化 I/O 对其进行读、写操作：

1. 每次只操作一个字符的 I/O：`fgetc` 函数和 `fputc` 函数在每次执行 I/O 操作时，只会读写一个字符；
2. 每次操作一行字符的 I/O：`fgets` 函数和 `fputs` 函数在每次执行 I/O 操作时，会以换行符为标记，读取一行的内容；
3. 直接 I/O（二进制 I/O）：`fread` 函数和 `fwrite` 函数在每次执行 I/O 操作时，会读写特定数量的对象，每个对象具有指定的长度。这两个函数通常用于从二进制文件中读写一个结构。

### 输入函数

调用 `fgetc` 函数、`getc` 函数或 `getchar` 函数可以读取一个字符。

```c
#include <stdio.h>

int fgetc(FILE *stream);
int getc(FILE *stream);
int getchar(void);

int ungetc(int c, FILE *stream);
```

`fgetc` 函数可以作为地址，被当作参数传递给另一个函数。

`getc` 函数等同于 `fgetc` 函数，它可能是通过宏来实现的。

`getchar` 函数等同于 `getc(stdin)` 函数调用。

`fgetc` 函数、`getc` 函数和 `getchar` 函数在返回下一个字符时，会将 unsigned char 类型转换为 int 类型。

### 判断是出错还是达到文件尾端

不管是出错还是达到文件尾端，这 3 个函数都会返回同样的值。为了区分这两种不同的场景，必须调用 `ferror` 函数或 `feof` 函数。

```c
#include <stdio.h>

void clearerr(FILE *stream);
int feof(FILE *stream);
int ferror(FILE *stream);
```

### 压送字符回到流中

从流中读取数据之后，可以调用 `ungetc` 函数将字符再压送回到流中。用 `ungetc` 函数压送回字符时，并没有真正将它们写到底层文件或者设备中，只是将它们写回到标准 I/O 库的缓冲区中。

```c
#include <stdio.h>

int ungetc(int c, FILE *stream);
```

压送回到流中的字符之后可以再从流中被读取，但读取字符的顺序与压送回的字符顺序相反。虽然 ISO C 允许实现库支持任意次数的将字符压送回到流中的操作，但是它要求实现库一次只能压送一个字符。压送回到流中的字符不一定是上次读取的字符，但不能压送 EOF 字符。

当正在读取一个输入流，并进行某种形式的分词或者记号切分操作时，会经常用到 `ungetc` 函数。

### 输出函数

调用 `fputc` 函数、`putc` 函数或 `putchar` 函数可以写入一个字符。

```c
#include <stdio.h>

int fputc(int c, FILE *stream);
int putc(int c, FILE *stream);
int putchar(int c);
```

## 每次一行 I/O

调用 `fgets` 函数可以读取一行字符。

```c
#include <stdio.h>

char *fgets(char *restrict s, int size, FILE *restrict stream);
```

调用 `fputs` 函数和 `puts` 函数可以写入一行字符。

```c
#include <stdio.h>

int fputs(const char *restrict s, FILE *restrict stream);
int puts(const char *s);
```

## 二进制 I/O

调用 `fread` 函数和 `fwrite` 函数可以读写一个完整的结构。

```c
#include <stdio.h>

size_t fread(void *restrict ptr, size_t size, size_t nmemb,
             FILE *restrict stream);
size_t fwrite(const void *restrict ptr, size_t size, size_t nmemb,
             FILE *restrict stream);
```

`fread` 函数和 `fwrite` 函数通常被用于：读写一个二进制数组、读写一个结构。

## 定位流

调用 `fseek` 函数、`ftell` 函数、`rewind` 函数、`fgetpos` 函数和 `fsetpos` 函数可以查询和设置流的偏移量。

```c
#include <stdio.h>

int fseek(FILE *stream, long offset, int whence);
long ftell(FILE *stream);

void rewind(FILE *stream);

int fgetpos(FILE *restrict stream, fpos_t *restrict pos);
int fsetpos(FILE *stream, const fpos_t *pos);
```

对于一个二进制文件，其文件位置指示器是从文件起始位置开始度量，并以字节为度量单位。但对于一个文本文件，它的文件当前位置可能不能简单地以字节偏移量来度量。

`ftell` 函数可以获取流的偏移量，`fseek` 函数可以设置流的偏移量。`rewind` 函数可以重置流的偏移量为文件的起始位置。

`fgetpos` 函数和 `fsetpos` 函数与 `ftell` 函数和 `fseek` 函数类似，不过它们是 ISO C 标准引入的。

`ftello` 函数和 `fseeko` 函数支持以 off_t 类型来设置流的偏移量。

```c
#include <stdio.h>

int fseeko(FILE *stream, off_t offset, int whence);
off_t ftello(FILE *stream);
```

## 格式化 I/O

### 格式化输出

调用 `printf` 函数、`fprintf` 函数、`dprintf` 函数、`sprintf` 函数和 `snprintf` 函数可以以格式化的方式输出 I/O 流。

```c
#include <stdio.h>

int printf(const char *restrict format, ...);
int fprintf(FILE *restrict stream,
            const char *restrict format, ...);
int dprintf(int fd,
            const char *restrict format, ...);
int sprintf(char *restrict str,
            const char *restrict format, ...);
int snprintf(char *restrict str, size_t size,
            const char *restrict format, ...);
```

`vprintf` 函数、`vfprintf` 函数、`vdprintf` 函数、`vsprintf` 函数和 `vsnprintf` 函数也可以以格式化的方式输出 I/O 流，不过它们的可变参数列表变成了 vs_list 类型。

```c
#include <stdarg.h>

int vprintf(const char *restrict format, va_list ap);
int vfprintf(FILE *restrict stream,
            const char *restrict format, va_list ap);
int vdprintf(int fd,
            const char *restrict format, va_list ap);
int vsprintf(char *restrict str,
            const char *restrict format, va_list ap);
int vsnprintf(char *restrict str, size_t size,
            const char *restrict format, va_list ap);
```

### 格式化输入

调用 `scanf` 函数、`fscanf` 函数和 `sscanf` 函数可以以格式化的方式输入 I/O 流。

```c
#include <stdio.h>

int scanf(const char *restrict format, ...);
int fscanf(FILE *restrict stream,
           const char *restrict format, ...);
int sscanf(const char *restrict str,
           const char *restrict format, ...);
```

`vscanf` 函数、`vfscanf` 函数和 `vsscanf` 函数也可以以格式化的方式输入 I/O 流，不过它们的可变参数列表变成了 vs_list 类型。

```c
#include <stdarg.h>

int vscanf(const char *restrict format, va_list ap);
int vfscanf(FILE *restrict stream,
           const char *restrict format, va_list ap);
int vsscanf(const char *restrict str,
           const char *restrict format, va_list ap);
```

## 实现细节

在 UNIX 中，标准 I/O 库最终都要调用文件 I/O 中的函数。每个标准 I/O 流中都有一个与其相关联的文件描述符。

调用 `fileno` 函数可以获取与流相关的文件描述符。

```c
#include <stdio.h>

int fileno(FILE *stream);
```

## 临时文件

调用 `tmpnam` 函数或 `tmpfile` 函数可以创建一个临时文件。

```c
#include <stdio.h>

char *tmpnam(char *s);
char *tmpnam_r(char *s);

FILE *tmpfile(void);
```

## 内存流

调用 `fmemopen` 函数、`open_memstream` 函数或 `open_wmemstream` 函数可以创建一个内存流。

```c
#include <stdio.h>

FILE *fmemopen(void *buf, size_t size, const char *mode);

FILE *open_memstream(char **ptr, size_t *sizeloc);
```

```c
#include <wchar.h>

FILE *open_wmemstream(wchar_t **ptr, size_t *sizeloc);
```

`fmemopen` 函数允许调用者将提供的缓冲区用于内存流。

`open_memstream` 函数可以创建面向字节的内存流， `open_wmemstream` 函数可以创建面向宽字节的内存流。

## 标准 I/O 库的替代软件

标准 I/O 库的一个不足之处是效率不高，这与它需要复制的数据量有关系。当使用每次读写一行的 `fgets` 函数和 `fputs` 函数时，通常需要复制两次数据：第一次是在内核和标准 I/O 库缓冲区之间，第二次是在标准 I/O 缓冲区和用户程序中之间。（源码位于 [iofgets.c](https://sourceware.org/git/?p=glibc.git;a=blob;f=libio/iofgets.c;h=68774aaefd67ec39acf2abc6a4ca21f23cf3de36;hb=refs/heads/release/2.30/master) 和 [iogetline.c](https://sourceware.org/git/?p=glibc.git;a=blob;f=libio/iogetline.c;h=ad1e64b1848ecb87c5ec9b3f60a51e82ecf84f8b;hb=refs/heads/release/2.30/master) 两个源文件中。）

可替代标准 I/O 库的软件包有：

- fio：快速 I/O 库；
- sfio 库；
- ASI（Alloc Stream Interface）：提供了 `mmap` 函数；
- uClibc C 库和 Newlib C 库：适用于内存较小的系统，例如嵌入式系统。
