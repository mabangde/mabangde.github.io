---
title: 正则表达式笔记
date: 2016-05-27 17:21
tags: 开发基础
---

## POSIX字符集
`[:space:]`  所有空白字符（新行，空格，制表符）`[\t\r\n\v\f]`

`[:digit:]`  数字

`[:xdigit:]` 任何一个十六进制数（即：0-9，a-f，A-F）

`[:punct:]`  所有标点符号

`[:alpha:]`  所有的字母  包括汉字

`[:upper:]`  所有的大写字母

`[:lower:]`  所有的小写字母

`[:alnum:]`  字母和数字 相当于`[0-9a-zA-Z]`  包括汉字

`[:cntrl:]`  控制字符 `[\x00-\x1F\x7F]` 

`[:print:]`  非空字符 类似`[:graph:]` 但包括空白字符 `[\x20-x7E]`

`[:graph:]`  空白字符之外的字符 `[\x21-\x7E]`（非空格、控制字符）  

`[:blank:]`  空格(space)与定位符(tab)

`[:ascii:]`  ASCII 字符 `[\x00-\x7F]`

## 正则表达之环视(Lookaround)
`(?<=Expression)` **逆序肯定环视**，表示所在位置左侧能够匹配`Expression`

`(?<!Expression)` **逆序否定环视**，表示所在位置左侧不能匹配`Expression`

`(?=Expression)` **顺序肯定环视**，表示所在位置右侧能够匹配`Expression`

`(?!Expression)` **顺序否定环视**，表示所在位置右侧不能匹配`Expression`

环视是正则中的一个难点，对于环视的理解，可以从应用和原理两个角度理解，如果想理解得更清晰、深入一些，还是从原理的角度理解好一些，正则匹配基本原理参考 NFA引擎匹配原理。

上面提到环视相当于对“所在位置”附加了一个条件，环视的难点在于找到这个“位置”，这一点解决了，环视也就没什么秘密可言了。

**顺序环视匹配过程**
对于顺序肯定环视`(?=Expression)`来说，当子表达式`Expression`匹配成功时，`(?=Expression)`匹配成功，并报告`(?=Expression)`匹配当前位置成功。

对于顺序否定环视`(?!Expression)`来说，当子表达式`Expression`匹配成功时，`(?!Expression)`匹配失败；当子表达式`Expression`匹配失败时，`(?!Expression)`匹配成功，并报告`(?!Expression)`匹配当前位置成功；
