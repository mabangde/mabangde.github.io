---
title: windows 7 小容量ssd硬盘解决方案
date: 2015-05-24 18:27
tags: 电脑基础
---

[文章发表于T00ls](https://www.t00ls.net/thread-28547-1-1.html)
[参考资料](http://www.tomshardware.com/faq/id-1934205/move-users-programfiles-x86-programdata-hdd-installing-windows-ssd-boot-drive.html)

超级本 通常都设计 得 小、薄、待机长、开机速度快
体积小 功耗小 就得牺牲 硬件性能
通常超级本 都配置一个 小容量的(固态硬盘) ssd 我测试的机器固态硬盘体积22GB 左右( thinkpad x230s)
完全不能愉快的 安装一个正常版的win7

我们可以把系统相关目录转移到 大的磁盘(机械)分区

c: 盘为系统盘 固态硬盘 (ssd)
d: 盘为软件安装盘 机械硬盘(hdd)

在安装系统时(非ghos安装)

切记：
最后一次重启，系统要求设置用户名和密码时候
按住 **shitf+f10** 进入命令行 执行下面命令

一定不要在 系统以安装完成作以下操作 否则会出现故障
> 以下命令纯手敲 未经过验证 非专业人士请勿使用
在使用过程中一定要注意盘符 有可能 d:为u盘
命令行 运行 **compmgmt.msc** 或**diskpart** 命令 进行磁盘管理 重新分配盘符
```
robocopy "C:\Users" "D:\Users" /E /COPYALL /XJ /V /ZB
robocopy "C:\Program Files" "D:\Program Files" /E /COPYALL /XJ /V /ZB
robocopy "C:\Program Files (x86)" "D:\Program Files (x86)" /E /COPYALL /XJ /V /ZB
robocopy "C:\ProgramData" "D:\ProgramData" /E /COPYALL /XJ /V /ZB
rmdir "C:\Users" /S /Q
rmdir "C:\Program Files" /S /Q
rmdir "C:\Program Files (x86)" /S /Q
rmdir "C:\ProgramData" /S /Q
mklink /J "C:\Users" "D:\Users"
mklink /J "C:\Program Files" "D:\Program Files"
mklink /J "C:\Program Files (x86)" "D:\Program Files (x86)"

reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion /v CommonFilesDir /t REG_SZ /d "d:\Program Files\Common Files" /f
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion /v "CommonFilesDir (x86)" /t REG_SZ /d "d:\Program Files (x86)\Common Files" /f
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion /v CommonW6432Dir /t REG_SZ /d "d:\Program Files\Common Files" /f
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion /v programfilesdir /t REG_SZ /d "d:\Program Files" /f
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion /v "programfilesdir (x86)" /t REG_SZ /d "d:\Program Files (x86)" /f
reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion /v programw6432dir /t REG_SZ /d "d:\Program Files" /f
reg add HKLM\SOFTWARE\Microsoft\Windows" "NT\CurrentVersion\ProfileList /v ProfilesDirectory /t REG_EXPAND_SZ /d d:\Users /f
reg add HKLM\SOFTWARE\Microsoft\Windows" "NT\CurrentVersion\ProfileList /v Default /t REG_EXPAND_SZ /d d:\Users\Default /f
reg add HKLM\SOFTWARE\Microsoft\Windows" "NT\CurrentVersion\ProfileList /v Public  /t REG_EXPAND_SZ /d d:\Users\Public /f
reg add HKLM\SOFTWARE\Microsoft\Windows" "NT\CurrentVersion\ProfileList /v ProgramData /t REG_EXPAND_SZ /d d:\ProgramData /f
```

进入系统后 以管理员权限 运行cmd

```
rmdir "C:\ProgramData" /S /Q

mklink /J "C:\ProgramData" "D:\ProgramData"
```
 

或以当前用户运行(未验证该方法)

```
takeown /f "c:\ProgramData" /r /d y  && icacls "c:\ProgramData" /grant %USERNAME%:F /T && rmdir c:\ProgramData /s /q &&  mklink /J "C:\ProgramData" "D:\ProgramData"
```
查看c 盘隐藏文件 我们会发现 有两个可恶的文件 占数GB 甚至数十GB 磁盘空间！
分别是 **pagefile.sys** **hiberfil.sys**

用如下方法 移动 或清理它
清理windows 7 休眠文件 关闭休眠


powercfg -h off
清理 移动虚拟内存文件 管理员 权限运行命令行
```
wmic pagefileset list /format:list 查看 虚拟内存设置
wmic pagefileset create name='d:\pagefile.sys',initialsize=0,maximumsize=0
wmic pagefileset where"name='c:\\pagefile.sys'" delete
```
