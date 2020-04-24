---
title: "golang调用高德api查询天气"
date: 2020-01-31T21:57:51+08:00
draft: false
tags: ["go"]
---

学习了golang的json之后，完全仿照这一章的程序结构，我尝试写了个调用高德api查询天气的程序，代码如下：

## 获取JSON

```shell
curl https://restapi.amap.com/v3/weather/weatherInfo\?key\=xxx\&city\=310115\&extensions\=all
```

可以得到以下结构：

```json
{
  "status": "1",
  "count": "1",
  "info": "OK",
  "infocode": "10000",
  "forecasts": [
    {
      "city": "浦东新区",
      "adcode": "310115",
      "province": "上海",
      "reporttime": "2020-01-31 21:27:03",
      "casts": [
        {
          "date": "2020-01-31",
          "week": "5",
          "dayweather": "晴",
          "nightweather": "晴",
          "daytemp": "10",
          "nighttemp": "0",
          "daywind": "西北",
          "nightwind": "西北",
          "daypower": "≤3",
          "nightpower": "≤3"
        },
       ...
      ]
    }
  ]
}
```

## 定义结构体和常量：

```go
package api

const WeatherURL string = "https://restapi.amap.com/v3/weather/weatherInfo"
//替换成真正的apikey
const Apikey = ""

type Weather struct {
	Status string   `json:"status"`
	Fc     []*Inner `json:"forecasts"`
}

type DayInfo struct {
	Date         string `json:"date"`
	Dayweather   string `json:"dayweather"`
	Nightweather string `json:"nightweather"`
	Daytemp      string `json:"daytemp"`
	Nighttemp    string `json:"nighttemp"`
}

type Inner struct {
	City       string     `json:"city"`
	Province   string     `json:"province"`
	Reporttime string     `json:"reporttime"`
	Casts      []*DayInfo `json:"casts"`
}

```

## 定义方法



```go
package api

import (
	"encoding/json"
	"fmt"
	"github.com/lwxntm/demo/internal/utils"
	"net/http"
)
//由于使用高德api查到的adcode只能精确到市级别，不是具体的区，所以我直接成常量：
const adcode = "310115"

func GetWeather() (*Weather, error) {
	resp, err := http.Get(WeatherURL + "?key=" + Apikey + `&city=` + adcode + `&extensions=all`)
	utils.HandleError(err)
	defer resp.Body.Close()
	if resp.StatusCode != http.StatusOK {
		return nil, fmt.Errorf("Http error:%d\n ", resp.StatusCode)
	}
	var res Weather
	if err = json.NewDecoder(resp.Body).Decode(&res); err != nil {
		utils.HandleError(err)
	}
	return &res, err
}

func (w Weather) HandleCast( days int) {
	forecast := w.Fc[0]
	fmt.Printf("地区：%s\t%s\n", forecast.Province, forecast.City)
	for i, v := range forecast.Casts {
		if i < days {
			fmt.Printf("日期：%s  白天： %s  %s ℃\t夜里： %s  %s ℃\n", v.Date, v.Dayweather, v.Daytemp, v.Nightweather, v.Nighttemp)
		}
	}
}
```

## 主函数调用

```go
package main

import (
	"github.com/lwxntm/demo/internal/app/adweather/api"
	"github.com/lwxntm/demo/internal/utils"
	"os"
	"strconv"
)

func main() {
	//第二个参数是要显示的天数，默认是 1
	n := 1
	if len(os.Args) != 1 {
		n, _ = strconv.Atoi(os.Args[1])
	}
	//获取天气
	w, err := api.GetWeather()
	utils.HandleError(err)
	//显示天气
	w.HandleCast(n)
}

```

参考文章：

* [4.5. JSON | Go语言圣经](https://yar999.gitbooks.io/gopl-zh/content/ch4/ch4-05.html)