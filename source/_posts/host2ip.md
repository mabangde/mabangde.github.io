---
title: 批量域名转ip及相关信息
date: 2015-05-24 18:11
tags: 多进程 php
---

功能描述：
将大量的域名转换成ip 按ip排序 并获取服务器基本信息

使用场景：
如某个大型网络 我们获取了数千子域名
有多个网段
我们通过ip 以及服务器基本信息 判断 域名所对应的网段 及所采用的 WebServer 服务器 版本

该脚本采用 多进程技术 加快了转换速度
控制超时时间 防止 执行卡死 丢弃 无法访问的 ip
使用前请确认您dns 解析速度够快
建议将dns 设置为 `8.8.8.8` 或`114.114.114.114`
或挂海外vpn
以获取最佳结果
由于该脚本使用`wmi` 控制多进程所以仅支持**windows**，**linux**下用其它技术实现
该脚本调用php 模块：`pdo_sqlite` `curl` 使用前请确认 您的php 开启相关模块
使用前请不要修改文件名字

windows 命令行下：

`commandline > php host2ip.php host.txt`
**host.txt** 为多个 网站域名文件
执行完毕将会在当前目录生成**result.txt**的结果文件

**resolveip.php**
```php
<?php
isset($argv[1]) ? $host = trim($argv[1]) : exit();
include 'sqlite.php';
$sql = new sqllite();
$result = _gethostbyname($host);

//域名转ip

function _gethostbyname($host) {
    $ip = gethostbyname($host);
    if (chk_ip($ip)) {
        $i['host'] = $host;
        $i['ip'] = $ip;
        return $i;
    }
}

if (!empty($result['ip'])) {
    $sc = $sql->server_ip_exit($result['ip']);
    if ($sc != 0) {
        
        $serv = $sql->get_server($result['ip']);
        $sql->insert($result['host'], $result['ip'], $serv);
    } 
    else {
        $server = get_server($result['ip']);
        $sql->insert($result['host'], $result['ip'], $server);
    }
}

function get_server($host) {
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $host);
    curl_setopt($ch, CURLOPT_HEADER, true);
    curl_setopt($ch, CURLOPT_NOBODY, true);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_USERAGENT, 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.75.14 (KHTML, like Gecko) Version/7.0.3 Safari/7046A194A');
    curl_setopt($ch, CURLOPT_TIMEOUT, 5);
    curl_setopt($ch, CURLOPT_CONNECTTIMEOUT, 5);
    
    $head = curl_exec($ch);
    preg_match_all('/^(?:Server|X-Powered-By):\s([^[:cntrl:]]+)/im', $head, $server);
    $info = $server[1];
    $c = count($info);
    empty($info) ? $banner = false : $banner = true;
    if ($banner && $c != 0) {
        $c == 2 ? $return = $info[0] . ' | ' . $info[1] : $return = $info[0];
    } 
    else {
        return '(none)';
    }
    return $return;
}

function chk_ip($ip) {
    if (filter_var($ip, FILTER_VALIDATE_IP, FILTER_FLAG_IPV4 | FILTER_FLAG_NO_PRIV_RANGE) && substr($ip, 0, 4) != '127.' && substr($ip, 0, 4) != '255.' && $ip != '0.0.0.0') {
        return $ip;
    } 
    else {
        return false;
    }
}
?>
```
**host2ip.php**
```php
<?php
date_default_timezone_set('PRC');
$start_time = func_time();
is_file($argv[1])?$hostfile=$argv[1]:exit("php $argv[0] host.txt");
$list=geturllist($hostfile);
$counts=sizeof($list);	

$Refresh=3000000;//刷新时间3 秒
$thread=20; //并发
$ExecTimeout=10;
//$chunk=array_chunk($list,$thread);//从0 开始
printf("[i] 数量 %d 条,线程数%d ,刷新时间%d微秒,超时时间 %d秒\r\n",$counts,$thread,$Refresh,$ExecTimeout);
usleep(500000);
echo "程序执行中...\r\n";
foreach ($list as $c ){
	$task= task('resolveip.php',$ExecTimeout);
	back_run('php.exe resolveip.php '.$c);
  if($task >= $thread){
		  usleep($Refresh);
	}
	
}

function task($taskname,$ExecTimeout){
$LocalWMISvc=new com('WinMgmts:{impersonationLevel=impersonate}!\\\\.\root\CIMV2');

$QueryObjSet=$LocalWMISvc->ExecQuery("select * from win32_process where commandline like '%{$taskname}%' and (name='php.exe' or name='cmd.exe')");
$taskcount= $QueryObjSet->Count;
foreach ($QueryObjSet as $process){
if((func_time() - strtotime(substr($process->CreationDate,0,14))) >= $ExecTimeout){
	@exec("taskkill -pid $process->ProcessId /f 2>nul",$o,$r); //超时则强制退出
	}
}
return $taskcount;  //返回进程数
}



function back_run($cmd){
	pclose(popen("start /B ". $cmd , "r")); 
}


function _echo($str){
echo $str.PHP_EOL;

}

function func_time()
{
    list($microsec, $sec) = explode(' ', microtime());
    return $sec;
}

function geturllist($hostfile){
$urllist=file($hostfile);
if(is_array($urllist)){
	$arr=array_unique($urllist);
	$url='';
	$urls='';
foreach ($arr as $s){
	
	$p = array(
        '/(http|https|ftp):\/\//i',
        '/\//i',
        '/\s*?/',
    );
    $url = preg_replace($p, "", $s);
    if(!empty($url)){
    $urls[]=$url;
    }
}

}
return $urls;
}

sleep(1);
if(task('resolveip.php',$ExecTimeout) !=0){
sleep(5);
}
include 'sqlite.php';
$sql=new sqllite();
$c=$sql->show_count();
$Etime=round((func_time() - $start_time), 4);
$fs=($counts-$c);
printf("共 %d host , 转换成功 %d 条, 失败 %d 条\r\n耗时:%d 秒 \r\n\r\n",$counts,$c,$fs,$Etime);
$sql->dump();
//$sql->deleteTable();
$sql=null;
?>
```
**sqlite.php**
```php
<?php
class sqllite
{
    private $ResultFile = 'result.txt';
    private $r_db;
    private $r_table = 'url2ip';
    public function __construct() {
        try {
            
            $this->r_db = new PDO('sqlite:url2ip.db', null, null, array(PDO::ATTR_PERSISTENT => true));
             //存储结果集
            
            //初始化 创建相应的表
            $sql = "PRAGMA synchronous = OFF;";
            $sql.= <<<EOF
CREATE TABLE IF NOT EXISTS {$this->r_table} (
	'id'  INTEGER primary key AUTOINCREMENT NOT NULL,
	'domain'  varchar(50) NOT NULL ,
	'ip'  varchar(20)  NOT NULL,
	'server' varchar(100)	
);

CREATE UNIQUE INDEX 'unique_{$this->r_table}'
ON 'url2ip' ('domain' ASC, 'ip' ASC);
EOF;
            $this->r_db->exec($sql);
        }
        catch(PDOException $ex) {
            die("[Error]  " . $ex->getMessage() . PHP_EOL . '         add/edit in php.ini' . PHP_EOL . '         extension=pdo_sqlite.so or pdo_sqlite.dll' . PHP_EOL);
        }
    }
    
    public function insert($value1, $value2, $value3 = NULL) {
        $db = $this->r_db;
        $sql = <<<EOF
insert into {$this->r_table}('domain','ip','server') VALUES ('{$value1}', '{$value2}','{$value3}') ;
EOF;
        $db->exec($sql);
    }
    
    public function show_result() {
        $sql = <<<EOF
SELECT domain,ip,server from {$this->r_table}  ORDER BY ip;
EOF;
        
        $sql = $this->r_db->query($sql);
        $result = $sql->fetchAll();
        return $result;
    }
    
    public function show_count() {
        $sql = <<<EOF
SELECT count(*) from {$this->r_table};
EOF;
        
        $result = $this->r_db->prepare($sql);
        $result->execute();
        $result = $result->fetch();
        
        return $result[0];
    }
    
    public function deleteTable() {
        $db = $this->r_db;
        $sql = <<<EOF
drop table if exists {$this->r_table};
delete from sqlite_sequence;
EOF;
        $db->exec($sql);
    }
    
    public function server_ip_exit($ip) {
        $sql = <<<EOF
SELECT count(*) from {$this->r_table} where ip='{$ip}' and server not NULL;   //该ip server无值
EOF;
        $result = $this->r_db->prepare($sql);
        $result->execute();
        $result = $result->fetch();
        return $result[0];
    }
    
    public function get_server($ip) {
        $sql = <<<EOF
select server from {$this->r_table} where ip='{$ip}' and server is not null limit 1;
EOF;
        $result = $this->r_db->prepare($sql);
        $result->execute();
        $result = $result->fetch();
        return $result[0];
    }
    
    /*
    public function update_server($server,$ip){
    $db=$this->r_db;
    $sql =<<<EOF
    update {$this->r_table} set server='{$server}' where ip='{ip}';
    EOF;
    $db->exec($sql);
    }
    */
    
    public function import($file) {
        if (FALSE !== ($fp = fopen($file, 'r'))) {
            $db = $this->r_db;
            $db->beginTransaction();
            while (!feof($fp)) {
                $buffer = trim(fgets($fp, 1024));
                list($host, $ip, $server) = preg_split('/\s+/', $buffer);
                $sql = "insert into {$this->r_table}('domain','ip','server') VALUES ('{$host}', '{$ip}','$server') ;";
                $db->exec($sql);
            }
            $db->commit();
            fclose($fp);
            return true;
        }
    }
    
    public function dump() {
        $results = $this->show_result();
        
        if (!empty($results) && is_array($results)) {
            foreach ($results as $key => $r) {
                file_put_contents($this->ResultFile, str_pad($r['domain'], 50, ' ') . str_pad($r['ip'], 50, ' ') . str_pad($r['server'], 100, ' ') . PHP_EOL, FILE_APPEND);
            }
            echo 'Dump Sussess! SaveTo: ' . $this->ResultFile;
        } 
        else {
            
            exit("Dump Fail..!  The Result is the empty... \r\n");
        }
    }
    
    public function __destruct() {
        $this->r_db = null;
    }
}
?>
```
