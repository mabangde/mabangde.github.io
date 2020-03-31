---
title: sqlmap udf解密脚本
date: 2016-7-29 23:07
tags:
---


一次渗透测试中上传 **sqlmap** 自带的 **udf**发现无法创建函数打开文件一看根本就不像传统的二进制文件像是一种编码查看相关说明 ，原来新版sqlmap 为了防止文件被误杀对文件进行异或加密.

所幸在sqlmap 目录下发现 一个解密 脚本

路径 
`./sqlmap/extra/cloak/cloak.py`

![2019-07-20-17-36-26](https://wolvez.oss-cn-hangzhou.aliyuncs.com/4073e2e993dbcf2491743e2f4f554e0c.png)

如 **lib_mysqludf_sys.dll** (比较喜欢用 sqlmap 里面的 udf 不容易被杀掉)

`cloak.py -d -i D:\sqlmap\udf\mysql\windows\64\lib_mysqludf_sys.dll_`
解码之后便可直接使用了 **webshell** 解码方式也一样

顺便提下 `./sqlmap/extra/` 目录有几个有意思的 东西 感兴趣的同学可以研究下