---
title: 查找网站文件中mysql连接信息
date: 2015-05-24 14:50
tags: 实用脚本
---

使用场景：
一个服务器上有上千万 `php`文件
如何在里面找出所有 数据库连接信息呢？
当然很多人第一个想到的就是用grep 正则匹配
但是 数据库连接接字符串很多种形式
所以写了个 `php` 匹配多种形式的 匹配`mysql`连接 字符串的脚本
暂时只写了找php文件中 `mysql`密码的
使用方法，可以将服务器文件弄到本地(尽量在服务器上少作操作)
实例：
**windows cmd下**
cd 服务器存放网站的目录
```
dir /s /b *.php *.inc *.conf *.config >>list.txt
for /f "tokens=*" %i in (list.txt) do  php findpass.php "%i" >>info.txt
```
命令自己组合 如有找不到的 请贴出 连接数据库 部分代码
```php
<?php
isset($argv[1]) ? $file = trim($argv[1]) : exit();
$str = @file_get_contents($file);

$sql = find_pass($str);
if (!empty($sql)) {
    echo '---------------------------------------------' . PHP_EOL;
    echo $file . PHP_EOL . PHP_EOL;
    foreach ($sql as $s) {
        echo trim($s) . PHP_EOL;
    }
    echo '---------------------------------------------' . PHP_EOL . PHP_EOL;
}
//debug
//else {

//    echo 'false ! => ' . $file . PHP_EOL;
//}
function find_pass($str) {
    if (preg_match_all('#\$\w*(?:host(?:name)?|server|user(?:name)?|pass(?:word)?)\w*\s*=\s*(?:\'|\")[[:alnum:][:punct:]]+(?:\'|\")#ism', $str, $sqlstr)) {
        if (count($sqlstr[0]) > 1) {
            //echo "No 1" . PHP_EOL;
            return array_unique($sqlstr[0]);
        }
    }

    if (preg_match_all('#mysqli?(?:_p?connect)?\((?:\'|\")([[:alnum:][:punct:]]*)(?:\'|\")\s*,\s*(?:\'|\")([[:alnum:][:punct:]]*)(?:\'|\")\s*,\s*(?:\'|\")([[:alnum:][:punct:]]*)(?:\'|\")#im', $str, $sqlstr)) {
        //echo "No 2" . PHP_EOL;
        return array_unique($sqlstr[0]);
    }
    if (preg_match_all('#\$[\w]+->db(?:Host|Name|User|Pass)\s+?=\s*\'(.*?)\';#im', $str, $sqlstr)) {
       // echo "No 3" . PHP_EOL;
        return array_unique($sqlstr[0]);
    }
    if (preg_match_all('#^((?!\*).)*(mysqli?:\/\/(?!username:password)[[:alnum:][:punct:]]+@[[:alnum:][:punct:]]*\/[[:alnum:][:punct:]]*)(?:\'|\")#im', $str, $sqlstr)) {
        //echo "No 4" . PHP_EOL;
        return array_unique($sqlstr[0]);
    }
    if (preg_match_all('#^((?!\#|\/\/|\*).)*define\s*\((?:\'|\")(?:\w*SERVER\w*|\w*USER\w*|\w*PASS(?:WORD)?\w*|\w*HOST\w*)(?:\'|\"),\s*(?:\'|\")(.*)(?:\'|\")\)#im', $str, $sqlstr)) {
       // echo "No 5" . PHP_EOL;
        return array_unique($sqlstr[0]);
    }
    if (preg_match_all('#\[database\]\s*driver\s*?=\s*?.*\s*host\s*?=\s*?(?:\'|\")(.*)(?:\'|\")\s*?username\s*?=\s*?(.*)\s*?password\s*?=\s*?(.*)#im', $str, $sqlstr)) {
      //  echo "No 6" . PHP_EOL;
        return array_unique($sqlstr[0]);
    }

    if (preg_match_all('#^((?!\*).)*(?:\'|\")[[:alnum:][:punct:]]*(?:server|user|login|pass|host)[[:alnum:][:punct:]]*(?:\'|\")\s=>\s*[[:alnum:][:punct:]]+(?:\'|\")#im', $str, $sqlstr)) {

        if (count($sqlstr[0]) > 1) {
           // echo "No 7" . PHP_EOL;
            return array_unique($sqlstr[0]);
        }
    }
    if (preg_match_all('#\$[\w\[\]\'\"\s]*(?:host|server|user|name|pass|password|dbpw|hn|un|pw)\w*[\w\[\]\'\"\s]*=\s*(?:\'|\")[[:alnum:][:punct:]]+(?:\'|\")#im', $str, $sqlstr)) {

        if (count($sqlstr[0]) > 1) {
         //   echo "No 8" . PHP_EOL;
            return array_unique($sqlstr[0]);
        }
    }

    if (preg_match_all('#new\sPDO\((?:\'|\")([\w[:punct:]]+)(?:\'|\")\s*,\s*(?:\'|\")([\w[:punct:]]+)\s*,\s*(?:\'|\")([\w[:punct:]]+)(?:\'|\")\)#im', $str, $sqlstr)) {
        //echo "No 9" . PHP_EOL;
        return $sqlstr[0];
    }

    if (preg_match_all('#connect\(\'([[:alnum:][:punct:]]+)\'\s*,\s*\'([[:alnum:][:punct:]]+)\'\s*,\s*\'([[:alnum:][:punct:]]+)\'\s*,\s*\'[[:alnum:][:punct:]]+\'\)#im', $str, $sqlstr)) {
      //  echo "No 10" . PHP_EOL;
        return $sqlstr[0];
    }

    if (preg_match_all('#db_(?:host|login|password|user|username):\s*[[:alnum:][:punct:]]+#im', $str, $sqlstr)) {
       // echo "No 11" . PHP_EOL;
        return $sqlstr[0];
    }
}
?>
```