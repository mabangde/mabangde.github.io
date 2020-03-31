---
title: php Thread实例
date: 2014-08-06 20:21
tags: php代码控
---

>支持windows linux

[安装配置参考](http://php.net/manual/en/pthreads.installation.php)
[参考1](http://docs.php.net/manual/zh/class.thread.php)
[代码示例1](https://github.com/krakjoe/pthreads/tree/master/examples)
[代码示例2](http://netkiller.github.io/journal/thread.php.html)

拿个 批量url转 ip 做例子：

```php
<?php
class test_thread_run extends Thread
{
public $url;
public $data;

public function __construct($url)
{
$this->url = $url;
}

public function run()
{
if(($url = $this->url))
{
$this->data = gethostbyname($url);
}
}
}

function model_thread_result_get($urls_array)
{
foreach ($urls_array as $key => $value)
{
$thread_array[$key] = new test_thread_run($value);
$thread_array[$key]->start();
}

foreach ($thread_array as $thread_array_key => $thread_array_value)
{
while($thread_array[$thread_array_key]->isRunning())
{
usleep(10);
}
if($thread_array[$thread_array_key]->join())
{
$variable_data[$thread_array_key] = $thread_array[$thread_array_key]->data;
}
}
return $variable_data;
}

$urls=file('hosts.txt');
//$print_r($urls);
$c= count($urls);
//$h[]=array(");
foreach ($urls as $u){
$urls_array[]=trim($u);
}

//print_r($urls_array);
$t = microtime(true);
$result = model_thread_result_get($urls_array);
//print_r($result);
echo "将$c 条url转换为 ip\n";
$e = microtime(true);
echo "多线程耗时：".($e-$t)."\n";

$t = microtime(true);
foreach ($urls_array as $value)
{
gethostbyname($value);
}

$e = microtime(true);
echo "单线程耗时：".($e-$t)."\n";
?>
```



