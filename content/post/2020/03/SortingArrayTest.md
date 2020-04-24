---
title: "用golang排序比较单核性能"
date: 2020-01-23T22:23:51+08:00
draft: false
tags: ["go"]
---
采用的golang代码如下，就是最简单的冒泡排序：

``` golang
package main

import (
	"fmt"
	"math/rand"
	"time"
)

func main() {
	t1 := time.Now().UnixNano()
	var arr1 = makeArr(99)
	t2 := time.Now().UnixNano()
	fmt.Println("长度为100000的数组测试")
	fmt.Printf("生成数组耗时 %v s\n", float32(t2-t1)/1000000000)
	//fmt.Println("随机生成的数组是：", arr1)
	for j := 1; j < len(arr1); j++ {
		for i := 0; i < len(arr1)-j; i++ {
			if arr1[i] > arr1[i+1] {
				arr1[i], arr1[i+1] = arr1[i+1], arr1[i]
			}
		}
	}
	t3 := time.Now().UnixNano()
	fmt.Printf("排序数组耗时 %v s\n", float32(t3-t2)/1000000000)
	//fmt.Println("冒泡排序的结果是：", arr1)
}
func makeArr(max int) [100000]int {
	rand.Seed(time.Now().UnixNano())
	var arr = [100000]int{}
	for i := 0; i < 100000; i++ {
		arr[i] = rand.Intn(max)
	}
	return arr
}
```

我首先在本地电脑上不同的系统中进行测试,结果符合预期。

``` shell
I7 8700 - windows

长度为100000的数组测试
生成数组耗时 0.0019821 s
排序数组耗时 13.415154 s

i7 8700 -  macOS

长度为100000的数组测试
生成数组耗时 0.003636 s
排序数组耗时 12.02046 s

i7 8700 -  linux

长度为100000的数组测试
生成数组耗时 0.004791608 s
排序数组耗时 11.992343 s
```

然后我在其他设备和linux进行了测试：

``` shell

pacificrack:

长度为100000的数组测试
生成数组耗时 0.02271442 s
排序数组耗时 25.984821 s

阿里云

长度为100000的数组测试
生成数组耗时 0.005643272 s
排序数组耗时 23.403088 s

斐讯N1

长度为100000的数组测试
生成数组耗时 0.018707875 s
排序数组耗时 78.00233 s

cloudcone

长度为100000的数组测试
生成数组耗时 0.014986864 s
排序数组耗时 21.606495 s

timeweb.ru

长度为100000的数组测试
生成数组耗时 0.014752394 s
排序数组耗时 18.938543 s

星际蜗牛

长度为100000的数组测试
生成数组耗时 0.011626679 s
排序数组耗时 30.803465 s

Nubia Z17 - Termux

长度为100000的数组测试
生成数组耗时 0.015635053 s
排序数组耗时 32.497234 s

```
