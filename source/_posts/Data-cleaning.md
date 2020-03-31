---
title: 数据清洗杂项
date: 2019-03-05 22:44:17
tags: regx sql
---

## 0x01 正则基础
### POSIX字符组
|POSIX字符组|说明|范围|
|:-------|:-|:-|
|`[[:alnum:]]`|字母字符和数字字符|`[a-zA-Z0-9]`|
|`[[:alpha:]]`|字母|`[a-zA-Z]`|
|`[[:ascii:]]`|ASCII字符|`[\x00-\x7F]`|
|`[[:blank:]]`|空格字符和制表符|`[ \t]`|
|`[[:cntrl:]]`|控制字符|`[\x00-\x1F\x7F]`|
|`[[:digit:]]`|数字字符|`[0-9]`|
|`[[:graph:]]`|空白字符之外的字符|`[\x21-\x7E]`|
|`[[:lower:]]`|小写字母字符|`[a-z]`|
|`[[:print:]]`|`[:graph:]`和空白字符|`[\x20-\x7E\]`|
|`[[:punct:]]`|标点符号|`[]!"\#$%&'\()*+,./:;<=>? @^_{\|\}~-]`|
|`[[:space:]]`|空白字符|`[ \t\r\n\v\f]`|
|`[[:upper:]]`|大写字母字符|`[A-Z]`|
|`[[:xdigit:]]`|十六进制字符|`[A-Fa-f0-9]`|




### 正反向预查
字符组|说明
:--:|:--:
`(?:pattern)`	|匹配pattern但不获取匹配结果，也就是说这是一个非获取匹配，不进行存储供以后使用。这在使用或字符`(\|)`来组合一个模式的各个部分是很有用。例如`industr(?:y\|ies)`就是一个比`industry\|industries`更简略的表达式。
`(?=pattern)`	|正向肯定预查，在任何匹配pattern的字符串开始处匹配查找字符串。这是一个非获取匹配，也就是说，该匹配不需要获取供以后使用。例如，`Windows(?=95\|98\|NT\|2000)`能匹配`Windows2000`中的`Windows`，但不能匹配`Windows3.1`中的`Windows`。预查不消耗字符，也就是说，在一个匹配发生后，在最后一次匹配之后立即开始下一次匹配的搜索，而不是从包含预查的字符之后开始。
`(?!pattern)`	|正向否定预查，在任何不匹配pattern的字符串开始处匹配查找字符串。这是一个非获取匹配，也就是说，该匹配不需要获取供以后使用。例如`Windows(?!95\|98\|NT\|2000)`能匹配`Windows3.1`中的`Windows`，但不能匹配`Windows2000`中的`Windows`。预查不消耗字符，也就是说，在一个匹配发生后，在最后一次匹配之后立即开始下一次匹配的搜索，而不是从包含预查的字符之后开始
`(?<=pattern)`|	反向肯定预查，与正向肯定预查类拟，只是方向相反。例如，`(?<=95\|98\|NT\|2000)Windows`能匹配`2000Windows`中的`Windows`，但不能匹配`3.1Windows`中的`Windows`。
`(?<!pattern)`|	反向否定预查，与正向否定预查类拟，只是方向相反。例如`(?<!95\|98\|NT\|2000)Windows`能匹配`3.1Windows`中的`Windows`，但不能匹配`2000Windows`中的`Windows`。

### 测试代码
```php
preg_match('#Windows(?=2000|XP|7)#', 'WindowsXP', $matchResult1);
preg_match('#Windows(?=2000|XP|7)#', 'Windows7', $matchResult2);
preg_match('#Windows(?=2000|XP|7)#', 'Windows2000', $matchResult3);
preg_match('#Windows(?=2000|XP|7)#', 'Windows2008', $matchResult4);

//高级点
preg_match('#Win\w+(?=2000|XP|7)#', 'Windows2000', $matchResult5);

preg_match('#Win\w+(?=2\d+)#', 'Windows2006', $matchResult6);
preg_match('#Win\w+(?=2\d+)#', 'Windows2007', $matchResult7);
preg_match('#Win\w+(?=2\d+)#', 'Windows2008', $matchResult8);
preg_match('#Win\w+(?=2\d+)#', 'Windows3008', $matchResult9);

header('Content-type:text/plain');
print_r([
	$matchResult1[0], //Windows
	$matchResult2[0], //Windows
	$matchResult3[0], //Windows
	$matchResult4, //匹配失败
	
	$matchResult5[0], //Windows
	$matchResult6[0], //Windows
	$matchResult7[0], //Windows
	$matchResult8[0], //匹配失败
	$matchResult9, //匹配失败
]);

```
### 反向预查

这个思维是相反的，比如`Windows(?!2000|XP|7)`不能匹配Windows7、Windows2000、WindowsXP里的Windows，却能匹配Windows2008里的Windows，例子就不做了，关键符号就是`?!`

## 0x02 字符处理
### 编码转换
>编码不正确会导致 各种莫名奇妙的问题 乱码 甚至 编辑的时候卡住 或 报错 
所有第一步就是要将转换编码
```bash
enca -L zh_CN.UTF-8 baidu.txt  #查看该文本编码
iconv -c -f GB2312 -t utf-8 #转换方法1
enca -L zh_CN.UTF-8 -c  #方法2 自动转换 
```

当然也可以直接

`iconv -c -f ASCII -t ASCII  test.txt`

### 转换成 unix 格式(转换换行符)
`dos2unix`  当然也可以用`tr` ,`sed`

导入数据报错`ERROR 1300 (HY000): Invalid utf8 character string:`
```sql
mysql> SET character_set_client = utf8mb4; 
mysql> SET character_set_results = utf8mb4; 
mysql> SET character_set_connection = utf8mb4; 
mysql> SET character_set_server = utf8mb4;
mysql> SET character_set_database = utf8mb4;

```

### 剔除无效字符及空行
```bash
sed -e 's/[\x01-\x19]//g' -e 's/\x00/ /g'  -e 's/[\x1a-\x1f]//g' -e 's/^[ \t]*//' -e 's/[ \t]*$//'
```
当然也可以使用
```bash
grep -a -v '[[:cntrl:]]' #处理不够干净 还是喜欢用上面的方法
```
### 去除重复
`sort baidu.log | uniq` #测试该方法是我所知的最快的 缺点就是只是按照开头字符排序
`awk "{print length($1),$1}" file.txt | sort -n | awk "{print $2}"`  字符串按照长度排序(短->长)   很慢


### 清洗案例
>如下数据中`152221111null` 由于导出数据库时替换问题缺少引号导致无法导入数据库
```sql
INSERT INTO OT_FORMH VALUES('554354353', null, '武汉武昌', '周杰伦', '152221111null,
'666666666666', null, 'HEHE', null, 'hahah', 'ddddd', null, null, null);
```

**修复后：**
```sql
INSERT INTO OT_FORMH VALUES('554354353', null, '武汉武昌', '周杰伦', '152221111',
'666666666666', null, 'HEHE', null, 'hahah', 'ddddd', null, null, null);
```

**语句**
```bash
sed -i "s/null\([[:alnum:]]\)/\1/" a.txt
```


**数据**
```sql
INSERT INTO OT_FORMH VALUES('54534543535', '302', '武汉武昌', '马培月', '6757657', '546565469879', null, null, null, null, 'fd', null7657657', null, null);
```
**`null7657657'`** 缺少引号,多一个null

**修复语句**
```bash
sed -i "s/null\([[:alnum:]]\)/\'\1/" a.txt
```


**问题数据**
```sql
INSERT INTO OT_FORMH VALUES('564654654', '302', '武汉华东师范大学,a栋302', '马晓晓', '15678123456', '999999', null, null, null, null, 'HUAWEI', 543535', null, null);
```
>**`0937536518'`** 缺少单引号，**武汉华东师范大学**`,`逗号也需要考虑

**修复命令**
```shell
cat a.txt |sed "s/,\s\([[:digit:]]\{5,\}',\)/ ,'\1/g"
```

名字如：`Tobey 's` 包含单引号造成sql执行错误
**问题数据**
```
INSERT INTO OT_FORMH VALUES('xxxxxxxx', '807', '厦门市', 'Tom's xxxxx', '777777', '0694013143', 'xxxxx', null, null, null, '100050', '073222138', null, null);
 INSERT INTO OT_FORMH VALUES('UEU117022513079', '106', 'ddddd Ce'Tive', '林月月', '09276576510', '800056033355', 'ddddddd', null, null, null, '200004', null, '丹东', null);
```
**修复命令**
```
cat f.txt |sed "s/\([^( ]\)'\([[:alpha:]]\)/\1 \2/g"
```
**涉及知识点:**
>`&` 元字符是有用的，但更有用的功能是能够在正则表达式中定义特定区域，通过定义正则表达式的特定的一部分，您可以引用字符引用这部分。
反向引用时,你必须首先定义一个区域,然后回顾这个区域。定义一个区域是在你感兴趣的区域插入\和括号。你周围的第一区域被通过`\1`引用,第二个地区用 `\2`引用，等等。


>替换所有不包含`;`符号换行，并重新换行
```bash
cat a.txt | sed ':a;N;$!ba;s/[\r\n]/ /g' |sed 's/\;/;\r\n/g' >a1.txt
```
**剔除多余的单引号**
```text
cat ff.txt
INSERT INTO CU_AAA VALUES('5345435', '65467457');
INSERT INTO CU_AAA VALUES('5435435', ''6546543');
INSERT INTO CU_AAA VALUES('5354354', '5345345');
INSERT INTO CU_AAA VALUES('543543543', '');
INSERT INTO CU_AAA VALUES('', '5435435');
```
```bash
cat tel1.txt|sed '/\'\')/!s/\'\'/\'/g' >tel2.txt
```

**剔除无意义字符**
```bash
sed -e 's/[^[:print:]]//g' -e 's/\xef//g' -e 's/\xbf//g' -e 's/\xbd//g' 
```
## 导入数据库
`mysql -uroot -proot -Ddddd --default_character_set utf8 <sql.txt`

### 常见grep匹配

查找email
```bash
grep -b -a -E -o "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,6}\b"
```
匹配 `<h1 class="page-title">`</h1>之间字符

```bash
grep -oP '(?<=<h1 class="page-title">).*?(?=<\/h1>)'
```



## 0x04 oracle aes解密
#### 使用mysql
```SQL
## 解密
SET block_encryption_mode = 'aes-256-cbc';
SET @init_vector = UNHEX('00000000000000000000000000000000');
SELECT AES_DECRYPT(unhex('39B84FD13BF16DC7813CGB3D4A113685'),upper(md5('key-text')),@init_vector);
##或直接key
SELECT AES_DECRYPT(unhex('39B84FD13BF16DC7813CGB3D4A113685'),'KEY0000000000000',@init_vector);

##加密
select HEX(AES_ENCRYPT('54353453453', 'KEY', @init_vector));
```

#### 使用php
```php
function encryptNew($str, $key)
{
    return bin2hex(openssl_encrypt($str, 'AES-256-ECB', $key, OPENSSL_RAW_DATA));
}

function decryptNew($encryptedStr, $key)
{
    return openssl_decrypt(hex2bin($encryptedStr), 'AES-256-ECB', $key, OPENSSL_RAW_DATA);
}
```


## 0x05 其他导库笔记

快速显示某一行：
`cat sql.txt | head -n 981 | tail -n +981`
`sed -n '5243,5245p'`

### mysql 相关
***mysql导入大数据处理***
`set global slave_net_timeout=3600;`
`SET GLOBAL max_allowed_packet=1073741824;select @@max_allowed_packet`

**快速导库**
>先导库后创索引要快得多
>load data infile 很快

`load data infile '/var/lib/mysql-files/CU_AAA.txt' IGNORE into table CUST_TEL fields terminated by '<terminated>';`
```sql
CREATE  TABLE `qqinfo`.`qqinfo` (
  `ID` INT NOT NULL ,
  `QQNum` INT NOT NULL ,
  `Nick` VARCHAR(20) NULL ,
  `QunNum` VARCHAR(45) NULL ,
  PRIMARY KEY (`ID`) )
ENGINE = MyISAM
DEFAULT CHARACTER SET = utf8
COLLATE = utf8_general_ci;

##修改字段编码
alter table `qqinfo`  CHANGE `Nick`  `Nick` VARCHAR(45)  CHARACTER SET utf8 COLLATE utf8_bin

load data infile 'H:\\mysql\\sql.txt' into table qqinfo fields terminated by ':' (ID,QQNum,Nick,QunNum);

##设置 所有编码 
set names utf8;
##查看编码
show variables like 'character%';


##修改数据库编码
alter database `qqinfo` DEFAULT CHARACTER SET utf8 COLLATE utf8_bin
alter database `qqinfo` DEFAULT CHARACTER SET gbk COLLATE gbk_chinese_ci
ALTER DATABASE `db_name` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;


##修改表编码
use qqinfo;
alter table `qqinfo` default character set utf8 collate utf8_bin;


##注：1.UTF8不要导入gbk，gbk不要导入UTF8;

##都在界面下设置好

1 hour 26 min 38.72 sec


CREATE INDEX ix_QQNum ON qqinfo (QQNum) 创建索引


CREATE TABLE `qqinfo` (
  `ID` int(11) NOT NULL,
  `QQNum` int(11) NOT NULL,
  `Nick` char(12)  DEFAULT NULL,
  `QunNum` int(11) DEFAULT NULL,
  PRIMARY KEY (`ID`)
) ENGINE=MyISAM DEFAULT CHARSET=gb2312;


CREATE INDEX idx_qqinfo ON qqinfo (QQNum,QunNum);

ALTER TABLE qqinfo ADD INDEX index_qqinfo_QQNum_QunNum (QQNum,QunNum);

set GLOBAL innodb_flush_log_at_trx_commit = 0

##先建索引，再执行set GLOBAL innodb_flush_log_at_trx_commit = 0;再倒入数据；



innodb_flush_log_at_trx_commit

innodb_buffer_pool_size
如果用Innodb，那么这是一个重要变量。相对于MyISAM来说，Innodb对于buffer size更敏感。MySIAM可能对于大数据量使用默认的key_buffer_size也还好，但Innodb在大数据量时用默认值就感觉在爬了。 Innodb的缓冲池会缓存数据和索引，所以不需要给系统的缓存留空间，如果只用Innodb，可以把这个值设为内存的70%-80%。和 key_buffer相同，如果数据量比较小也不怎么增加，那么不要把这个值设太高也可以提高内存的使用率。

innodb_additional_pool_size 
这个的效果不是很明显，至少是当操作系统能合理分配内存时。但你可能仍需要设成20M或更多一点以看Innodb会分配多少内存做其他用途。

innodb_log_file_size
对于写很多尤其是大数据量时非常重要。要注意，大的文件提供更高的性能，但数据库恢复时会用更多的时间。我一般用64M-512M，具体取决于服务器的空间。

innodb_log_buffer_size 
默认值对于多数中等写操作和事务短的运用都是可以的。如 果经常做更新或者使用了很多blob数据，应该增大这个值。但太大了也是浪费内存，因为1秒钟总会 flush（这个词的中文怎么说呢？）一次，所以不需要设到超过1秒的需求。8M-16M一般应该够了。小的运用可以设更小一点。

innodb_flush_log_at_trx_commit  （这个很管用） 
抱怨Innodb比MyISAM慢 
100倍？那么你大概是忘了调整这个值。默认值1的意思是每一次事务提交或事务外的指令都需要把日志写入（flush）硬盘，这是很费时的。特别是使用电 
池供电缓存（Battery backed up 
cache）时。设成2对于很多运用，特别是从MyISAM表转过来的是可以的，它的意思是不写入硬盘而是写入系统缓存。日志仍然会每秒flush到硬 
盘，所以你一般不会丢失超过1-2秒的更新。设成0会更快一点，但安全方面比较差，即使MySQL挂了也可能会丢失事务的数据。而值2只会在整个操作系统 
挂了时才可能丢数据。 


每个MyISAM在磁盘上存储成三个文件。第一个文件的名字以表的名字开始，扩展名指出文件类型。.frm文件存储表定义。数据文件的扩展名为.MYD (MYData)。索引文件的扩展名是.MYI (MYIndex)。

mysql 状态
SHOW STATUS like '_cache%'



mysql> show variables like '%tmp%';
+----------------------------+-------------------------------------------------+
| Variable_name              | Value                                           |
+----------------------------+-------------------------------------------------+
| default_tmp_storage_engine | InnoDB                                          |
| max_tmp_tables             | 32                                              |
| slave_load_tmpdir          | C:\Windows\SERVIC~2\NETWOR~1\AppData\Local\Temp |
| tmp_table_size             | 57671680                                        |
| tmpdir                     | C:\Windows\SERVIC~2\NETWOR~1\AppData\Local\Temp |
+----------------------------+-------------------------------------------------+


 show global status
 
 
 优化相关
 
 http://wenku.baidu.com/link?url=wPDRuX34e7Z8o-JJG5BT7Tu1gq43c5ZS2X8_2IRnlSrniYUTxMsPk_Hz8Pqw3FxFE0v3tiKO-Sr_Jumn6bZAIYbZMWo5BAFg9rJkJEfD75e
 http://blog.chinaunix.net/uid-24145780-id-125159.html
 
 复制表
 
CREATE TABLE `qqinfo_1` (
  `ID` int(11) NOT NULL AUTO_INCREMENT,
  `QQNum` int(11) NOT NULL,
  `Nick` char(20) DEFAULT NULL,
  `QunNum` int(11) DEFAULT NULL,
  PRIMARY KEY (`ID`),
  KEY `inx_qqinfo` (`QQNum`,`QunNum`) USING HASH
) ENGINE=MyISAM AUTO_INCREMENT=179780305 DEFAULT CHARSET=gbk DELAY_KEY_WRITE=1;


 insert into qqinfo_1 (`QQNum`,`Nick`,`QunNum`) select `QQNum`,`Nick`,`QunNum` from qqinfo.qqinfo; 



mysql 导库
指定编码不易出错

mysql -uroot --default-character-set=utf8 net < net.txt 

mysqldump -h 127.0.0.1 -uroot  -proot --opt --skip-lock-tables --default-character-set=utf8  net1 members >/data/www/pmxy/data/net1.txt


mysql中所有库查找包含某个字段的表 如 password

select table_schema,table_name,column_name from information_schema.columns where table_schema !=0x696E666F726D6174696F6E5F736368656D61 and table_schema !=0x6D7973716C and table_schema !=0x706572666F726D616E63655F736368656D61 and (column_name like '%pass%' or column_name like '%pwd%'); 


load data infile 比source  及直接导入sql 文件快无数倍！强烈建议使用


```

## 0x06 字符统计筛选
>以下内容来源t00ls.net @download
```bash
//整理字符集为键盘上的字符5-30位的密码
grep -o -P "[[:lower:][:upper:][:digit:][:punct:] ]+" 2014_1_last | awk '{if(length($0) > 4 && length($0) < 31) print $0}' >> 2014_1_last_

//删除1-7位的纯大写、1-7位的纯小写、1-9位的纯数字
sed "/^[[:upper:]]\{1,7\}$/d;/^[[:lower:]]\{1,7\}$/d;/^[[:digit:]]\{1,9\}$/d;" 2014_1_last_ >>2014_1_last__

//hashcat_masks 排名
sed 's/[[:lower:]]/l/g;s/[[:upper:]]/u/g;s/[[:digit:]]/d/g;s/[[:punct:] ]/s/g' 2014_1_last__ | sort | uniq -c | sort -nr >>2014_1_last__hashcat_masks_sus

//hashcat_masks 将luds前边加 ?
sed 's/\([luds]\)/?\0/g;' 2014_1_last__hashcat_masks_sus
 
//密码长度排名
1、awk '{printf("%s %s %s\r\n",$1,length($2),$2)}' 2014_1_last__hashcat_masks_sus >>2014_1_last__hashcat_masks_sus_length
2、awk '{a[$2]+=$1;}END{for(i in a){{print i" "a[i];}}}' 2014_1_last_x_hashcat_masks_sus_length | sort -nr -k2
11 25281160
10 15342231
12 11047379
14 9794974
13 9105853
9 8726583
15 6864187
8 6617661
16 6235360
7 1848134
6 899233
17 83551
18 40174
19 32202
21 12649
20 8847
22 2216
23 1824
24 1333
25 927
28 918
26 829
27 733
29 400
30 255
5 28

//长度百分比排名
$ awk '{a[$2]+=$1;}END{for(i in a){{print i" "a[i];}}}' 2014_1_last__hashcat_masks_sus_length | awk '{a[NR]=$1;b[NR]=$2;s+=$2}END{for(j=1;j<NR+1;j++) printf "%s\t%.5f%\n",a[j],b[j]*100/s}' 2014_1_last__length_sus | sort -nr -k2
11      24.79769%
10      15.04883%
12      10.83611%
14      9.60766%
13      8.93172%
9       8.55970%
15      6.73292%
8       6.49111%
16      6.11612%
7       1.81279%
6       0.88204%
17      0.08195%
18      0.03941%
19      0.03159%
21      0.01241%
20      0.00868%
22      0.00217%
23      0.00179%
24      0.00131%
25      0.00091%
28      0.00090%
26      0.00081%
27      0.00072%
29      0.00039%
30      0.00025%
5       0.00003%

-----------------------------------
//同时包含四种字符的排序
$ grep '[u]' 2014_1_last_x_hashcat_masks_sus | grep '[d]' | grep '[l]' | grep '[s]' | head -n 10 | nl
     1     1074 ullsdddd
     2      963 ulllsdddd
     3      834 ullsdddddd
     4      807 ulldddds
     5      800 ullllsdddd
     6      735 uldddddds
     7      735 ulddddddddddds
     8      715 ulldddddds
     9      703 ulddddddds
    10      687 ullldddds

//同时包含大小写数字的排序
$ grep '[u]' 2014_1_last__hashcat_masks_sus | grep '[d]' | grep '[l]' | grep '[^s]' | head -n 10 | nl
     1    14383 ulldddddd
     2    13679 uldddddd
     3    11557 ulldddddddd
     4    11054 uldddddddd
     5    11003 ulddddddddddd
     6    10521 ullldddd
     7    10099 ullddddddd
     8    10060 ullddddddddddd
     9     9967 ulddddddd
    10     9264 ulllldddd

//同时包含大小写的排序
$ grep '[u]' 2014_1_last__hashcat_masks_sus | grep '[l]' | grep '[^d]' | grep '[^s]' | head -n 10 | nl
     1    14383 ulldddddd
     2    13679 uldddddd
     3    11557 ulldddddddd
     4    11054 uldddddddd
     5    11003 ulddddddddddd
     6    10521 ullldddd
     7    10099 ullddddddd
     8    10060 ullddddddddddd
     9     9967 ulddddddd
    10     9264 ulllldddd


//连续的纯字符的排序
$ sed -n '/\b\([luds]\)\1*$/p' 2014_1_last__hashcat_masks_sus
16406454 ddddddddddd
5475964 dddddddddd
2129702 dddddddddddd
1189570 dddddddddddddd
 822811 llllllllll
 734170 lllllllll
 658042 ddddddddddddd
 653511 llllllll
 643282 lllllllllll
 606358 dddddddddddddddd
 570914 llllllllllll

//统计开头的第一个字母的数量排序
$ awk '{a[substr($3,1,1)]+=$1;}END{for(i in a){{print i" "a[i];}}}' 2014_1_last__hashcat_masks_sus_length | sort -nr -k2

//统计开头的第二个字母的数量排序
$ awk '{a[substr($3,2,1)]+=$1;}END{for(i in a){{print i" "a[i];}}}' 2014_1_last__hashcat_masks_sus_length | sort -nr -k2

//统计开头的第三个字母的数量排序
$ awk '{a[substr($3,3,1)]+=$1;}END{for(i in a){{print i" "a[i];}}}' 2014_1_last__hashcat_masks_sus_length | sort -nr -k2

//统计开头的前二个字母的数量排序
$ awk '{a[substr($3,1,2)]+=$1;}END{for(i in a){{print i" "a[i];}}}' 2014_1_last__hashcat_masks_sus_length | sort -nr -k2

//统计开头的前三个字母的数量排序
$ awk '{a[substr($3,1,3)]+=$1;}END{for(i in a){{print i" "a[i];}}}' 2014_1_last_x_hashcat_masks_sus_length | sort -nr -k2

//统计开头的前四个字母的数量排序
$ awk '{a[substr($3,1,4)]+=$1;}END{for(i in a){{print i" "a[i];}}}' 2014_1_last__hashcat_masks_sus_length | sort -nr -k2

//统计开头的最后一个字母的数量排序
$ awk '{a[substr($3,length($3),1)]+=$1;}END{for(i in a){{print i" "a[i];}}}' 2014_1_last__hashcat_masks_sus_length | sort -nr -k2

//提取特定字符集的字典

grep -P "^[[:lower:]]+$" 2014_1_last__ >>2014_1_last__lower
grep -P "^[[:upper:]]+$" 2014_1_last__ >>2014_1_last__upper
grep -P "^[[:digit:]]+$" 2014_1_last__ >>2014_1_last__digit
grep -P "^[[:punct:] ]+$" 2014_1_last__ >>2014_1_last__punct
grep -P "^[[:lower:][:upper:]]+$" 2014_1_last__ | grep -P -v "^[[:lower:]]+$" | grep -P -v "^[[:upper:]]+$" >>2014_1_last__lowerupper
grep -P "^[[:upper:][:punct:] ]+$" 2014_1_last__ | grep -P -v "^[[:upper:]]+$" | grep -P -v "^[[:punct:] ]+$" >>2014_1_last__upperpunct
grep -P "^[[:lower:][:punct:] ]+$" 2014_1_last__ | grep -P -v "^[[:lower:]]+$" | grep -P -v "^[[:punct:] ]+$" >>2014_1_last__lowerpunct
grep -P "^[[:lower:][:digit:]]+$" 2014_1_last__ | grep -P -v "^[[:lower:]]+$" | grep -P -v "^[[:digit:]]+$" >>2014_1_last__lowerdigit
grep -P "^[[:upper:][:digit:]]+$" 2014_1_last__ | grep -P -v "^[[:upper:]]+$" | grep -P -v "^[[:digit:]]+$" >>2014_1_last__upperdigit
grep -P "^[[:digit:][:punct:] ]+$" 2014_1_last__ | grep -P -v "^[[:digit:]]+$" | grep -P -v "^[[:punct:] ]+$" >>2014_1_last__digitpunct
grep -P "^[[:lower:][:digit:][:punct:] ]+$" 2014_1_last__ | grep -P -v "^[[:lower:]]+$" | grep -P -v "^[[:digit:]]+$" | grep -P -v "^[[:punct:] ]+$" | grep -P -v "^[[:lower:][:digit:]]+$" | grep -P -v "^[[:lower:][:punct:] ]+$" | grep -P -v "^[[:digit:][:punct:] ]+$">>2014_1_last__lowerdigitpunct
grep -P "^[[:upper:][:digit:][:punct:] ]+$" 2014_1_last__ | grep -P -v "^[[:upper:]]+$" | grep -P -v "^[[:digit:]]+$" | grep -P -v "^[[:punct:] ]+$" | grep -P -v "^[[:upper:][:digit:]]+$" | grep -P -v "^[[:upper:][:punct:] ]+$" | grep -P -v "^[[:digit:][:punct:] ]+$">>2014_1_last__upperdigitpunct
grep -P "^[[:lower:][:upper:][:digit:]]+$" 2014_1_last__ | grep -P -v "^[[:lower:]]+$" | grep -P -v "^[[:upper:]]+$" | grep -P -v "^[[:digit:]]+$" | grep -P -v "^[[:lower:][:upper:]]+$" | grep -P -v "^[[:lower:][:digit:]]+$" | grep -P -v "^[[:upper:][:digit:]]+$" >>2014_1_last__lowerupperdigit
grep -P "^[[:lower:][:upper:][:punct:] ]+$" 2014_1_last__ | grep -P -v "^[[:lower:][:upper:]]+$" | grep -P -v "^[[:lower:]]+$"  | grep -P -v "^[[:upper:]]+$" | grep -P -v "^[[:punct:] ]+$" | grep -P -v "^[[:lower:][:punct:] ]+$" | grep -P -v "^[[:upper:][:punct:] ]+$" | grep -P -v "^[[:lower:][:upper:]]+$" >>2014_1_last__lowerupperpunct
grep -P "^[[:lower:][:upper:][:digit:][:punct:] ]+$" 2014_1_last__ | grep -P -v "^[[:alnum:]]+$" | grep -P -v "^[[:punct:] ]+$" | grep -P -v "^[[:lower:][:punct:] ]+$" | grep -P -v "^[[:upper:][:punct:] ]+$" | grep -P -v "^[[:digit:][:punct:] ]+$" | grep -P -v "^[[:upper:][:digit:][:punct:] ]+$" | grep -P -v "^[[:lower:][:digit:][:punct:] ]+$" | grep -P -v "^[[:lower:][:upper:][:punct:] ]+$" >>2014_1_last__lowerupperdigitpunct

//用户名和密码组合
$ cat user.txt
root
manager

$ cat pass.txt
123
456
asd
password
admin

$ awk 'NR==FNR{a[FNR]=$1;num=FNR}NR>FNR{for(i=1;i<=num;i++){print a[i] ":" $1}}' user.txt pass.txt
root:123
manager:123
root:456
manager:456
root:asd
manager:asd
root:password
manager:password
root:admin
manager:admin
```
