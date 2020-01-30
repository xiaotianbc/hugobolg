---
title: "用uglify-js压缩js代码"
date: 2020-01-11T20:23:51+08:00
draft: false
---

>>正常情况下请不要给别人看他们不懂的代码，仅仅用于运行时，给机器看即可。

``` shell
npm install -g babel-cli uglify-js
# ES2015转码规则
npm install --save-dev babel-preset-es2015
cat '{"presets": ["es2015"],"plugins": []}' >>.babelrc
```

使用下面的命令即可压缩代码,也可以直接加到package.json里

```shell
babel src/a.js | uglifyjs -c -m  > src/a.min.js
#或者加入package.json里
"min": "babel src\/a.js | uglifyjs -c -m  > src\/a.min.js"
```

效果可以参考下面：  
原版本，这是一个用海龟于画图的代码

``` javascript
const square = function(x, y, l, n, m, space) {
  jump(x + l * n + space * n, y + m * (l + space));
  setHeading(0);
  var i = 0;
  while (i < 4) {
    forward(l);
    right(90);
    i = i + 1;
  }
};

const square_square = function(x, y, space, len, n, m) {
  for (let mt = 0; mt < m; mt++) {
    for (let nt = 0; nt < n; nt++) {
      square(x, y, len, nt, mt, space);
    }
  }
};

square_square(-130, -130, 10, 50, 3, 3);
```

翻译之后：

``` javascript
"use strict";var square=function(r,a,u,e,s,f){jump(r+u*e+f*e,a+s*(u+f)),setHeading(0);for(var o=0;o<4;)forward(u),right(90),o+=1},square_square=function(r,a,u,e,s,f){for(var o=0;o<f;o++)for(var q=0;q<s;q++)square(r,a,e,q,o,u)};square_square(-130,-130,10,50,3,3);
```

用`Prettier`之类的格式化插件格式化之后：

```javascript
"use strict";
var square = function(r, a, u, e, s, f) {
    jump(r + u * e + f * e, a + s * (u + f)), setHeading(0);
    for (var o = 0; o < 4; ) forward(u), right(90), (o += 1);
  },
  square_square = function(r, a, u, e, s, f) {
    for (var o = 0; o < f; o++)
      for (var q = 0; q < s; q++) square(r, a, e, q, o, u);
  };
square_square(-130, -130, 10, 50, 3, 3);
```
