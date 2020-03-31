---
title: wmi一些操作
date: 2017-01-06 01:06
tags: pentest
---

**使用wmic识别安装到系统中的补丁情况**

```bat
C:\> wmic qfe get description,installedOn
```
**外部调用获取补丁情况**
```bat
select * from Win32_QuickFixEngineering
SELECT * FROM Win32_OperatingSystemQFE
```

**识别正在运行的服务**
```bat
C:\>sc query type= service
C:\>net start
```

**识别开机启动的程序，包括路径**
```bat
C:\>wmic startup list full
```

**查看系统中网卡的IP地址和MAC地址**
```bat
D:\>wmic nicconfig get ipaddress,macaddress
```
**用户列表**
```bat
D:\>wmic useraccount list brief
```
**查看当前系统是否有屏保保护，延迟是多少**
```bat
D:\>wmic desktop get screensaversecure,screensavertimeout
```

**域控机器**
```bat
D:\>wmic ntdomain list brief
```

**登录用户**
```bat
D:\>wmic logon list brief
```


**查看系统中开放的共享**
```bat
D:\>wmic share get name,path
D:\>net share
```
**卸载和重新安装程序**

```
wmic product where "name like '%Office%'" get name
wmic product where name="Office" call uninstall
```
[来源:](https://uknowsec.cn/posts/notes/%E6%9D%83%E9%99%90%E7%BB%B4%E6%8C%81-WMI.html)
**查看系统中开启的日志**
```bat
C:\>wmic nteventlog get path,filename,writeable
```

**清除相关的日志（这里是全部清除）**
```bat
wevtutil cl "windows powershell"
wevtutil cl "security"
wevtutil cl "system"
```
*博主注:建议使用tr 或sed 或其他方法替换关键字符
**查看系统中安装的软件以及版本**
```bat
C:\>wmic product get name,version
```
**查看某个进程的详细信息 （路径，命令行参数等）**
```
C:\>wmic process where name="chrome.exe" list full
```
**终止一个进程**
```bat
D:\>wmic process where name="xshell.exe" call terminate
D:\>ntsd -c q -p 进程的PID
D:\>taskkill -im pid
```
**获取存储在注册表中所有包含密码的键值：**
```bat
REG query HKCU  /v "pwd" /s  #pwd可替换为password \ HKCU 可替换为HKCR
```
**显示系统中的曾经连接过的无线密码**
```bat
D:\>netsh wlan show profiles 
D:\>netsh wlan show profiles name="profiles的名字" key=clear
```
**博主首发:**
一键获取:
```bat
for /f "skip=9 tokens=1,2 delims=:" %i in ('netsh wlan show profiles') do @echo %j | findstr -i -v echo | netsh wlan show profiles %j key=clear
```
**查看当前系统是否是VMWARE**
```bat
C:\>wmic bios list full | find /i "vmware"
```
**获取进程服务名称 PID**
```bat
tasklist /svc | findstr "TermService" 
netstat -ano | findstr "PID"
```

**杀毒软件**
```powershell
Get-WmiObject -Namespace root\SecurityCenter2 -Class AntiVirusProduct
```
**虚拟机检测**
1. 判断TotalPhysicalMemory和NumberOfLogicalProcessors
```powershell
$VMDetected = $False
$Arguments = @{
 Class = 'Win32_ComputerSystem'
 Filter = 'NumberOfLogicalProcessors < 2 AND TotalPhysicalMemory < 2147483648'
}
if (Get-WmiObject @Arguments) { 
$VMDetected = $True
"In vm"
 } 
 else{
 "Not in vm"
 }
```
2. 判断虚拟机进程
```powershell
$VMwareDetected = $False
$VMAdapter = Get-WmiObject Win32_NetworkAdapter -Filter 'Manufacturer LIKE
"%VMware%" OR Name LIKE "%VMware%"'
$VMBios = Get-WmiObject Win32_BIOS -Filter 'SerialNumber LIKE "%VMware%"'
$VMToolsRunning = Get-WmiObject Win32_Process -Filter 'Name="vmtoolsd.exe"'
if ($VMAdapter -or $VMBios -or $VMToolsRunning) 
{ $VMwareDetected = $True 
"in vm"
} 
else
{
"not in vm"
}
```
[来源:权限维持-WMI](https://uknowsec.cn/posts/notes/%E6%9D%83%E9%99%90%E7%BB%B4%E6%8C%81-WMI.html)

**获取电脑产品编号和型号信息**
```bat
wmic baseboard get Product,SerialNumber
wmic bios get serialnumber
```
**安装软件**
```bat
wmic product get name,version
wmic product list brief
```
**程序运行时间**
```bat
wmic process get CreationDate
```
**检查服务路径中包含空格且没有双引号的服务**
*博主首发
```bat
wmic service where   "((state='running') and  (pathname like '% %') and not (pathname like '%\"%') and not (pathname like '%system32%') and not (pathname like '%syswow64%'))"  get pathname,name,displayname,startname
```
