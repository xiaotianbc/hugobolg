---
title: "go语言圣经阅读笔记"
date: 2020-01-29T11:57:51+08:00
draft: false
---

## 词法作用域

``` go
if x := f(); x == 0 {
    fmt.Println(x)
} else if y := g(x); x == y {
    fmt.Println(x, y)
} else {
    fmt.Println(x, y)
}
fmt.Println(x, y) // compile error: x and y are not visible here
```

第二个if语句嵌套在第一个内部，因此第一个if语句条件初始化词法域声明的变量在第二个if中也可以访问。switch语句的每个分支也有类似的词法域规则：条件部分为一个隐式词法域，然后每个是每个分支的词法域。

## fmt使用技巧

当使用fmt包打印一个数值时，我们可以用%d、%o或%x参数控制输出的进制格式，就像下面的例子：

``` go
o := 0666
fmt.Printf("%d %[1]o %#[1]o\n", o) // "438 666 0666"
x := int64(0xdeadbeef)
fmt.Printf("%d %[1]x %#[1]x %#[1]X\n", x)
// Output:
// 3735928559 deadbeef 0xdeadbeef 0XDEADBEEF
```

请注意fmt的两个使用技巧。通常Printf格式化字符串包含多个%参数时将会包含对应相同数量的额外操作数，但是%之后的`[1]`副词告诉Printf函数再次使用第一个操作数。第二，%后的`#`副词告诉Printf在用%o、%x或%X输出时生成0、0x或0X前缀。  

`[1]`中的1是`fmt.Printf`函数内参数的索引，也可以使用2,3,4等显示后面等参数，但是`[0]`是不合法的，不能调用第一个参数，会提示`%!(BADINDEX)`

## 逻辑优先级

布尔值可以和&&（AND）和||（OR）操作符结合，并且可能会有短路行为：如果运算符左边值已经可以确定整个布尔表达式的值，那么运算符右边的值将不在被求值，因此下面的表达式总是安全的：  
``` go
s != "" && s[0] == 'x'
```
其中s[0]操作如果应用于空字符串将会导致panic异常。

&&的优先级比||高（助记：&&对应逻辑乘法，||对应逻辑加法，乘法比加法优先级要高），下面形式的布尔表达式是不需要加小括弧的：  

``` go
if 'a' <= c && c <= 'z' ||
    'A' <= c && c <= 'Z' ||
    '0' <= c && c <= '9' {
    // ...ASCII letter or digit...
}
```

## 字符串和数字的转换

将一个整数转为字符串，一种方法是用fmt.Sprintf返回一个格式化的字符串；另一个方法是用strconv.Itoa(“整数到ASCII”)：  

```go
x := 123
y := fmt.Sprintf("%d", x)
fmt.Println(y, strconv.Itoa(x)) // "123 123"
```

如果要将一个字符串解析为整数，可以使用strconv包的Atoi或ParseInt函数，还有用于解析无符号整数的ParseUint函数：  

```go
x, err := strconv.Atoi("123")             // x is an int
y, err := strconv.ParseInt("123", 10, 64) // base 10, up to 64 bits
```

ParseInt函数的第三个参数是用于指定整型数的大小；例如16表示int16，0则表示int。在任何情况下，返回的结果y总是int64类型，你可以通过强制类型转换将它转为更小的整数类型。  
