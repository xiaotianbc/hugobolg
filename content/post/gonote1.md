---
title: "go语言圣经阅读笔记"
date: 2020-01-29T11:57:51+08:00
draft: false
---

## 词法作用域

```go
if x := f(); x == 0 {
    fmt.Println(x)
} else if y := g(x); x == y {
    fmt.Println(x, y)
} else {
    fmt.Println(x, y)
}
fmt.Println(x, y) // compile error: x and y are not visible here
```

第二个 if 语句嵌套在第一个内部，因此第一个 if 语句条件初始化词法域声明的变量在第二个 if 中也可以访问。switch 语句的每个分支也有类似的词法域规则：条件部分为一个隐式词法域，然后每个是每个分支的词法域。

## fmt 使用技巧

当使用 fmt 包打印一个数值时，我们可以用%d、%o 或%x 参数控制输出的进制格式，就像下面的例子：

```go
o := 0666
fmt.Printf("%d %[1]o %#[1]o\n", o) // "438 666 0666"
x := int64(0xdeadbeef)
fmt.Printf("%d %[1]x %#[1]x %#[1]X\n", x)
// Output:
// 3735928559 deadbeef 0xdeadbeef 0XDEADBEEF
```

请注意 fmt 的两个使用技巧。通常 Printf 格式化字符串包含多个%参数时将会包含对应相同数量的额外操作数，但是%之后的`[1]`副词告诉 Printf 函数再次使用第一个操作数。第二，%后的`#`副词告诉 Printf 在用%o、%x 或%X 输出时生成 0、0x 或 0X 前缀。

`[1]`中的 1 是`fmt.Printf`函数内参数的索引，也可以使用 2,3,4 等显示后面等参数，但是`[0]`是不合法的，不能调用第一个参数，会提示`%!(BADINDEX)`

## 逻辑优先级

布尔值可以和&&（AND）和||（OR）操作符结合，并且可能会有短路行为：如果运算符左边值已经可以确定整个布尔表达式的值，那么运算符右边的值将不在被求值，因此下面的表达式总是安全的：

```go
s != "" && s[0] == 'x'
```

其中 s[0]操作如果应用于空字符串将会导致 panic 异常。

&&的优先级比||高（助记：&&对应逻辑乘法，||对应逻辑加法，乘法比加法优先级要高），下面形式的布尔表达式是不需要加小括弧的：

```go
if 'a' <= c && c <= 'z' ||
    'A' <= c && c <= 'Z' ||
    '0' <= c && c <= '9' {
    // ...ASCII letter or digit...
}
```

## 字符串和数字的转换

将一个整数转为字符串，一种方法是用 fmt.Sprintf 返回一个格式化的字符串；另一个方法是用 strconv.Itoa(“整数到 ASCII”)：

```go
x := 123
y := fmt.Sprintf("%d", x)
fmt.Println(y, strconv.Itoa(x)) // "123 123"
```

如果要将一个字符串解析为整数，可以使用 strconv 包的 Atoi 或 ParseInt 函数，还有用于解析无符号整数的 ParseUint 函数：

```go
x, err := strconv.Atoi("123")             // x is an int
y, err := strconv.ParseInt("123", 10, 64) // base 10, up to 64 bits
```

ParseInt 函数的第三个参数是用于指定整型数的大小；例如 16 表示 int16，0 则表示 int。在任何情况下，返回的结果 y 总是 int64 类型，你可以通过强制类型转换将它转为更小的整数类型。

## 数组的声明方式

默认情况下，数组的每个元素都被初始化为元素类型对应的零值，对于数字类型来说就是 0。我们也可以使用数组字面值语法用一组值来初始化数组：

```go
var q [3]int = [3]int{1, 2, 3}
var r [3]int = [3]int{1, 2}
fmt.Println(r[2]) // "0"
```

在数组字面值中，如果在数组的长度位置出现的是“...”省略号，则表示数组的长度是根据初始化值的个数来计算。因此，上面 q 数组的定义可以简化为

```go
q := [...]int{1, 2, 3}
fmt.Printf("%T\n", q) // "[3]int"
```

未指定初始值的元素将用零值初始化。例如下面的代码定义了一个含有 100 个元素的数组 r，最后一个元素被初始化为-1。  
我们声明数组时，索引为 99 的第 100 个元素被设置为-1，直接跳过第 0 到第 99 个元素，它们会被自动初始化为 0 :

```go
r := [...]int{99: -1}
```

如果一个数组的元素类型是可以相互比较的，那么数组类型也是可以相互比较的，这时候我们可以直接通过==比较运算符来比较两个数组，只有当两个数组的所有元素都是相等的时候数组才是相等的。不相等比较运算符!=遵循同样的规则。

```go
a := [2]int{1, 2}
b := [...]int{1, 2}
c := [2]int{1, 3}
fmt.Println(a == b, a == c, b == c) // "true false false"
d := [3]int{1, 2}
fmt.Println(a == d) // compile error: cannot compare [2]int == [3]int
```

我们可以显式地传入一个数组指针，那样的话函数通过指针对数组的任何修改都可以直接反馈到调用者。下面的函数用于给[32]byte 类型的数组清零：

```go
func zero(ptr *[32]byte) {
    for i := range ptr {
        ptr[i] = 0
    }
}
```

其实数组字面值[32]byte{}就可以生成一个 32 字节的数组。而且每个数组的元素都是零值初始化，也就是 0。因此，我们可以将上面的 zero 函数写的更简洁一点：

```go
func zero(ptr *[32]byte) {
    *ptr = [32]byte{}
}
```

## Slice 的零值与 nil

一个零值的 slice 等于 nil。一个 nil 值的 slice 并没有底层数组。一个 nil 值的 slice 的长度和容量都是 0，但是也有非 nil 值的 slice 的长度和容量也是 0 的，例如`[]int{}`或`make([]int, 3)[3:]`。与任意类型的 nil 值一样，我们可以用`[]int(nil)`类型转换表达式来生成一个对应类型 slice 的 nil 值。

```go
var s []int    // len(s) == 0, s == nil
s = nil        // len(s) == 0, s == nil
s = []int(nil) // len(s) == 0, s == nil
s = []int{}    // len(s) == 0, s != nil
```

如果你需要测试一个 slice 是否是空的，使用`len(s) == 0`来判断，而不应该用`s == nil`来判断。除了和 nil 相等比较外，一个 nil 值的 slice 的行为和其它任意 0 长度的 slice 一样；除了文档已经明确说明的地方，所有的 Go 语言函数应该以相同的方式对待 nil 值的 slice 和 0 长度的 slice。

## 结构体初始化

你不能企图在外部包中用顺序赋值的技巧来偷偷地初始化结构体中未导出的成员。

```go
package p
type T struct{ a, b int } // a and b are not exported

package q
import "p"
var _ = p.T{a: 1, b: 2} // compile error: can't reference a, b 这样不可以
var _ = p.T{1, 2}       // compile error: can't reference a, b
```

虽然上面最后一行代码的编译错误信息中并没有显式提到未导出的成员，但是这样企图隐式使用未导出成员的行为也是不允许的。

## defer和return谁先执行？

直接看下面代码：

```go
package main

import "fmt"

func main() {
	f2()
}

func f1() int {
	fmt.Println("f1")
	return 0
}

func f2() int {
	defer fmt.Println("defer")
	return f1()
}
// Output:
//f1
//defer
```

由于main函数不能有返回值，所以用f2来判断顺序，可以看到是先执行return里的函数得到结果，再执行defer.