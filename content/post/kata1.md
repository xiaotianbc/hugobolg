---
title: "[kata]用golang生成顺时针螺旋"
date: 2020-02-05T13:23:51+08:00
draft: false
tags: ["kata"]
---

## 题目：

Your objective is to complete a function createSpiral(N) that receives an integer N and returns an NxN two-dimensional array with numbers 1 through NxN represented as a clockwise spiral.

## 示例：

Examples:

N = 3 Output: `[[1,2,3],[8,9,4],[7,6,5]]`

```go
1    2    3
8    9    4
7    6    5
```

N = 4 Output: `[[1,2,3,4],[12,13,14,5],[11,16,15,6],[10,9,8,7]]`

```go
1   2   3   4
12  13  14  5
11  16  15  6
10  9   8   7
```

N = 5 Output: `[[1,2,3,4,5],[16,17,18,19,6],[15,24,25,20,7],[14,23,22,21,8],[13,12,11,10,9]]`

```go
1   2   3   4   5
16  17  18  19  6
15  24  25  20  7
14  23  22  21  8
13  12  11  10  9
```

## 代码：

```go
package kata
import "strings"

//向右转向函数，用wasd代表当前的方向，转向就返回右边的方向
func turn(string2 string) string {
  return string("dsaw"[(strings.Index("dsaw", string2)+1)%4])
}

func CreateSpiral(n int) [][]int {
  if n < 1 {
    return [][]int{}
  }
  arr := make([][]int, n)
  for i := 0; i < n; i++ {
    arr[i] = make([]int, n)
  }
  i := 1
  //方向，出发时向右
  direction := "d"
  //当前的坐标
  now := [2]int{0, 0}
  for {
    //给当前坐标赋值
    arr[now[1]][now[0]] = i
    i++
    if i == n*n+1 {
      break
    }
    //考虑下一个方向，如果超出边界，或者值已经存在，说明需要转向了，否则前进一步
  sw:
    switch direction {
    case "d":
      if now[0] == n-1 || arr[now[1]][now[0]+1] != 0 {
        direction = turn(direction)
        goto sw
      }
      now[0]++
    case "s":
      if now[1] == n-1 || arr[now[1]+1][now[0]] != 0 {
        direction = turn(direction)
        goto sw
      }
      now[1]++
    case "a":
      if now[0] == 0 || arr[now[1]][now[0]-1] != 0 {
        direction = turn(direction)
        goto sw
      }
      now[0]--
    case "w":
      if now[1] == 0 || arr[now[1]-1][now[0]] != 0 {
        direction = turn(direction)
        goto sw
      }
      now[1]--
    }
  }
  return arr
}
```