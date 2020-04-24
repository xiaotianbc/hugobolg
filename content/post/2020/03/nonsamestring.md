---
title: "go写一个在原地完成消除[]string中相邻重复的字符串的操作"
date: 2020-01-30T22:57:51+08:00
draft: false
tags: ["go"]
---

这个是在《go语言圣经》中的一道题，我花了大约一个小时写出了一个自己都不想看的双重循环版本：

```go
func nonSameString(s []string) []string {
	for i, _ := range s {
		if i+1 < len(s) && s[i] == s[i+1] {
			fmt.Printf("发现%d和%d重复，开始修改", i, i+1)
			for i1 := i; i1 < len(s)-1; i1++ {
				s[i1] = s[i1+1]
			}
			s = s[:len(s)-1]
			fmt.Println("原地移动，修改完成", s)
		}
	}
	return s
}
```

如果思路不变的话，稍微优化一下嵌套也只能写成下面这个样子：

```go
func nonSameString(s []string) []string {
	for i, _ := range s {
		if i+1 >= len(s) || s[i] != s[i+1] {
			continue
		}
		for i1 := i; i1 < len(s)-1; i1++ {
			s[i1] = s[i1+1]
		}
		s = s[:len(s)-1]
	}
	return s
}
```

但是时间复杂度O(n2)简直不能忍，再回头看书里原本删除空字符串的例子，发现其实解题的思路是完全一样的：

```go
// nonempty returns a slice holding only the non-empty strings.
// The underlying array is modified during the call.
func nonempty(strings []string) []string {
    i := 0
    for _, s := range strings {
        if s != "" {
            strings[i] = s
            i++
        }
    }
    return strings[:i]
}
```

根据这个思路就可以重新写一个新的删除相同字符串的函数：

```go
//遍历一遍输入的字符串，把所有相邻元素中较为靠后的那一个赋值给索引i，就实现了O(n)的原地删除。
//当然，最后一个元素肯定没有能够与之比较的元素了，所以单独分配一下也ok。
func nonNearSameStr(s []string) []string {
	i := 0
	for in, v := range s {
		if in+1 < len(s) && s[in] != s[in+1] {
			s[i] = v
			i++
		}
	}
	s[i] = s[len(s)-1]
	return s[:i+1]
}
```

写出的程序看起来都比之前正常多了。。  
《go语言圣经》这本书里的例子都非常经典，没有一个多余的废话，值得慢慢品味。

 偷偷的说一下，在谷歌搜索这个题目，第一页的好几个答案都是时间复杂度超高的解法，比如使用了`copy`函数。  