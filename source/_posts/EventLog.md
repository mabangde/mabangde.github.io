---
title: windows 日志分析
date: 2019-4-7 17:38
tags: 应急响应
---

## 前置知识

### 常用事件 ID 含义

Event ID(2000/XP/2003)|	Event ID(Vista/7/8/2008/2012)|	描述|	日志名称
---|---|---|---
528|	4624|	成功登录|	Security
529|	4625|	失败登录|	Security
680 |	4776|	成功/失败的账户认证|	Security
624 |	4720|	创建用户|	Security
636	|4732 |	添加用户到启用安全性的本地组中|	Security
632	|4728	|添加用户到启用安全性的全局组中|	Security
2934 |	7030 |	服务创建错误|	System
2944 |	7040 |	IPSEC服务服务的启动类型已从禁用更改为自动启动|	System
2949|	7045 |	服务创建|System
556|    4662|   DC hash传递攻击|Security


### 登陆类型

登录类型 |	登录类型|	描述
---|---|---
2	|Interactive|	用户登录到本机
3	|Network	|用户或计算手机从网络登录到本机，如果网络共享，或使用 net use 访问网络共享，net view 查看网络共享
4|	Batch|	批处理登录类型，无需用户干预
5|	Service	|服务控制管理器登录
7	|Unlock	|用户解锁主机
8	|NetworkCleartext|	用户从网络登录到此计算机，用户密码用非哈希的形式传递
9	|NewCredentials	|进程或线程克隆了其当前令牌，但为出站连接指定了新凭据
10|	Remotelnteractive	|使用终端服务或远程桌面连接登录
11	|Cachedlnteractive|	用户使用本地存储在计算机上的凭据登录到计算机（域控制器可能无法验证凭据），如主机不能连接域控，以前使用域账户登录过这台主机，再登录就会产生这样日志
12|	CachedRemotelnteractive	|与 Remotelnteractive 相同，内部用于审计目的
13|	CachedUnlock|	登录尝试解锁

## Log Parser 常用命令
失败记录(爆破行为)

**直接查看**

`LogParser.exe  -stats:OFF -i:EVT  "SELECT top 1000 TimeGenerated AS Date,  EXTRACT_TOKEN(strings,10,'|') as Logtype,EXTRACT_TOKEN(strings,19,'|') as SourceIP,EXTRACT_TOKEN(strings,13,'|') as ComputerName,EXTRACT_TOKEN(strings,5,'|') as User  from 'Security.evtx' where EventID=4625" -o:DATAGRID`

**导出csv**
>方便后期excel 分析

`LogParser.exe  -stats:OFF -i:EVT  "SELECT  TimeGenerated AS Date,  EXTRACT_TOKEN(strings,10,'|') as Logtype,EXTRACT_TOKEN(strings,19,'|') as SourceIP,EXTRACT_TOKEN(strings,5,'|') as User into tmp_result.csv from 'Security.evtx' where EventID=4625" -o:csv`

**成功登陆(4624)**
>`not like '%$'` 一般用于查询非机器登陆域控账号

`LogParser.exe  -stats:OFF -i:EVT  "SELECT  top 1000 TimeGenerated AS Date,EXTRACT_TOKEN(strings,8,'|') as Logtype,EXTRACT_TOKEN(strings,18,'|') as SourceIP,EXTRACT_TOKEN(strings,5,'|') as User from  'Security.evtx' where EventID=4624 And User not like '%$'" -o:DATAGRID`

**hash传递(4662)**
`LogParser.exe -i:EVT "select distinct TimeGenerated,EXTRACT_TOKEN(Strings,2,'|') AS Domain,EXTRACT_TOKEN(Strings,1,'|') AS UserName,ComputerName from Security where EventID=4662 order by TimeGenerated desc" -o:DATAGRID`




## 使用powershell 将外部日志导成csv

**evtx** 导出**csv** 主要目的方便后面**excel** 分析、筛选。前面用多种方法导出csv发现速度慢得要命，**LogParser.exe**命令又太长， 后面版本将参数化、并加入多种日志一键导出

>使用Get-WinEvent 方法导出日志特点是非常非常非常慢，对于日志量小还可以接受。优点是完全使用powershell不依赖第三方，且可以导入外部日志 Get-EventLog 就要快很多，但是不支持导入外部日志，单纯分析当前机器日志可以使用Get-EventLog来实现

```powershell
$hostname=(hostname)
$LogonType = @{
    [uint32]2 = 'Interactive'
    [uint32]3 = 'ipc 登陆'
    [uint32]4 = 'Batch'
    [uint32]5 = 'Service'
    [uint32]7 = 'Unlock'
    [uint32]8 = 'NetworkCleartext'
    [uint32]9 = 'NewCredentials'
    [uint32]10 = '远程桌面登陆'
    [uint32]11 = 'CachedInteractive'
}

$events_fail =Get-WinEvent -FilterHashTable @{ ID=4625;ProviderName='Microsoft-Windows-Security-Auditing';LogName="Security";Path="日志路径\_Security.evtx";}
$events_success =Get-WinEvent -FilterHashTable @{ ID=4624;ProviderName='Microsoft-Windows-Security-Auditing';LogName="Security";Path="日志路径\_Security.evtx";}

$events_fail|ForEach-Object { New-Object PSObject -Property ([ordered]@{
            MachineName = ($_.MachineName).Split('.')[0]
            TimeCreated = $_.TimeCreated
            User = $_.Properties[5].Value
            Domain = $_.Properties[6].Value
            LogonType = $_.Properties[10].Value
            LogonTypeString = $LogonType[$_.Properties[10].Value]
            SourceIP = $_.Properties[19].Value
            Keywords = $_.KeywordsDisplayNames -join ";"
        })}|Export-Csv -Path $hostname"_fail_login.csv" -NoTypeInformation -UseCulture  -Encoding Default


$events_success |ForEach-Object { New-Object PSObject -Property ([ordered]@{
            MachineName = ($_.MachineName).Split('.')[0]
            TimeCreated = $_.TimeCreated
            User = $_.Properties[5].Value
            Domain = $_.Properties[6].Value
            LogonType = $_.Properties[8].Value
            LogonTypeString = $LogonType[$_.Properties[8].Value]
            SourceIP = $_.Properties[18].Value
            Keywords = $_.KeywordsDisplayNames -join ";"
        })} | Export-Csv -Path $hostname"_success_login.csv" -NoTypeInformation -UseCulture -Encoding Default
```

**效果如下：**
![2019-07-19-04-53-03](https://wolvez.oss-cn-hangzhou.aliyuncs.com/c8584938b22eec1dd3f86528b1d630af.png)

## 使用WEVTUtil+powershell 将外部日志导出csv（最终版）
>此种方法优点就是快，支持导入外部evtx，对于几百兆巨大日志丝毫不费力
```powershell
Param (
    [string]$evtx = $pwd.Path+"\*_Security.evtx"
)

$time=Get-Date -Format h:mm:ss
$evtx=(Get-Item $evtx).fullname
$outfile=(Get-Item $evtx).BaseName+".csv"

$logsize=[int]((Get-Item $evtx).length/1MB)

write-host [+] $time Load $evtx "("Size: $logsize MB")" ... -ForegroundColor Green
[xml]$xmldoc=WEVTUtil qe  $evtx /q:"*[System[Provider[@Name='Microsoft-Windows-Security-Auditing']  and (EventID=4624 or EventID=4625)] and EventData[Data[@Name='LogonType']='3'] or EventData[Data[@Name='LogonType']='10']]" /e:root /f:Xml  /lf

$xmlEvent=$xmldoc.root.Event

function OneEventToDict {
    Param (
        $event
    )
    $ret = @{
        "SystemTime" = $event.System.TimeCreated.SystemTime | Convert-DateTimeFormat -OutputFormat 'yyyy"/"MM"/"dd HH:mm:ss';
        "EventID" = $event.System.EventID
    }
    $data=$event.EventData.Data
    for ($i=0; $i -lt $data.Count; $i++){
        $ret.Add($data[$i].name, $data[$i].'#text')
    }
    return $ret
}

filter Convert-DateTimeFormat
{
  Param($OutputFormat='yyyy-MM-dd HH:mm:ss fff')
  try {
    ([DateTime]$_).ToString($OutputFormat)
  } catch {}
}

$time=Get-Date -Format h:mm:ss
write-host [+] $time Extract XML ... -ForegroundColor Green
[System.Collections.ArrayList]$results = New-Object System.Collections.ArrayList($null)
for ($i=0; $i -lt $xmlEvent.Count; $i++){
    $event = $xmlEvent[$i]
    $datas = OneEventToDict $event

    $results.Add((New-Object PSObject -Property $datas))|out-null
}

$time=Get-Date -Format h:mm:ss
write-host [+] $time Dump into CSV: $outfile ... -ForegroundColor Green
$results | Select-Object SystemTime,IpAddress,TargetDomainName,TargetUserName,EventID,LogonType | Export-Csv $outfile -NoTypeInformation -UseCulture  -Encoding Default -Force
```
最终根据excel处理
**效果如下：**


![2019-07-19-04-57-06](https://wolvez.oss-cn-hangzhou.aliyuncs.com/cebf28fffd121868e2b0b9b688cdc2a9.png)
![2019-07-19-04-57-25](https://wolvez.oss-cn-hangzhou.aliyuncs.com/76632fe9dcbdffb78a9786506386a109.png)

**附录**
[windows日志分析](https://xz.aliyun.com/t/2524)
[恶意日志分析](https://jpcertcc.github.io/ToolAnalysisResultSheet/
)
[windows日志](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/appendix-l--events-to-monitor)
[logparser tips](https://gist.github.com/exp0se/1bae653b790cf5571d20)

