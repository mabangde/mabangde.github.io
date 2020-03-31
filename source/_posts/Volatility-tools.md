---
title: 应急分析取证工具篇
date: 2019-01-26 23:16
tags: 应急响应
---

>应急响应笔记持续更新中，大部分内容都是工作上遇到的，欢迎讨论

### 1. 共享访问监控
- 功能：查看连接
- 场景：某机器提供共享，共享文件夹一直报毒，使用该工具可快速知道哪些机器访问了该机器共享
- 下载地址：[networkopenedfiles-x64.zip](http://www.nirsoft.net/utils/networkopenedfiles-x64.zip)

**使用效果如下：**
![2019-07-29-02-42-37](https://wolvez.oss-cn-hangzhou.aliyuncs.com/cf74d7647803cf1df3e9681447abdb5a.png)

**设置存储日志：**
![2019-07-29-02-42-58](https://wolvez.oss-cn-hangzhou.aliyuncs.com/4ae41839c00bf802df062cc0b8f888ea.png)

>ps:后来发现windows 自带共享会话监视功能

![2019-07-29-02-43-27](https://wolvez.oss-cn-hangzhou.aliyuncs.com/492f6a446ac1ccdbb1bb9785f3e7524a.png)
**其它:**
组策略中开启**审核对象访问**,并对**共享目录**中添加对应的审核账号的写入属性**成功**和**删除**的审核,即可在操作系统安全日志中看到相关记录
![2019-07-29-02-39-55](https://wolvez.oss-cn-hangzhou.aliyuncs.com/f5951b7fa73a21e77638f9c64e473c3b.png)

### 2. wifi使用记录
- 功能：查看使用wifi历史记录
- 场景：看该电脑是否有连接过其它无线网络，以证明非本内网络导致感染
- 下载地址：[wifihistoryview](http://www.nirsoft.net/utils/wifihistoryview.zip)
  
![2019-07-29-02-43-57](https://wolvez.oss-cn-hangzhou.aliyuncs.com/1e444bd284979447c123f80e7945693c.png)

### 3. usb使用记录
- 功能：查看插入或拔出系统的任何USB设备的详细信息。
- 场景：查看是否为u盘传输导致感染
- 下载地址：[usblogview.zip](https://www.nirsoft.net/utils/usblogview.zip)

![2019-07-29-02-44-19](https://wolvez.oss-cn-hangzhou.aliyuncs.com/83dbf2e863f4b81328df571368e1e3de.png)

### 4. 内存取证
- 内存镜像工具 DumpIt.exe
- 介绍：DumpIt 是一款绿色免安装的 windows 内存镜像取证工具。利用它我们可以轻松地将一个系统的完整内存镜像下来，并用于后续的调查取证工作。
  
![突破来源:即刻安全](https://wolvez.oss-cn-hangzhou.aliyuncs.com/57b13d67c449a4294454557d7002fed3.png)
>下载地址 [Comae-Toolkit](https://my.comae.io/tools)
- http://www.secist.com/archives/2076.html
- http://blog.md5.red/?p=578

- raw 内存取证
- github:https://github.com/volatilityfoundation
- exe下载:https://www.volatilityfoundation.org/releases
  
```
python vol.py -f /root/lltest/PC-20170527XAOD-20180409-232828.raw --profile=Win7SP1x86 pslist
python vol.py -f /root/lltest/PC-20170527XAOD-20180409-232828.raw --profile=Win7SP1x86 psxview
```
![2019-07-29-02-45-40](https://wolvez.oss-cn-hangzhou.aliyuncs.com/ee03222c2bccbf497ae37f7173ec6a53.png)

- 参考:https://xz.aliyun.com/t/2497


### 5. Ghost 镜像浏览
- ghost-explorer 
- 系统比较复杂不便现场分析，可备份ghost 文件，后面用ghost-explorer 进行提取相关文件
- 背景：一次应急过程，客户铲掉了受感染系统，而这台服务器刚好是关键线索。幸好客户前面做了ghost备份。后面通过ghost-explorer  提取ghost 镜像注册表、日志信息成功溯源。

- 官网 https://www.symantec.com/connect/blogs/ghost-explorer 
