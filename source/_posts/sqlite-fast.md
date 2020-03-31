---
title: sqlite 高速插入文本文件
date: 2015-05-24 15:15
tags: 数据优化
---
文章首发：
[t00ls](https://www.t00ls.net/thread-29905-1-1.html)
[本文参考文章](http://blog.csdn.net/chenguanzhou123/article/details/9376537)
为什么要用**sqlite** ?
1、想将数据放进移动存储设备方便随时随地查询
2、**sqlite** 跨平台 ，无需服务，仅一个可执行文件即可

需解决的问题：
sqlite 按常规插入数据是非常慢的，不适合插入大数据
采用下面的方法后 速度 大大的增加，数G文本 入库都 非常快(相对)

采用技术：
1、关闭写同步(**synchronous**)
说明：

简要说来，full写入速度最慢，但保证数据是安全的，不受断电、系统崩溃等影响，而off可以加速数据库的一些操作，但如果系统崩溃或断电，则数据库可能会损毁。
SQLite3中，该选项的默认值就是full，如果我们再插入数据前将其改为off，则会提高效率。如果仅仅将SQLite当做一种临时数据库的话，完全没必要设置为full

2、执行准备 **prepare**

说明：

SQLite执行SQL语句的时候，有两种方式：一种是使用前文提到的函数sqlite3_exec()，该函数直接调用包含SQL语句的字符串；另一种方法就是“执行准备”（类似于存储过程）操作，即先将SQL语句编译好，然后再一步一步（或一行一行）地执行。如果采用前者的话，就算开起了事务，SQLite仍然要对循环中每一句SQL语句进行“词法分析”和“语法分析”，这对于同时插入大量数据的操作来说，简直就是浪费时间。因此，要进一步提高插入效率的话，就应该使用后者。

3、显式开启事务 **BEGIN…COMMIT**
说明：

SQLite的数据库本质上来讲就是一个磁盘上的文件，所以一切的数据库操作其实都会转化为对文件的操作，而频繁的文件操作将会是一个很好时的过程，会极大地影响数据库存取的速度。
        采用该种方法 就是将全部要执行的SQL语句先缓存在内存当中，然后等到COMMIT的时候一次性的写入数据库，这样数据库文件只被打开关闭了一次，效率自然大大的提高。
4、查询的话普通索引依旧很慢 我们可以采用fulltext 索引 速度比mysql快

下面我们将1千万数据插入sqlite 看看消耗多少时间
`php test.php`
![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/buhuo.png)

测试代码：
```php
<?php
$t1 = microtime(true);
ini_set("memory_limit",'-1');
class sqllite{
    private $r_db;
    private $r_table='test';   
       
public function __construct() {
    try{
    $this->r_db=new PDO('sqlite:test.db3');
$sql =<<<EOF
PRAGMA temp_store = memory;
PRAGMA locking_mode = EXCLUSIVE;
PRAGMA synchronous = OFF;
PRAGMA cache_size = 400000;
PRAGMA page_size = 4096;
PRAGMA auto_vacuum=NONE;
PRAGMA count_changes = OFF;
PRAGMA journal_mode = OFF;

CREATE TABLE IF NOT EXISTS {$this->r_table} (
        'id'  INTEGER primary key AUTOINCREMENT NOT NULL,
        'test' char(15) NOT NULL
);
EOF;
//exit($sql.PHP_EOL);
$this->r_db->exec($sql);
    } catch (PDOException $ex){
        die ("[Error]  ".$ex->getMessage() .PHP_EOL.
        '         sudo apt-get install php5-sqlite php5-pdo'.PHP_EOL.
        '         add/edit in php.ini'.PHP_EOL.
        '         extension=pdo.so or pdo_sqlite.dll'.PHP_EOL
        );
    }
 }
 



public function show_count(){
$sql =<<<EOF
SELECT count(id) from {$this->r_table};
EOF;

$result=$this->r_db->prepare($sql);
$result->execute();
$result= $result->fetch();
return $result[0];
}

public function deleteTable(){
$db=$this->r_db;   
$sql =<<<EOF
drop table if exists {$this->r_table};
drop INDEX if exists qq_idx;
delete from sqlite_sequence;
vacuum;
EOF;
$db->exec($sql);
}


public function import(){
$db=$this->r_db;
$db->beginTransaction();
for ($i=1;$i<=10000000;$i++){
$sql ="insert into {$this->r_table}('test') VALUES ('{$i}');";
//echo $sql.PHP_EOL;
$db->exec($sql);
}
$db->commit();
}
public function __destruct() {
   $this->r_db=null;      
    }

}   


$sql=new sqllite();
$sql->import();
$t2 = microtime(true);
echo '插入 10000000 条数据'.PHP_EOL;
echo '耗时'.round($t2-$t1,3).'秒';
?>
```
下面的例子就是将某文本文件(某数据库)插入数据库
建议先用shell命令 处理好 再入库 减少php操作 (如剔除 不规则字符 、转换编码 格式规范)
awk sed grep…

```php
<?php
ini_set("memory_limit",'-1');
$table='sgk';

$txt='moko.txt';  //要导入的文本
$from='moko';    //数据来源
//$db=$from.'.db3';  //测试临时库名

empty($db)?$db='sgk.db3':$db;

//$index="{$table}_idx";

$db = new SQLite3($db);
$s =<<<EOF
PRAGMA temp_store = memory;
PRAGMA locking_mode = EXCLUSIVE;
PRAGMA synchronous = OFF;
PRAGMA cache_size = 400000;
PRAGMA page_size = 4096;
PRAGMA auto_vacuum=NONE;
PRAGMA count_changes = OFF;
PRAGMA journal_mode = OFF;

CREATE TABLE IF NOT EXISTS {$table} (
        'id'  INTEGER primary key AUTOINCREMENT NOT NULL,
        'uname' char(15) NOT NULL ,
        'pass'  char(25)  NOT NULL,
'from'  char(15) NULL
);

EOF;

$db->exec($s);

$sql ="insert into {$table}('uname','pass','from') VALUES (?,?,'{$from}');";

$stmt = $db->prepare($sql);

$db->exec("BEGIN");
if(FALSE !==($fp=fopen($txt,'r'))){
while(!feof($fp)){
$buffer=rtrim(fgets($fp));

@list($email,$uname,$pass)=@explode("----",$buffer);
@$stmt->reset();
@$stmt->clear();
$stmt->bindValue(1, $email."\t".$uname, SQLITE3_TEXT);        
$stmt->bindValue(2, $pass, SQLITE3_TEXT);
@$stmt->execute();        
}
$db->exec("COMMIT");
}

?>
```
其实本文就是提供搭建 可移动社工库的一个思路。

