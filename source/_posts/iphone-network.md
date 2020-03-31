---
title: 全局抓取iphone 手机数据包
date: 2016-10-19 10:27
tags: app安全 
---
传统方法用 **burpsuite** 只能抓(**wifi**) **http**数据包
iOS 5后，Apple引入了**RVI remote virtual interface**的特性，它只需要将iOS设备使用USB数据线连接到mac上，然后使用rvictl工具以iOS设备的**UDID**为参数在Mac中建立一个虚拟网络接口rvi，就可以在mac设备上使用tcpdump，**wireshark**等工具对创建的接口进行抓包分析了。

`rvictl + wireshark(tcpdump)`能抓取该网卡下所有通讯

**rvictl需xcode支持**

1. 手机通过usb线连接到PC

2. 通过iTunes查看手机的UDID

3. wireshark 指定 rvi0 接口