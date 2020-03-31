---
title: 应急响应笔记 windows 常用命令记录
date: 2019-2-12 13:12:02
tags: 应急响应
---

>以下案例均为项目中遇到的，欢迎补充,文章随时更新

### 0x01 删除指定hash值的文件
需求：
已知恶意文件md5值为：`59B18D6146A2AA066F661599C496090D`、`6FF97A7DABF09EBB07C157F286DC81AD` ，需要全部删除。

代码如下：
```powershell
[array]$md5=Get-FileHash .\*.exe -Algorithm md5
$md5 | Where -Property Hash -in -Value "59B18D6146A2AA066F661599C496090D","6FF97A7DABF09EBB07C157F286DC81AD"
 | Remove-Item
```
**例图：**
![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/1103363282.png)
>ps:低版本powershell不支持，以下代码为通用获取md5函数

```powershell
function Get-md5 {
  Param($path)
  if(Test-Path $path -PathType Leaf ){
  $md5file=certutil -hashfile $path MD5
  [string]$hash=$md5file -match "^[a-f0-9]{32}$"
  $retVal = New-Object -TypeName psobject -Property @{
                    PATH = $path
                    Hash = $hash
                }
                $retVal

  }else{
  write-host [-] 'Get-md5' c:\windows\system32\calc.exe -ForegroundColor Red
  break
  }
}
```

![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/3939328751.png)
cmd下获取md5:
`certutil -hashfile c:\windows\system32\cmd.exe MD5 |findstr /r "^[a-fA-F0-9]*$"`

>支持powershell 2.0 的get-hash
https://gist.github.com/jaredcatkinson/7d561b553a04501238f8e4f061f112b7


![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/287109341.png)

**案例：搜索系统目录下恶意文件**

```powershell
function Get-md5 {
  Param($path)
  if(Test-Path $path -PathType Leaf ){
  $md5file=certutil -hashfile "$path" MD5
  [string]$hash=$md5file -match "^[a-f0-9]{32}$"
  $retVal = New-Object -TypeName psobject -Property @{
                    Path = $path
                    Hash = $hash.ToUpper()
                }
                $retVal

  }else{
  }
}

[array]$md5=Get-ChildItem  $env:SystemRoot  -ErrorAction 0  -Force -recurse -Filter *.exe | % {Get-md5  $_.FullName}
$md5  | where {($_.Hash -eq "6983F7001DE10F4D19FC2D794C3EB534" -or  $_.Hash -eq "59B18D6146A2AA066F661599C496090D" -or  $_.Hash -eq "CCE36235A525858EB55070847296C4C8" -or  $_.Hash -eq "9911858E9BEC100CCF6D8134915103EC" -or  $_.Hash -eq "74E2A43B2B7C6E258B3A3FC2516C1235" -or  $_.Hash -eq "D4E2EBCF92CF1B2E759FF7CE1F5688CA" -or  $_.Hash -eq "FB89D40E24F5FF55228C38B2B07B2E77" -or  $_.Hash -eq "637BF46077AD083659D3B96A010F38FE")} | %{write-host $_Path}
[array]$md5=@()
```
![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/2058851295.png)
### 0x02 寻找某一日期创建的文件

需求：得知某一日期遭攻击，想列出攻击日期内产生的文件

```cmd
forfiles /m *.exe /d +2019/2/12 /s /p c:\  /c "cmd /c echo @path @fdate @ftime" 2>nul
```

**例图**

![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/280331542.png)
>forfiles是一个很强大的命令，windows下有非常详细的帮助，这里就不赘述用法了。

### 0x03 wmi无文件后门检测
```powershell
Get-WmiObject -Namespace root\default -list | Where-Object {$_.name -Match "^[a-z]"}
Get-WmiObject -Namespace root\subscription -class commandlineeventconsumer
Get-WmiObject -Namespace root\subscription -class __eventfilter
Get-WmiObject -Namespace root\subscription -class __FilterToConsumerBinding
```
图例：
![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/1023506850.png)
![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/1550764772.png)
![111.png](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/2955844708.png)

![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/1129965653.png)

![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/4279610697.png)
### 0x04 powershell解码

>需求：遇到powershell -enc 方式执行需要解码

**解码**

```powershell
[System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String("UnicodeBase64编码放到此处"))
```

**编码**

```powershell
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes("任意字符串放此处"))
```

**例图：**

![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/738368265.png)

### 0x05 计划任务相关

1、 查看计划任务列表

`schtasks /query /fo LIST`

2、查看计划任务详情


`schtasks /query /v /tn "\Microsoft\windows\Bluetooths" /fo list`

**例图：**

![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/2586815607.png)

### 0x06 判断是否有打永恒之蓝补丁
```powershell
[array]$hotfixid=get-hotfix -id KB4012606,KB3210720,KB3210721,KB4012598,KB4012212,KB4012215,KB4012213,KB4012216,KB4012214,KB4012217,KB4013198,KB4015549 -ErrorAction 0  | ForEach-Object {$_.HotFixID}

if ($hotfixid -eq $null)
 {
 Write-host "[-] 危险: MS17010B补丁未打，易遭永恒之蓝攻击!" -ForegroundColor Red
 Write-host "补丁参考1: https://docs.microsoft.com/zh-cn/security-updates/Securitybulletins/2017/ms17-010" -ForegroundColor DarkYellow
 Write-host "补丁参考2: https://b.360.cn/other/onionwormfix" -ForegroundColor DarkYellow
 } else {
 Write-Host "[+] ms17010 补丁以打"  -ForegroundColor Green
}
```
![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/1926447350.png)
### 0x07 防火墙及ipsec相关操作

**windows防火墙允许445入站**

`netsh advfirewall set allprofiles state on netsh advfirewall firewall add rule name="allow tcp 445" dir=in protocol=tcp localport=445 action=allow` #允许445

**ipsec 禁止139、445、135端口**
```
netsh ipsec static add policy name=tomcatgo
netsh ipsec static add filterlist name=Filter1
netsh ipsec static add filter filterlist=Filter1 srcaddr=any dstaddr=Me dstport=135 protocol=TCP mirrored = yes
echo “135端口已经关闭”
netsh ipsec static add filter filterlist=Filter1 srcaddr=any dstaddr=Me dstport=139 protocol=TCP mirrored = yes
echo “139端口已经关闭”
netsh ipsec static add filter filterlist=Filter1 srcaddr=any dstaddr=Me dstport=445 protocol=TCP mirrored = yes
echo “445端口已经关闭”

netsh ipsec static add filter filterlist=Filter1 srcaddr=any dstaddr=Me dstport=135 protocol=UDP mirrored = yes
netsh ipsec static add filter filterlist=Filter1 srcaddr=any dstaddr=Me dstport=139 protocol=UDP mirrored = yes
netsh ipsec static add filter filterlist=Filter1 srcaddr=any dstaddr=Me dstport=445 protocol=UDP mirrored = yes

netsh ipsec static add filteraction name=Filteraction1 action=block
netsh ipsec static add rule name=Rule1 policy=tomcatgo filterlist=Filter1 filteraction=FilteraAtion1
netsh ipsec static set policy name=tomcatgo assign=y
```


### 0x08 获取进程md5

```powershell
get-process | where path -ne $null | %{Get-FileHash $_.path -Algorithm md5}
```
![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/3289779667.png)
### 0x09 恶意服务检测

```powershell
Get-WmiObject win32_service |?{ $_.name -eq 'svchost.exe' -and $_.PathName -notlike  '*C:\WINDOWS\System32\svchost.exe*' -and $_.PathName -not
like '*c:\Windows\SysWOW64\svchost.exe*'} | select Name, DisplayName, State, PathName
```
```powershell
Get-WmiObject win32_service | ?{$_.PathName -like '*svchost.exe*'} | select Name, DisplayName, @{Name="Path"; Expression={$_.PathName.split('
')[0]}} | Format-List
```
![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/494053681.png)

![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/550706275.png)


```powershell
Get-WmiObject win32_service |select name, @{N='FileHash';E={(Get-FileHash $_.pathname -Algorithm md5 -ErrorAction 0).hash}},pa
thname| %{if($_.filehash -eq '51D3A1E2285E2E931A553281BBA10E81'){Write-Host 恶意服务:  $_.name 执行路径: $_.pathname -ForegroundColor Red}}
```
**检测多个hash值**

```powershell
function Get-md5 {
  Param($path)
  if(Test-Path $path -PathType Leaf ){
  $md5file=certutil -hashfile "$path" MD5
  [string]$hash=$md5file -match "^[a-f0-9]{32}$"
  $retVal = New-Object -TypeName psobject -Property @{
                    Path = $path
                    Hash = $hash.ToUpper()
                }
                $retVal

  }else{  
  }
}

[array]$services_md5=Get-WmiObject win32_service  |?{ $_.PathName -notlike  '*C:\WINDOWS\System32\svchost*' -and $_.PathName -notlike '*c:\Windows\SysWOW64\svchost*'}|select  name, @{N='FileHash';E={(Get-md5 $_.pathname.replace('"','') ).hash}},pathname 
#$services_md5 |Where {$_.FileHash -eq "6983F7001DE10F4D19FC2D794C3EB534" -or  $_.FileHash -eq "59B18D6146A2AA066F661599C496090D" -or  $_.FileHash -eq "CCE36235A525858EB55070847296C4C8" -or  $_.FileHash -eq "9911858E9BEC100CCF6D8134915103EC" -or  $_.FileHash -eq "74E2A43B2B7C6E258B3A3FC2516C1235" -or  $_.FileHash -eq "D4E2EBCF92CF1B2E759FF7CE1F5688CA" -or  $_.FileHash -eq "FB89D40E24F5FF55228C38B2B07B2E77" -or  $_.FileHash -eq "637BF46077AD083659D3B96A010F38FE"}
$services_md5 |Where {$_.FileHash -eq '00A8C2DD875BC4B458CBFED72AAF45F4' -or $_.FileHash -eq '498A07B121D7A3815563DC15AC306EBD' -or $_.FileHash -eq 'B4DAABEBBF16A7C8871209D946E917F3'}
```

![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/972269086.png)

### 0x10 常用wmi命令

**1、查看服务详情**

`wmic service get name,pathname,processid,startname,status,state /value`

**例图：**

![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/3420416264.png)

**2、查看进程详情**

`wmic process get CreationDate,name,processid,commandline,ExecutablePath /value`

图例：

![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/2650766313.png)

**3、查看补丁**

`wmic qfe get hotfixid`

**4、查看启动项**

`wmic startup`

图例：

![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/2239360247.png)
**5、安装软件列表**

`wmic /NAMESPACE:"\\root\CIMV2" PATH Win32_Product get name /FORMAT:table`

**6、获取快捷方式列表**

`wmic PATH Win32_ShortcutFile get name`

**7、获取dns缓存记录**
>可通过dns 缓存记录中查看是否有恶意请求
`ipconfig /displaydns`

## 其它
>更详细快捷的查杀建议使用**pchunter** **火绒剑** **auturuns** 等安全辅助工具

