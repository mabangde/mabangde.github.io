---
title: WannaMiner 挖矿木马手工检测笔记
date: 2019-03-06 16:36:59
tags: 应急响应
---

### 发现可疑进程:

**外部扫描:**
> 该木马通常会开启65531-65533 端口

`nmap -p65531-65533 --open -oG d:\result1.txt 10.230.12.1/16`

**过滤出受影响IP：**
`grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" 230-result1.txt`

![2019-07-29-02-00-07](https://wolvez.oss-cn-hangzhou.aliyuncs.com/a0bb3745392df5dbf807ee805f07544f.png)

**恶意端口对应进程:**
![2019-07-29-02-00-33](https://wolvez.oss-cn-hangzhou.aliyuncs.com/e10abfa4791caddda43161a31ad7ddcb.png)
**恶意进程对应服务:**
![2019-07-29-02-01-22](https://wolvez.oss-cn-hangzhou.aliyuncs.com/f7118c5858256943273daa21ea0df961.png)
对应服务名:`snmpstorsrv`

**恶意服务详情**
![2019-07-29-02-02-05](https://wolvez.oss-cn-hangzhou.aliyuncs.com/f945b3d48d450d8441f20902a1f41289.png)

**服务加载模块**

![2019-07-29-02-02-28](https://wolvez.oss-cn-hangzhou.aliyuncs.com/d3827ee675618acd8b6682077206c65e.png)
加载dll:`snmpstorsrv.dll`

**加载恶意模块位置**

![2019-07-29-02-03-45](https://wolvez.oss-cn-hangzhou.aliyuncs.com/ea6cd096b4e4fccc20ee173d82b1ca5d.png)
dll位置:`C:\Windows\system32\snmpstorsrv.dll`

**dll模块对应的md5**

![2019-07-29-02-04-42](https://wolvez.oss-cn-hangzhou.aliyuncs.com/5c2f88b609b50a1c9f8e7d539fb5d839.png)

MD5:`42A12DE5A2B8CFF827407877DBD66B16`
![2019-07-29-02-05-07](https://wolvez.oss-cn-hangzhou.aliyuncs.com/5b23a7091fb761b47cb025d821b223a2.png)
360威胁情报查看确实有相应的威胁情报信息，并有相关安全报道

**查看文件创建日期**
`Get-Item C:\Windows\system32\snmpstorsrv.dll`

`2009/7/14      9:39`  修改过文件时间不准确


![2019-07-29-02-06-00](https://wolvez.oss-cn-hangzhou.aliyuncs.com/b571017d34845bd594d4388c429f02b0.png)
**导出注册表查看服务创建日期**
服务创建日期:`2018/11/16 - 18:17`

![2019-07-29-02-06-27](https://wolvez.oss-cn-hangzhou.aliyuncs.com/4c0d6f2cec5c3610d5f6c6de54e494ba.png)
![2019-07-29-02-06-51](https://wolvez.oss-cn-hangzhou.aliyuncs.com/c28f0ec08a9a7c2f6b215a6a07043c2d.png)

>更详细快捷的查杀建议使用**pchunter** **火绒剑** **auturuns**等安全工具
