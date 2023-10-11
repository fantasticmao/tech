# 《A Tour of Go》笔记

## 程序结构

### 包

按照约定，包名与包路径的最后一个元素相同。例如 `math/rand` 包中的源码均以 `package rand` 开始。

可以使用括号来对 `import` 语句进行分组，对 `import` 语句进行分组是一种好的风格。

在 Go 中，没有 `public`、`protected`、`private` 关键字。若（变量、函数......）的 **名称是以大写字母开头，则表示它们是 exported 的，即可以被外部包中的代码所访问**。

### 函数

函数可以接收零个或多个参数。Go 的 **类型是位于名称之后** 的，至于为什么这么设计，可以看 [Go's Declaration Syntax](https://go.dev/blog/declaration-syntax)。

```go
func add(x int, y int) int {
	return x + y
}
```

当函数中有两个或者更多连续相同类型的形式参数，则可以省略除最后一个参数之外的所有参数的类型声明。

```go
func add(x, y int) int {
	return x + y
}
```

**函数可以返回任意数量的结果**。

函数的返回值可以被命名，它们会被视为在函数顶部定义的变量。

### 变量

`var` 关键字可以声明一个变量列表，变量的类型也是位于名称之后的。变量声明语句可以位于包级别和函数级别。

变量的声明可以包含初始化值，每个变量对应一个值。变量的声明也可以使用括号来进行分组。

变量的声明中若包含初始化值，则可以省略变量的类型定义。**该变量的类型会从初始化值中推导得出**。

在函数内部，`:=` 赋值语句可以替换隐式类型声明的变量声明语句。在函数外部，每个语句都必须以 Go 关键字开始（`var`、`func`......），所以不能使用 `:=` 语法。

Go 的基础类型有：

```plain text
bool

string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // alias for uint8

rune // alias for int32
     // represents a Unicode code point

float32 float64

complex64 complex128
```

没有被明确初始化的变量，会被赋予。数值类型变量的零值是 0，布尔类型变量的零值是 false，字符串类型变量的零值是空字符串。

表达式 `T(v)` 用于将值 v 转换成类型 T。

在变量初始化的类型推导中，当右侧是没有类型的数值常量时，声明的变量可能会是 `int`、`float64` 或 `complex128`，这取决于数值常量的精度。

```go
i := 42           // int
f := 3.142        // float64
g := 0.867 + 0.5i // complex128
```

`const` 关键字可以声明常量，常量可以是字符、字符串、布尔值、数值。常量不可以使用 `:=` 语法赋值。

## 控制流程

### for

Go 中只有 `for` 语句一种循环结构。Go 的 `for` 语句由逗号分隔的三个部分组成：初始化语句、条件语句、后置语句，它们不需要由括号包裹，并且都是可以忽略的。

```go
func main() {
	sum := 0
	for i := 0; i < 10; i++ {
		sum += i
	}
	fmt.Println(sum)
}
```

```go
func main() {
	sum := 1
	for ; sum < 1000; {
		sum += sum
	}
	fmt.Println(sum)
}
```

在 `for` 语句中，当忽略初始化语句和后置语句时，也可以忽略 `;` 符号，这便是 Go 中的 `where` 语句。

```go
func main() {
	sum := 1
	for sum < 1000 {
		sum += sum
	}
	fmt.Println(sum)
}
```

在 `for` 语句中，当连条件语句也被忽略时，则表示无限循环。

### if-else

`if` 语句和 `for` 语句类似，判断条件也不需要由括号包裹。

`if` 语句和 `for` 语句类似，也可以在判断条件之前，执行一个只在当前作用域内有效的变量初始化语句。

在 `if` 语句中初始化的变量，在 `else` 语句中也是有效的。

### switch

**`switch` 语句只会执行匹配成功的 `case` 语句，而不是其后的所有 `case` 语句**。实际上，Go 会在每个 `case` 语句末尾自动添加 `break` 语句。

`switch` 语句和 `case` 语句的条件语句不需要是常量，它们所涉及的值也不需要是整数。

```go
func main() {
	fmt.Print("Go runs on ")
	switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("OS X.")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.\n", os)
	}
}
```

**`switch` 语句由上而下执行，在匹配成功时便会结束**。在下例中，当 `i==0` 时，`f()` 是不会被调用的。

```go
switch i {
	case 0:
	case f():
}
```

不带条件语句的 `switch` 语句等价于 `switch true`，这种写法是另一种简洁的 `if-then-else` 语句。

```go
func main() {
	t := time.Now()
	switch {
	case t.Hour() < 12:
		fmt.Println("Good morning!")
	case t.Hour() < 17:
		fmt.Println("Good afternoon.")
	default:
		fmt.Println("Good evening.")
	}
}
```

### defer

`defer` 语句可以将待执行函数，推迟到当前函数返回之后再执行，但待执行函数的参数是会被立即执行的。

```go
func main() {
	defer fmt.Println("world")

	fmt.Println("hello")
}
```

`defer` 语句的待执行函数会被压入到一个栈中，在当前函数返回之后，**会按照 Last-In-First-Out 的顺序被执行**。

```go
func main() {
	fmt.Println("counting")

	for i := 0; i < 10; i++ {
		defer fmt.Println(i)
	}

	fmt.Println("done")
}
```

## 数据结构

### pointer

Go 中也是有指针的。指针保存的是值的内存地址。

类型 `*T` 表示一个指向类型 `T` 的指针，指针的零值是 `nil`。

`&` 运算符用于生成一个指向其操作数的指针。`*` 运算符用于获取指针指向的值。

```go
func main() {
	i, j := 42, 2701

	p := &i         // point to i
	fmt.Println(*p) // read i through the pointer
	*p = 21         // set i through the pointer
	fmt.Println(i)  // see the new value of i

	p = &j         // point to j
	*p = *p / 37   // divide j through the pointer
	fmt.Println(j) // see the new value of j
}
```

与 C 语言不同，**Go 中是没有指针运算的**。

### structs

结构（struct）是一组字段的集合，使用 `.` 符号来访问结构的字段。

```go
type Vertex struct {
	X int
	Y int
}

func main() {
	v := Vertex{1, 2}
	v.X = 4
	fmt.Println(v.X)
}
```

可以通过指向结构的指针来访问结构的字段，Go 中 `(*p).X` 的简写方式为 `p.X`，对应了 C 语言中的 `p->X`。

结构字面量表示通过列出字段值的方式，来分配的一个新结构。可以使用 `Name:` 的语法来仅列出字段的子集。

```go
var (
	v1 = Vertex{1, 2}  // has type Vertex
	v2 = Vertex{X: 1}  // Y:0 is implicit
	v3 = Vertex{}      // X:0 and Y:0
	p  = &Vertex{1, 2} // has type *Vertex
)
```

### array

类型 `[n]T` 表示一个由 n 个 T 类型的值组成的数组（Array）。数组的长度是其类型的一部分，**数组的长度不可以被调整**。

```go
func main() {
	var a [2]string
	a[0] = "Hello"
	a[1] = "World"
	fmt.Println(a[0], a[1])
	fmt.Println(a)

	primes := [6]int{2, 3, 5, 7, 11, 13}
	fmt.Println(primes)
}
```

### slice

类型 `[]T` 表示一个由 T 类型的值组成的切片（slice）。**切片的长度可以被动态调整**。

切片是通过指定一对上下限的数组下标，并以冒号分隔而组成的。**切片会包含上限对应的元素，但不会包含下限对应的元素**。

```go
func main() {
	primes := [6]int{2, 3, 5, 7, 11, 13}

	var s []int = primes[1:4]
	fmt.Println(s)
}
```

切片不会存储任何数据，它只是展示了底层数组的一部分数据。**修改切片的元素，也会修改底层数组的元素，其它共享底层数组的切片也会看到数据元素的变更**。

```go
func main() {
	names := [4]string{
		"John",
		"Paul",
		"George",
		"Ringo",
	}
	fmt.Println(names)

	a := names[0:2]
	b := names[1:3]
	fmt.Println(a, b)

	b[0] = "XXX"
	fmt.Println(a, b)
	fmt.Println(names)
}
```

切面字面量类似于数组字面量，只是不需要指定长度。

```go
// 数组字面量
[3]bool{true, true, false}

// 切片字面量，创建了与上例数组字面量相同的数据，然后创建了引用该数组的切片
[]bool{true, true, false}
```

创建切片的时候，可以省略上下限的数组下标，并以默认值替代。下限的数组下标默认值为 0，上限的数组下标默认值为数组长度。

```go
var a [10]int
// 以下创建切片语句，都是等价的
a[0:10]
a[:10]
a[0:]
a[:]
```

**切片的长度表示切片包含的元素个数，切片的容量表示切片引用的底层数组的元素个数，从切片的第一个元素开始统计**。切片的长度可以使用 `len(s)` 表达式来获取，切片的容量可以使用 `cap(s)` 表达式获取。**可以通过重新定义切片的上下限来扩展切片的长度，但需要保证切片有足够的容量**。

```go
func main() {
	s := []int{2, 3, 5, 7, 11, 13}
	printSlice(s)

	// Slice the slice to give it zero length.
	s = s[:0]
	printSlice(s)

	// Extend its length.
	s = s[:4]
	printSlice(s)

	// Drop its first two values.
	s = s[2:]
	printSlice(s)
}

func printSlice(s []int) {
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
}
```

切片的零值是 nil，零值切片的长度和容量都是 0，并且没有引用的底层数组。

**可以使用内置的 `make` 函数来创建切片**。`make` 函数会创建一个元素均为零值的数组，然后返回引用该数组的切片。这便是 Go 中创建动态数组的方式。

```go
func main() {
	a := make([]int, 5)
	printSlice("a", a)

	b := make([]int, 0, 5)
	printSlice("b", b)

	c := b[:2]
	printSlice("c", c)

	d := c[2:5]
	printSlice("d", d)
}

func printSlice(s string, x []int) {
	fmt.Printf("%s len=%d cap=%d %v\n",
		s, len(x), cap(x), x)
}
```

切片可以包含任何类型，甚至是其它切片。

**可以使用内置的 `append` 函数来给切片追加元素**。当被追加的切片容量不够时，`append` 函数则会创建一个更大的底层数组，并且返回的切片也会引用这个新创建的数组。更多关于切片的细节，可以看 [Go Slices: usage and internals](https://go.dev/blog/slices-intro)。

#### 练习

```go
package main

import "golang.org/x/tour/pic"

func Pic(dx, dy int) [][]uint8 {
	var result [][]uint8 = make([][]uint8, dy)
	for y, _ := range result {
		result[y] = make([]uint8, dx)
		for x, _ := range result[y] {
			result[y][x] = uint8((x + y) / 2)
		}
	}
	return result
}

func main() {
	pic.Show(Pic)
}
```

### range

在 `for` 循环中，可以使用 `range` 来遍历切片和映射。当遍历切片的时候，每次迭代会返回两个值，第一个是当前元素的下标，第二个是该下标对应的元素的值副本。

```go
func main() {
    var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}
	for i, v := range pow {
		fmt.Printf("2**%d = %d\n", i, v)
	}
}
```

可以通过赋值 `_` 来忽略 `range` 返回的元素下标或者元素值副本。如果只需要元素下标，则可以忽略第二个返回值。

```go
for i, _ := range pow
for _, value := range pow
for i := range pow
```

### map

映射（map）用于将键（key）映射到值（value）。

映射的零值是 nil，零值映射没有任何 key，也不能添加 key。

**可以使用内置的 `make` 函数来创建映射**。`make` 函数会创建一个指定类型的映射，并会为其初始化。

```go
type Vertex struct {
	Lat, Long float64
}

var m map[string]Vertex

func main() {
	m = make(map[string]Vertex)
	m["Bell Labs"] = Vertex{
		40.68433, -74.39967,
	}
	fmt.Println(m["Bell Labs"])
}
```

映射字面量类似于结构体字面量，但必须要指定键。

```go
type Vertex struct {
	Lat, Long float64
}

var m = map[string]Vertex{
	"Bell Labs": Vertex{
		40.68433, -74.39967,
	},
	"Google": Vertex{
		37.42202, -122.08408,
	},
}
```

如果映射的值是一个类型，那么可以在字面量的元素中省略。

```go
type Vertex struct {
	Lat, Long float64
}

var m = map[string]Vertex{
	"Bell Labs": {40.68433, -74.39967},
	"Google":    {37.42202, -122.08408},
}
```

插入或更新映射中的元素：`m[key] = elem`，获取映射中的元素：`elem = m[key]`，删除映射中的元素：`delete(m, key)`，判断映射中是否存在键：`elem, ok = m[key]`，若键不存在，则 `elem` 是该类型的零值，并且 `ok` 值为 `false`。

#### 练习

```go
package main

import (
	"strings"

	"golang.org/x/tour/wc"
)

func WordCount(s string) map[string]int {
	result := make(map[string]int)
	for _, word := range strings.Fields(s) {
		if _, ok := result[word]; ok {
			result[word]++
		} else {
			result[word] = 1
		}
	}
	return result
}

func main() {
	wc.Test(WordCount)
}
```

### function

函数（function）也是值，可以像其它值一样被传递。函数值可以被作为函数的参数和返回值。

**Go 中的函数可以是闭包**。闭包是一个函数值，它引用了函数体之外的变量。该函数可以访问和赋值它引用的变量。从这个意义上来说，闭包函数是被绑定到变量上的。

```go
func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}

func main() {
	pos, neg := adder(), adder()
	for i := 0; i < 10; i++ {
		fmt.Println(
			pos(i),
			neg(-2*i),
		)
	}
}
```

#### 练习：斐波那契闭包

```go
package main

import "fmt"

// fibonacci is a function that returns
// a function that returns an int.
func fibonacci() func() int {
	a, b := 0, 1
	return func() int {
		fib := a
		a = b
		b = fib + b
		return fib
	}
}

func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f())
	}
}
```

## 方法

Go 中没有 class，但是类型（Type）可以有方法（Method）。方法是一种带有接收者（Receiver）参数的特殊函数。

```go
type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
```

除了可以给「结构体类型」定义方法，也可以给「非结构体类型」定义方法。但需要注意的是，只能给当前包范围内的类型定义方法。

```go
type MyFloat float64

func (f MyFloat) Abs() float64 {
	if f < 0 {
		return float64(-f)
	}
	return float64(f)
}
```

可以给「指针类型」定义方法，**接收者为「指针类型」的方法可以修改接收者指针所指向的值**。接收者为「值类型」的方法修改的只是值的副本，Go 中对函数参数的操作也是如此。

```go
type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}
```

接收者为「指针类型」的方法，既可以被指针所调用，也可以被值所调用。Go 会将 `v.Scale(5)` 语句解释为 `(&v).Scale()`。同样的，接收者为「值类型」的方法，也都可以被指针和值所调用，Go 会将 `p.Scale(5)` 解释为 `(*p).Scale(5)`。

通常都会定义接收者为「指针类型」的方法，主要有两点原因：

1. 可以修改指针所指向的值；
2. 避免在调用方法时候产生的值拷贝。

## 接口

接口（Interface）类型是一组方法签名定义的集合。接口类型的值可以是任何实现了所有这些方法的类型的值。

Go 中不需要「显式声明实现一个接口」，也没有 `implements` 关键字，**实现一个接口只需实现它的所有方法即可**。这种隐式的实现接口方式，可以解耦「接口的定义」和「接口的实现」。

在底层实现中，可以把接口类型认为是一个「值 + 具体类型」的元祖 `(value, type)`。接口值保存的就是接口具体类型的值，调用接口值的方法就是在接口具体类型上调用接口值的同名方法。

### 值为 nil 的接口

如果接口值为 nil，则调用接口方法时的接收者就是 nil。在一些语言中，这种情况可能会导致一个空指针异常，但在 Go 中，可以优雅地处理这种情况。

```go
type I interface {
	M()
}

type T struct {
	S string
}

func (t *T) M() {
	if t == nil {
		fmt.Println("<nil>")
		return
	}
	fmt.Println(t.S)
}
```

### nil 接口

**接口值为 nil 的接口本身并不为 nil，nil 接口本身不包含任何值和类型**，并且在 nil 接口上调用方法会导致一个运行时异常。

### 空接口

不包含任何方法的接口被称为空接口。**空接口可以保存任何类型的值，通常被用于处理未知的类型**。

### 类型断言

类型断言（Type Assertion）提供了访问接口值底层的具体类型的值的方式。如下，类型断言语句会断言接口值 `i` 底层的具体类型是否为 `T`，然后将 `T` 类型的具体值赋值给变量 `t`。如果接口值 `i` 底层的具体类型不是 `T`，则会抛出一个异常。

```go
t := i.(T)
```

类型断言可以返回两个值，第二个返回值用于判断接口值底层是否保存了某个具体类型。如下，如果接口值 `i` 底层的具体类型是 `T`，则 `t` 就是具体类型的值，`ok` 值为 `true`。如果接口值 `i` 底层的具体类型不是 `T`，则 `t` 是具体类型的零值，`ok` 值为 `false`，并且程序不会抛出异常。

```go
t, ok := i.(T)
```

### 类型选择

类型选择（Type Switch）是一种按序从多个类型断言中选择分支的结构。

```go
switch v := i.(type) {
case T:
    // here v has type T
case S:
    // here v has type S
default:
    // no match; here v has the same type as i
}
```

类型选择语句类似于标准的 `switch` 语句，不过 `case` 子句中匹配的是具体类型（而不是值），它们会被用于和接口值的具体类型所比较。

类型选择的声明部分类似于类型选择 `v :=i.(T)`，不过具体类型 `T` 被替换成了关键字 `type`。

### 常用接口

#### Stringer

`fmt` 包中定义了一个 `Stringer` 接口，用于将接口自身描述为一个字符串。`fmt` 包和其它很多包会使用 `Stringer` 接口来打印值。

```go
type Stringer interface {
    String() string
}
```

#### Error

`error` 是 Go 中内建的接口，用于表示程序的错误状态。类似于 `fmt.Stringer`，`fmt` 包会使用 `error` 接口来打印错误值。

```go
type error interface {
    Error() string
}
```

函数通常都会返回一个 `error` 值，调用函数时则需要判断 `error` 值是否为 nil，`error` 值为 nil 表示函数调用成功，`error` 值不为 nil 表示函数调用失败。

#### Reader

`io` 包中定义了一个 `Reader` 接口，用于读取流中的数据。Go 标准库中有很多 `Reader` 接口的实现，包括文件读写、网络连接、压缩算法、加密算法等等。

```go
type Reader interface {
	Read(p []byte) (n int, err error)
}
```

`Read` 方法会填充给定的字节切片，然后返回填充的字节数量和错误值。错误值 `io.EOF` 表示已经读取到了流的末尾。

#### Image

`image` 包中定义了一个 `Image` 接口，用于表示一张图片。

```go
type Image interface {
	ColorModel() color.Model
	Bounds() Rectangle
	At(x, y int) color.Color
}
```

## 并发

### goroutine

协程（goroutine）是由 Go 运行时管理的轻量级线程。

`go f(x, y, z)` 语句表示启动一个新的协程 `f(x, y, z)` 并执行。

`f`、`x`、`y`、`z` 会在当前的协程中求值，`f` 函数会在新的协程中执行。

多个 Go 协程会在相同的地址空间中运行，因此在协程中访问共享的内存变量时必须保证同步。

### channel

管道（channel）是带有类型的渠道，可以使用操作符 `<-` 在管道上发送和接收数据。

```go
ch <- v    // Send v to channel ch.
v := <-ch  // Receive from ch, and
           // assign value to v.
```

与映射和切片一样，管道必须先创建，然后才能使用：

```go
ch := make(chan int)
```

操作符 `<-` 表示管道的方向（发送或者接收），如果没有指定方向，则表示管道是双向的。**管道可以被限制为单向的（只能执行发送或者接收操作）**。

```go
var ch chan T          // can be used to send and receive values of type T
var ch chan<- float64  // can only be used to send float64s
var ch <-chan int      // can only be used to receive ints
```

在默认情况下，管道上的发送和接收操作会被阻塞，直到另一侧的操作已经准备就绪。这使得协程在没有显式锁或者条件变量的情况下可以实现同步操作。

管道是可以带缓冲的，把缓冲长度作为第二个参数赋值给 `make` 函数，就可以创建一个带缓冲的管道。

```go
ch := make(chan int, 100)
```

当缓冲填满时，向管道发送数据会被阻塞。当缓冲为空时，从管道接收数据会被阻塞。

管道的发送者可以使用 `close` 函数来关闭管道，用于表示不再发送任何数据。管道的接收者可以通过接收表达式的第二个参数，来判断管道是否已经关闭。当管道已经关闭时，`ok` 值为 `false`。

```go
v, ok := <-ch
```

`for i := range c` 循环语句将会重复地从管道中获取数据，直至该管道被关闭为止。

只有管道的发送者才可以关闭管道，而不是管道的接收者。在一个已关闭的管道上发送数据时，将会导致程序异常。

管道不同于文件，通常不需要被关闭。仅当需要告诉接收者该管道已经不再发送数据的时候，才需要关闭管道，例如终止一个管道的 `range` 循环。

### select

`select` 语句可以让一个协程等待多个通信操作。`select` 语句会一直阻塞，直到它的某个 `case` 语句可以运行为止，然后执行该 `case` 语句。如果有多个 `case` 语句准备就绪时，`select` 语句会随机选择一个来执行。

如果 `select` 语句中没有准备就绪的 `case` 语句时，则会执行 `default` 语句。

### sync.Mutex

如果只是想要保证在同一个时刻，只有一个协程可以访问一个共享变量，从而避免产生冲突的话，则需要使用一种叫互斥锁（mutex）的数据结构。

Go 标准库中提供了 `sync.Mutex` 结构和它的 `Lock`、`Unlock` 方法来实现互斥操作。可以通过调用 `Lock` 和 `Unlock` 方法来包裹一段代码块，来保证它的互斥执行。也可以通过 `defer` 语句来保证互斥锁一定会被解开。

## 参考资料

- [A Tour of Go](https://go.dev/tour)
- [Go's Declaration Syntax](https://go.dev/blog/declaration-syntax)
- [Go Slices: usage and internals](https://go.dev/blog/slices-intro)
- [The Go Programming Language Specification](https://go.dev/ref/spec)
