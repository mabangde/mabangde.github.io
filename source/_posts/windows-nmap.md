---
title: win下编译最新单文件nmap
date: 2017-6-4 23:39
tags: 计算机基础
---

nmap 功能非常强大一直是广大安全测试人员最喜爱工具之一
渗透测试过程中很多时候目标环境不方便安装nmap 所以就需要编译一份单文件版nmap
支持低版本windows(winxp\win2003)
本人代码功底非常粗浅，所遇到问题大多通过google找到解决方案的写这篇文章目的是当作个人笔记，如果想喷请有建设性的喷，欢迎指出不足，或加以改良(剔除不必要的库和功能以减少体积)

运行截图:
![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/nmap/1.PNG)

按照常规的方式静态编译 拿到第三方系统运行发现缺少openssl 相关库
查询原因原来是 openssl 库需要单独静态编译

静态编译openssl 请参考：
http://www.jianshu.com/p/4522f17ce2ff
http://seclists.org/nmap-dev/2011/q2/att-1090/ncat-portable-static.txt

简要说明下：
在Windows环境下编译openssl需要perl支持，安装ActivePerl
[下载OpenSSL源码](https://www.openssl.org/source/openssl-1.0.1f.tar.gz)

我们用**VS2013**来作为编译工具，打开命令行，切换到bin目录，比如

```bat
shell> cd d:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin
shell> vcvars32.bat
shell> perl Configure --prefix=C:/OpenSSL VC-WIN32 no-asm  -no-shared
shell> ms\do_ms.bat
shell> nmake -f ms\nt.mak install
```
此处：附上以静态编译好了的openssl静态库 免去安装编译环境的麻烦

[OpenSSL静态库下载](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/nmap/OpenSSL.zip)


![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/nmap/2.PNG)

配置包含目录及编译好的**openssl**库目录
![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/nmap/3.PNG)

设置平台工具集 使编译的文件支持低版本windows
![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/nmap/4.PNG)

**设置静态编译**
>说明：
>多线程（MT） 与 多线程调试（MTd）貌似一样，都没有MSVCP和MSVCR函数导入，只有Kernel32.dll。同时观察这两个文件的体积，都比MD或MDd大了很多，这正是它们不需要导入运行时库DLL函数的原因，因为它们把运行时库静态编译到自己的文件中去了。这也代表着它们运行的时候不会再依赖外部的运行时库DLL文件。

[来源](https://www.zhihu.com/question/25415940)

**这里编译会遇到几个坑：**

**第一个坑：**
报错：
>1>libeay32.lib(cryptlib.obj) : error LNK2019: unresolved external symbol __imp__GetUserObjectInformationW@20 referenced in function _OPENSSL_isservice1>libeay32.lib(cryptlib.obj) : error LNK2019: unresolved external symbol __imp__GetProcessWindowStation@0 referenced in function _OPENSSL_isservice1>libeay32.lib(cryptlib.obj) : error LNK2019: unresolved external symbol __imp__GetDesktopWindow@0 referenced in function _OPENSSL_isservice1>libeay32.lib(cryptlib.obj) : error LNK2019: unresolved external symbol __imp__MessageBoxA@16 referenced in function _OPENSSL_showfatal


解决办法
附加依赖项添加
**user32.lib**
**gdi32.lib**

![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/nmap/5.PNG)

[参考来源](http://seclists.org/nmap-dev/2011/q2/att-1090/ncat-portable-static.txt)

**第二个坑**

选release编译，在链接的时候出问题了：
>7>LIBCMT.lib(crt0init.obj) : warning LNK4098: 默认库“libcmtd.lib”与其他库的使用冲突；请使用 /NODEFAULTLIB:library
7>LIBCMT.lib(crt0init.obj) : warning LNK4098: 默认库“msvcrt.lib”与其他库的使用冲突；请使用 /NODEFAULTLIB:library

解决办法：

在项目连接器里面忽略两个lib就可以过了
**libcmtd.lib** 和 **libcmt.lib**

![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/nmap/6.PNG)

**第三个坑：**
>Error lnk2026: module unsafe for safeseh image

解决办法：

**stackoverflow** 上找到解决办法

>Right-click on your project ->
Properties ->
Configuration Properties ->
Linker ->
Advanced and changed "Image Has Safe Exception Handlers" to "No (/SAFESEH:NO)"


对应中文版配置如下：
![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/nmap/7.PNG)

解决方法二：
>在项目的“属性页”中找到“链接器”标签，然后点击“命令行”将/SAFESEH:NO添加到“附加选项”的框中，点击应用即可。

原始单文件nmap 下载：
[nmap原始单文件](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/nmap/nmap.zip)
大小:  2.25 MB 
MD5: a7628eb98bc1b705fb3099c2c6857e02

nmap 剔除openssl lua  win7 测试通过

[nmap瘦身版](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/nmap/nmap_mt_upx.zip)
大小: 486 KB
MD5: b518dba54aac5caa3ee29647411212dd

**已知问题：**
win2k3\winxp 报错：
在winxp 或win2k3 下编译 即可解决
由于没有winxp 开发环境，并未解决，期待大家来解决
![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/nmap/8.PNG)
