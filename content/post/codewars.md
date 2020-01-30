---
title: "在Codewars--代码战争中锻炼你的熟练度"
date: 2020-01-28T14:57:51+08:00
draft: false
---

近日在家学习golang，发现了这么一个网站：[codewars](https://www.codewars.com/r/cbyrzw)，里面的题目大多数都是比较侧重实用的，刷了一会儿感觉挺有趣的，同时对于初学者来说收获也是有的：  

- 学会看官网的文档，golang的文档友善度简直Max，如果觉得某个功能有内置的函数，找一下基本都能找到。
- 求除数之类的算法，必须求到sqrt(x)而不是x，两者的时间复杂度差了一个数量级。
- 创建切片的时候尽量先确定长度，然后向里填充元素，如果频繁使用Append，时间和空间性能都比较差。
- 尽量避免多层if，使用类似于switch结构的if代码可读性更高，有时候只需要改一点点判断顺序而已。
- 在可以使用`continue`的地方尽量使用，这个主要是golang的编译器优化问题。
- 随时写注释。
- 自己封装方法很好用，如果可以把一个逻辑单独封装出来，那就单独封装。

目前已经升到了`6 kyu`，节选最近做的题目：

[Given two integers m, n (1 <= m <= n) we want to find all integers between m and n whose sum of squared divisors is itself a square. 42 is such a number.](https://www.codewars.com/kata/55aa075506463dac6600010d/go)  

```go
package kata

import (
	"math"
)

func ListSquared(m, n int) (ans [][]int) {
	ans = [][]int{}
	for i := m; i <= n; i++ {
		sum := 0
		for _, v := range getDivisors(i) {
			sum += v * v
		}
		//decide if the sum of squared divisors is itself a square
		if math.Sqrt(float64(sum)) == float64(int(math.Sqrt(float64(sum)))) {
			ans = append(ans, []int{i, sum})
		}
	}
	return
}

func getDivisors(n int) (divisors []int) {
	divisors = []int{}
	//如果循环不是执行到平方根，消耗时间差了n个数量级
	for i := 1; i <= int(math.Sqrt(float64(n))); i++ {
		if n%i == 0 && i != n/i {
			divisors = append(divisors, i, n/i)
		} else {
			if n%i == 0 {
				divisors = append(divisors, i)
			}
		}
	}
	return
}
```

[Tribonacci Sequence](https://www.codewars.com/kata/556deca17c58da83c00002db)
```golang
package kata

func Tribonacci(signature [3]float64, n int) (ans []float64) {
	if n == 0 {
		return []float64{}
	}
	ans = make([]float64, n)
	for i, _ := range ans {
		if i <= 2 {
			ans[i] = signature[i]
		} else {
			ans[i] = ans[i-3] + ans[i-2] + ans[i-1]
		}
	}
	return
}
```

[	In this simple Kata your task is to create a function that turns a string into a Mexican Wave. You will be passed a string and you must return that string in an array where an uppercase letter is a person standing up.](https://www.codewars.com/kata/58f5c63f1e26ecda7e000029)
```golang
package kata

import "strings"
func wave(words string) (ans []string) {
	// init ans to be not nil
	ans = []string{}
	//replaceList show which should be replace
	var replaceList []int
	//get replaceList
	for i, v := range words {
		if v != ' ' {
			replaceList = append(replaceList, i)
		}
	}
	//The input string maybe empty
	if words == "" || len(replaceList) == 0 {
		return
	}
	//get Answer
	for _, v := range replaceList {
		newStr := words[:v] + strings.ToUpper(string(words[v])) + words[v+1:]
		ans = append(ans, newStr[:])
	}
	return
}
```

[Write a function toWeirdCase (weirdcase in Ruby) that accepts a string, and returns the same string with all even indexed characters in each word upper cased, and all odd indexed characters in each word lower cased. The indexing just explained is zero based, so the zero-ith index is even, therefore that character should be upper cased.](https://www.codewars.com/kata/52b757663a95b11b3d00062d)
```golang
package kata

import "strings"

func toWeirdCase(str string) string {
	strSlice := strings.Split(str, " ")
	var wcSlice []string

	for _, v := range strSlice {
		var newStr string
		for i, b1 := range v {
			if i%2 == 0 {
				newStr += strings.ToUpper(string(b1))
			} else {
				newStr += strings.ToLower(string(b1))
			}
		}
		wcSlice = append(wcSlice, newStr)
	}
	return strings.Join(wcSlice, " ")
}
```