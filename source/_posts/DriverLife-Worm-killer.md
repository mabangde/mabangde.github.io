---
title: 驱动人生供应链攻击挖矿专杀工具 Demo版
date: 2019-03-15 2:45:02
tags: 应急响应
---

**病毒描述：**
>360安全大脑监测到通过"驱动人生"供应链攻击传播的挖矿木马在1月30日下午4时左右再次更新。此次更新中，木马在此前抓取系统帐户密码的基础上增加了抓取密码hash值的功能，并试图通过pass the hash攻击进行横向渗透，使得该木马的传播能力进一步加强，即使是有高强度口令的机器也有可能被攻陷。
>pass the hash也称作哈希传递攻击，攻击者可以直接通过密码的哈希值访问远程主机或服务，而不用提供明文密码。攻击者使用pass the hash技术尝试在系统登录密码非弱口令并且无法抓取登录密码的情况下进行横向攻击，增加攻击成功率。
>永恒之蓝下载器木马再次更新，此前攻击者针对驱动人生公司的供应链进行攻击，利用其软件升级通道下发永恒之蓝下载器木马，并利用其软件升级通道下发木马，在攻击模块中利用“永恒之蓝”漏洞攻击，造成短时间内大范围感染。事件发生之后，驱动人生公司对受到木马影响的升级通道进行了紧急关闭。

>但该木马下载器的幕后控制者并没有就此放弃行动，而是借助其已经感染的机器进行持续攻击：包括通过云控指令下发挖矿模块，在中招机器安装多个服务以及通过添加计划任务获得持续执行的机会，后续版本在攻击模块新增SMB爆破、远程执行工具psexec攻击、利用Powershell版mimikatz获取密码，以增强其扩散传播能力。
>2019年2月10日发现的更新版本中，我们发现攻击者再次对攻击模块进行升级，改变其木马生成方式为Pyinstaller，同时在打包的Python代码中新增了email、cookie、ftp、http相关功能文件。木马通过将程序安装至系统服务、计划任务从而可以开机启动同时每隔一段时间重复启动，获得在感染电脑持续驻留的机会。

**控制维持方式：**
![2019-07-20-03-25-51](https://wolvez.oss-cn-hangzhou.aliyuncs.com/34aade5eee3e83d639ed4b2b78b03767.png)

**查杀脚本功能**

1. 删除恶意进程、服务、计划任务、注册表项、恶意文件
2. 删除无规则随机字符串服务
3. 支持最老版本样本，及最新样本
4. 由于该攻击样本不停变形增肥md5值一直在便，故采用md5黑名单+恶意证书方式来清理
5. 由于该系列病毒不断变种，本脚本不保证有效清除，仅作为技术交流使用

![2019-07-20-03-22-43](https://wolvez.oss-cn-hangzhou.aliyuncs.com/b60dec0b75dae9bec03a0ba5109990b5.png)
![2019-07-20-03-23-45](https://wolvez.oss-cn-hangzhou.aliyuncs.com/6dda05ea259444eec49c10a56849633e.png)


**恶意证书:**
驱动人生病毒所涉及的签名证书：

```text
CN="Shenzhen Smartspace Software technology Co.,Limited", OU=研发部, O="Shenzhen Smartspace Software technology Co.,Limited", L=Shenzhen, S=Guangdong, C=CN

CN="Shenzhen Qitu Software Technolgy Co., Ltd.", OU=Digital ID Class 3 - Microsoft Software Validation v2, O="Shenzhen Qitu Software Technolgy Co., Ltd.", L=Shenzhen, S=Guangdong, C=CN

CN=Shenzhen Le Zhuo Software Technologies Ltd., OU=Digital ID Class 3 - Microsoft Software Validation v2, O=Shenzhen Le Zhuo Software Technologies Ltd., L=Shenzhen, S=Guangdong, C=CN
```

根据证书判断恶意进程服务案例：
http://wolvez.club/2019/02/12/threathunter-windows/

**域名ioc**
>防火墙配置阻止以下域名访问
```text
http://r.minicen.ga/r
http://v.beahh.com/v
http://139.162.107.97/new.dat
http://p.beahh.com/upgrade.php
http://cert.beahh.com
i.haqo.net
dl.haqo.net
abbny.com

```

```powershell
Write-Host '# cowsay++' 
Write-Host '    _________________'
Write-Host '< DriverLife Worm killer v0.2>' 
Write-Host '   -----------------------'
Write-Host '    \   ,__,' -ForegroundColor Red
Write-Host '     \  (oo)____' -ForegroundColor Red
Write-Host '        (__)    )\' -ForegroundColor Red
Write-Host '           ||--|| *' -ForegroundColor Red
Write-Host 
write-host 奇安信安服应急小组出品
If (-NOT ([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator"))
 {    
  Write-Host "[x] This script needs to be run As Admin" -ForegroundColor Red
  Break
 }
 Write-Host 
$evilHashes=@()
$evilHashes = "6983F7001DE10F4D19FC2D794C3EB534",
"8B73928229C19A8F535DE8F2EB2ADC70",
"59B18D6146A2AA066F661599C496090D",
"DEF0E980D7C2A59B52D0C644A6E40763",
"23196DE0EDE25FB9659713FA6799F455",
"CE924B12FFC55021F5C1BCF308F29704",
"2FBCE2ECF670EB186C6E3E5886056312",
"30429A24F312153C0EC271CA3FEABF3D",
"F9144118127FF29D4A49A30B242CEB55",
"FB89D40E24F5FF55228C38B2B07B2E77",
"1E0DB9FDBC57525A2A5F5B4C69FAC3BB",
"5AB6F8CA1F22D88B8EF9A4E39FCA0C03",
"D4E2EBCF92CF1B2E759FF7CE1F5688CA",
"32653B2C277F18779C568A1E45CACC0F",
"AB1C947C0C707C0E0486D25D0AE58148",
"A4B7940B3D6B03269194F728610784D6",
"85013CC5D7A6DB3BCEE3F6B787BAF957",
"667A3848B411AF0B6C944D47B559150F",
"8B1AC2F487E9B0A56FBE0A07F0E1F6F4",
"CCE36235A525858EB55070847296C4C8",
"9911858E9BEC100CCF6D8134915103EC",
"74E2A43B2B7C6E258B3A3FC2516C1235",
"637BF46077AD083659D3B96A010F38FE"

$checkPath = @("$env:SystemRoot\", "$env:LOCALAPPDATA\", "$env:ALLUSERSPROFILE\")

function _calc_md5{
    Param($path)

    if($path -ne $null){
        if(Test-Path $path -PathType Leaf ){
            try{
                $fs=(new-object system.io.fileinfo($path)).openread();
                $r=(new-object system.security.cryptography.md5cryptoserviceprovider).computehash($fs);$fs.close();
                [string]$hash=[bitconverter]::tostring($r).replace('-','')   
            }catch{}

            $retVal = New-Object -TypeName psobject -Property @{
                Path = $path
                Hash = $hash
            }
            return $retVal
        }
    } else{
        write-host [-] '_calc_md5' $path -ForegroundColor Yellow
    }
}


function _del_if_match {
    Param (
        $path
    )

    $md5info = _calc_md5 $path
    if ($null -ne $md5info) {
        if ($evilHashes -eq $md5info.Hash) {
            write-host [-] 'deleting' $path -ForegroundColor Red
            Remove-Item -Path $path -Force
        }   
    }
}

function Check-And-Delete {
    Param(
        $path,
        [switch]$recurse
    )

    $params = @{
        Filter = '*.exe'
        Force = $true
        ErrorAction = 0
        Path = $path
    }

    if($recurse -ne $False) {
        $params.Add('recurse', $True)
    }

    write-host [i] 'Check-And-Delete' $path -ForegroundColor Green
    Get-ChildItem @params | %{_del_if_match $_.FullName}
}


# delete by hashes
foreach($path in $checkPath) {
    Check-And-Delete $path -recurse
}

Check-And-Delete "$env:HOMEDRIVE\" 


# delete by size
foreach ($path in @("$env:APPDATA\Microsoft\cred.ps1", "$env:ALLUSERSPROFILE\Microsoft\cred.ps1")) {
    $size = (Get-Item $path -ErrorAction SilentlyContinue).Length
    if ($size -eq 50731) {
        write-host [-] 'deleting' $path -ForegroundColor Red
        Remove-Item $path -Force
    }
}

write-host [i] 'Check-And-Delete' services -ForegroundColor Green
$servicesPath = @{}

foreach ($service in (Get-WmiObject win32_service)) {
    [String]$name = $service.pathname
    $exePath = $name.Split()[0]

    if($name.StartsWith('"')) {
        $exePath = $name.Split('"')[1]
    }

    # if powershell
    if($name.ToLower().IndexOf("powershell.exe ") -gt 0 -and $name.ToLower().IndexOf(" -enc") -gt 0) {
        write-host [-] 'deleting service' $path -ForegroundColor Red
        Stop-Service -Force -Name $service.Name -ErrorAction SilentlyContinue
        sc.exe delete $service.Name
    }

    if($name.ToLower().IndexOf("powershell") -gt 0 -and $name.ToLower().IndexOf("bypass") -gt 0) {
        write-host [-] 'deleting service' $path -ForegroundColor Red
        Stop-Service -Force -Name $service.Name -ErrorAction SilentlyContinue
        sc.exe delete $service.Name
    }

    if($servicesPath.ContainsKey($exePath)) { continue }
    $servicesPath.Add($exePath, 1)

    $servicemd5 = _calc_md5 $exePath
    if ($null -ne $servicemd5) {
        if ($evilHashes -eq $servicemd5.Hash) {
            write-host  [-] 'deleting Service:' $service.Name 'Path:' $exePath -ForegroundColor Red
            Remove-Item -Path $exePath -Force
            sc.exe delete $service.Name
        }   
    }
    ##_del_if_match $exePath
}

##删除服务

cmd.exe /c "sc.exe delete  WebServers 2>nul>nul"
cmd.exe /c "sc.exe delete Ddriver 2>nul>nul"
cmd.exe /c "sc.exe delete DnsScan 2>nul>nul"

##delete startup
write-host [i] delete startup -ForegroundColor Green
Get-ChildItem -Path 'C:\Users\*\AppData\Roaming\Microsoft\Windows\*' -force -Recurse  -ErrorAction 0 | ?{$_.name -eq "run.bat"} |Remove-Item


##删除注册表值
write-host [i] delete Registry -ForegroundColor Green
Remove-ItemProperty "HKlM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" -name "WebServers" -Force -ErrorAction 0
Remove-ItemProperty "HKlM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" -name "Ddriver" -Force -ErrorAction 0

##删除计划任务
write-host [i] delete tasks -ForegroundColor Green
cmd.exe /c "schtasks /delete /tn '\Microsoft\windows\Bluetooths' /f 2>nul"
cmd.exe /c "schtasks /delete /tn '\dnsscan' /f 2>nul>nul"
cmd.exe /c "schtasks /delete /tn '\Certificate' /f 2>nul>nul"
cmd.exe /c "schtasks /delete /tn '\dnsscan' /f 2>nul>nul"
cmd.exe /c "schtasks /delete /tn '\Ddrivers' /f 2>nul>nul"
cmd.exe /c "schtasks /delete /tn '\WebServers' /f 2>nul"

write-host [i] delete process -ForegroundColor Green
cmd.exe /c "taskkill -im wmiex.exe /f 2>nul"
wmic process where "name='svchost.exe' and ExecutablePath='C:\\windows\\system32\\drivers\\svchost.exe'" call Terminate |Out-Null
wmic process where "name='taskmgr.exe' and ExecutablePath='C:\\windows\\system32\\drivers\\taskmgr.exe'" call Terminate |Out-Null
wmic process where "name='svchost.exe' and ExecutablePath='C:\\windows\\SysWOW64\\drivers\\svchost.exe'" call Terminate |Out-Null
wmic process where "name='taskmgr.exe' and ExecutablePath='C:\\windows\\SysWOW64\\drivers\\taskmgr.exe'" call Terminate |Out-Null
wmic process where "name='updater.exe' and ExecutablePath='C:\\Windows\\temp\\updater.exe'" call Terminate |Out-Null

write-host [i] delete Forward -ForegroundColor Green
netsh interface portproxy delete v4tov4 listenport=65531 |Out-Null
netsh interface portproxy delete v4tov4 listenport=65532 |Out-Null

write-host [i] delete user:k8h3d -ForegroundColor Green
cmd.exe /c "net user k8h3d /del 2>nul" |out-null
##cmd.exe /c "taskkill -im powershell.exe /f 2>nul"

```

# 其它相关

## md5 以及签名证书信息
```powershell
function check_Signer{
 Param (
        $evil_file
    )
$Signature=Get-AuthenticodeSignature  $evil_file -ErrorAction 0
$Signature_Thumbprint= $Signature.SignerCertificate.Thumbprint
$cn=$Signature.SignerCertificate.Subject
$md5=(_calc_md5 $evil_file).hash

$retVals = New-Object -TypeName psobject -Property @{
               Signature  = $cn
                File = $evil_file
                Thumbprint=$Signature_Thumbprint
                MD5=$md5
            }
return $retVals
}
function _calc_md5{
    Param($path)

    if($path -ne $null){
        if(Test-Path $path -PathType Leaf ){
            try{
                $fs=(new-object system.io.fileinfo($path)).openread();
                $r=(new-object system.security.cryptography.md5cryptoserviceprovider).computehash($fs);$fs.close();
                [string]$hash=[bitconverter]::tostring($r).replace('-','')   
            }catch{}

            $retVal = New-Object -TypeName psobject -Property @{
                Path = $path
                Hash = $hash
            }
            return $retVal
        }
    } else{
        write-host [-] '_calc_md5' $path -ForegroundColor Yellow
    }
}
ls *.exe |% {check_Signer $_.FullName}|Select-Object MD5,Thumbprint,File,Signature|Sort-Object Th
umbprint |ft -AutoSize |Out-String -Width 99999 |Out-File infos.txt
```

```text
MD5                              Thumbprint                               File                                  Signature                                                                                                                                                                               
---                              ----------                               ----                                  ---------                                                                                                                                                                               
D80CFB955B1881F89BDC249B1844F612                                          C:\test\DriverLife\svchost1x.exe                                                                                                                                                                                              
96BDB44E072122EE05FB8E729063463C                                          C:\test\DriverLife\svchostb.exe                                                                                                                                                                                               
CCE36235A525858EB55070847296C4C8                                          C:\test\DriverLife\svchost1.exe                                                                                                                                                                                               
BC26FD7A0B7FE005E116F5FF2227EA4D                                          C:\test\DriverLife\svchos3t.exe                                                                                                                                                                                               
96BDB44E072122EE05FB8E729063463C                                          C:\test\DriverLife\svchost.exe                                                                                                                                                                                                
DF79BC5E41426BB87FE725554DD76A6E                                          C:\test\DriverLife\svvhost1.exe                                                                                                                                                                                               
9911858E9BEC100CCF6D8134915103EC                                          C:\test\DriverLife\svvhost2.exe                                                                                                                                                                                               
DF79BC5E41426BB87FE725554DD76A6E                                          C:\test\DriverLife\svvhost.exe                                                                                                                                                                                                
9911858E9BEC100CCF6D8134915103EC                                          C:\test\DriverLife\svchost_1.exe                                                                                                                                                                                              
D758AA69BC07881C136ED68A38B7BEEC                                          C:\test\DriverLife\svcxhostex.exe                                                                                                                                                                                             
0D3B854DFF7EABCC0A834F268392F3A1                                          C:\test\DriverLife\svchostx.exe                                                                                                                                                                                               
17F6582FE231831512E7CA90AC3BB97C                                          C:\test\DriverLife\log.exe                                                                                                                                                                                                    
2574D2B7CE00510323F18777EAF9F289                                          C:\test\DriverLife\lduwsx.exe                                                                                                                                                                                                 
6983F7001DE10F4D19FC2D794C3EB534                                          C:\test\DriverLife\liqegykg.exe                                                                                                                                                                                               
6983F7001DE10F4D19FC2D794C3EB534                                          C:\test\DriverLife\nidofopt.exe                                                                                                                                                                                               
6983F7001DE10F4D19FC2D794C3EB534                                          C:\test\DriverLife\jheikvql.exe                                                                                                                                                                                               
59B18D6146A2AA066F661599C496090D 02E739740B88328AC9C4A6DE0EE703B7610F977B C:\test\DriverLife\updatec.exe        CN="Shenzhen Smartspace Software technology Co.,Limited", OU=研发部, O="Shenzhen Smartspace Software technology Co.,Limited", L=Shenzhen, S=Guangdong, C=CN                                
59B18D6146A2AA066F661599C496090D 02E739740B88328AC9C4A6DE0EE703B7610F977B C:\test\DriverLife\update.exe         CN="Shenzhen Smartspace Software technology Co.,Limited", OU=研发部, O="Shenzhen Smartspace Software technology Co.,Limited", L=Shenzhen, S=Guangdong, C=CN                                
FB89D40E24F5FF55228C38B2B07B2E77 02E739740B88328AC9C4A6DE0EE703B7610F977B C:\test\DriverLife\updatedlxxxx.exe   CN="Shenzhen Smartspace Software technology Co.,Limited", OU=研发部, O="Shenzhen Smartspace Software technology Co.,Limited", L=Shenzhen, S=Guangdong, C=CN                                
FB89D40E24F5FF55228C38B2B07B2E77 02E739740B88328AC9C4A6DE0EE703B7610F977B C:\test\DriverLife\updatedl.exe       CN="Shenzhen Smartspace Software technology Co.,Limited", OU=研发部, O="Shenzhen Smartspace Software technology Co.,Limited", L=Shenzhen, S=Guangdong, C=CN                                
59B18D6146A2AA066F661599C496090D 02E739740B88328AC9C4A6DE0EE703B7610F977B C:\test\DriverLife\updll2.exe         CN="Shenzhen Smartspace Software technology Co.,Limited", OU=研发部, O="Shenzhen Smartspace Software technology Co.,Limited", L=Shenzhen, S=Guangdong, C=CN                                
59B18D6146A2AA066F661599C496090D 02E739740B88328AC9C4A6DE0EE703B7610F977B C:\test\DriverLife\updlld.exe         CN="Shenzhen Smartspace Software technology Co.,Limited", OU=研发部, O="Shenzhen Smartspace Software technology Co.,Limited", L=Shenzhen, S=Guangdong, C=CN                                
59B18D6146A2AA066F661599C496090D 02E739740B88328AC9C4A6DE0EE703B7610F977B C:\test\DriverLife\updll.exe          CN="Shenzhen Smartspace Software technology Co.,Limited", OU=研发部, O="Shenzhen Smartspace Software technology Co.,Limited", L=Shenzhen, S=Guangdong, C=CN                                
D4E2EBCF92CF1B2E759FF7CE1F5688CA 02E739740B88328AC9C4A6DE0EE703B7610F977B C:\test\DriverLife\taskmgr.exe        CN="Shenzhen Smartspace Software technology Co.,Limited", OU=研发部, O="Shenzhen Smartspace Software technology Co.,Limited", L=Shenzhen, S=Guangdong, C=CN                                
637BF46077AD083659D3B96A010F38FE 02E739740B88328AC9C4A6DE0EE703B7610F977B C:\test\DriverLife\wmiex.exe          CN="Shenzhen Smartspace Software technology Co.,Limited", OU=研发部, O="Shenzhen Smartspace Software technology Co.,Limited", L=Shenzhen, S=Guangdong, C=CN                                
59B18D6146A2AA066F661599C496090D 02E739740B88328AC9C4A6DE0EE703B7610F977B C:\test\DriverLife\svchost2.exe       CN="Shenzhen Smartspace Software technology Co.,Limited", OU=研发部, O="Shenzhen Smartspace Software technology Co.,Limited", L=Shenzhen, S=Guangdong, C=CN                                
59B18D6146A2AA066F661599C496090D 02E739740B88328AC9C4A6DE0EE703B7610F977B C:\test\DriverLife\setup-install.exe  CN="Shenzhen Smartspace Software technology Co.,Limited", OU=研发部, O="Shenzhen Smartspace Software technology Co.,Limited", L=Shenzhen, S=Guangdong, C=CN                                
59B18D6146A2AA066F661599C496090D 02E739740B88328AC9C4A6DE0EE703B7610F977B C:\test\DriverLife\installed.exe      CN="Shenzhen Smartspace Software technology Co.,Limited", OU=研发部, O="Shenzhen Smartspace Software technology Co.,Limited", L=Shenzhen, S=Guangdong, C=CN                                
59B18D6146A2AA066F661599C496090D 02E739740B88328AC9C4A6DE0EE703B7610F977B C:\test\DriverLife\setup-installa.exe CN="Shenzhen Smartspace Software technology Co.,Limited", OU=研发部, O="Shenzhen Smartspace Software technology Co.,Limited", L=Shenzhen, S=Guangdong, C=CN                                
74E2A43B2B7C6E258B3A3FC2516C1235 02E739740B88328AC9C4A6DE0EE703B7610F977B C:\test\DriverLife\svhhost2.exe       CN="Shenzhen Smartspace Software technology Co.,Limited", OU=研发部, O="Shenzhen Smartspace Software technology Co.,Limited", L=Shenzhen, S=Guangdong, C=CN                                
66EA09330BEE7239FCB11A911F8E8EA3 26E740973E7C575A9B8D0B03521F7D96CE9F3E4D C:\test\DriverLife\mn.exe             CN="Shenzhen Qitu Software Technolgy Co., Ltd.", OU=Digital ID Class 3 - Microsoft Software Validation v2, O="Shenzhen Qitu Software Technolgy Co., Ltd.", L=Shenzhen, S=Guangdong, C=CN
470B4F5BC84DB74AB1935186A3B5219F 78A149F9A04653B01DF09743571DF938F9873FA5 C:\test\DriverLife\ii.exe             CN="Shenzhen Smartspace Software technology Co.,Limited", OU=研发部, O="Shenzhen Smartspace Software technology Co.,Limited", L=Shenzhen, S=Guangdong, C=CN                                
D81233988EC80F56EA4094BAD7AB5814 FBA6E6001B3AB7C29C70A943AA0F45D911C487EA C:\test\DriverLife\yjgqsinx.exe       CN=Shenzhen Le Zhuo Software Technologies Ltd., OU=Digital ID Class 3 - Microsoft Software Validation v2, O=Shenzhen Le Zhuo Software Technologies Ltd., L=Shenzhen, S=Guangdong, C=CN  
D81233988EC80F56EA4094BAD7AB5814 FBA6E6001B3AB7C29C70A943AA0F45D911C487EA C:\test\DriverLife\yjgqsin.exe        CN=Shenzhen Le Zhuo Software Technologies Ltd., OU=Digital ID Class 3 - Microsoft Software Validation v2, O=Shenzhen Le Zhuo Software Technologies Ltd., L=Shenzhen, S=Guangdong, C=CN  
```
## 多次应急收集到的所有样本
[DriverLife.zip](https://wolvez.oss-cn-hangzhou.aliyuncs.com/files/DriverLife.zip)
>感兴趣的同学可以研究下

