# Go 语言中的陷阱

## 多行语句中自动添加 `;` 符号

Go 会在当行尾的元素属于以下元素时，自动添加 `;` 符号：

- [identifier](https://go.dev/ref/spec#Identifiers)
- [integer](https://go.dev/ref/spec#Integer_literals)、[floating-point](https://go.dev/ref/spec#Floating-point_literals)、[imaginary](https://go.dev/ref/spec#Imaginary_literals)、[rune](https://go.dev/ref/spec#Rune_literals)、[string](https://go.dev/ref/spec#String_literals) 字面量之一
- `break`、`continue`、`fallthrough`、`return` 关键字之一
- `++`、`--`、`)`、`]`、`}` 操作符之一

因此下列多行语句都是非法的：

```go
i := 1 + 2 + 3 + 4 // 会自动添加 ;
	+ 5 // error

i, j, k := 1, 2 // 会自动添加 ;
	, 3 // error

if i > 0 && j > 0 // 会自动添加 ;
	&& k > 0 { // error
}
```

参考 Go Spec：<https://go.dev/ref/spec#Semicolons>。

## 没有 `?:` 三目运算符

Go 中没有 `?:` 三目运算符，需要使用 `if {...} else {...}` 语句来替换。因为 Go 的设计者认为 `?:` 运算符经常被用于创建难以理解的表达式，而 `if {...} else {...}` 语句虽然冗余但易于理解。

参考 Go FAQ：<https://go.dev/doc/faq#Does_Go_have_a_ternary_form> 。

## 以 goroutine 来运行闭包

下面这段代码期望输出是 `a b c`（不过可能是乱序的），但实际输出的却是 `c c c` 。这是因为在 `for` 循环的每次迭代中，都会使用相同的变量 `v` 实例（`&v` 值是相同的），并且 goroutine 中每个闭包都共享了这个变量 `v`。当闭包在运行时通过 `fmt.Printf` 打印变量 `v` 的值时，变量 `v` 可能在 goroutine 启动之后就被修改了。可以使用 `go vet` 命令来检测这类潜在的问题。

```go
done := make(chan bool)
values := []string{"a", "b", "c"}

for _, v := range values {
	go func() {
		fmt.Printf("%v ", v)
		done <- true
	}()
}

for range values {
	<-done
}
```

为了避免上述问题，可以在 goroutine 启动时把变量 `v` 的当前值绑定到各个闭包上。例如把变量 `v` 作为参数传递给闭包。

```go
for _, v := range values {
	go func(u string) {
		fmt.Printf("%v ", u)
		done <- true
	}(v)
}
```

参考 Go FAQ：<https://go.dev/doc/faq#closures_and_goroutines> 。
