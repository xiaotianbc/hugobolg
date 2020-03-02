---
title: VSCode代码片段内开启快速建议
date: 2019-12-08 11:32:47
tags: ["vscode"]
---
在使用VSCode时经常遇到一个问题是：使用tab补全的方法自动生成的括号里没有快速建议，可以修改一个设置解决：
```
"editor.suggest.snippetsPreventQuickSuggestions": false
```