---
title: fiddler script功能实战
date: 2019-05-25 03:37:19
tags: app测试
---

>某视频app发现有观影次数限制很是不爽；想通过分析数据包来看看能不能绕过限制

使用fiddler包分析发现返回请求数据包都加密了
![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/2019-05-25-03-23-26.png)


## 0x01 加密方式
>对每个字符进行异或混淆

```java
 public static String decodeResponse(String str) {
        char[] toCharArray = str.toCharArray();
        for (int i = 0; i < toCharArray.length; i++) {
            toCharArray[i] = (char) (toCharArray[i] ^ 20190101);
        }
        return String.valueOf(toCharArray);
    }

```

## 0x02 使用Chrome console调试解密方式

![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/2019-05-25-03-09-14.png)

```javascript
var testStr="";

var toCharArray=testStr.split('')
en='';
for (i = 0; i < toCharArray.length; i++){
en += String.fromCharCode(toCharArray[i].charCodeAt(0).toString(10) ^ 20190101)
}
```

## 0x03 使用fiddler script进行自动解密
>fiddler 使用的是`jscript.net` 语言作为脚本，使用fiddler脚本实现了自动解加密包以便后面分析

**解密后的效果**
![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/2019-05-25-03-17-29.png)

```javascript
static function OnBeforeResponse(oSession: Session) {
    if (oSession.HostnameIs("api88.awk2.work") ||oSession.HostnameIs("api99.chinanb.work") ) {
        oSession["ui-color"] = "white";
        oSession["ui-backcolor"] = "green";
        oSession.utilDecodeResponse();
        var oBody = System.Text.Encoding.UTF8.GetString(oSession.responseBodyBytes);

        var en='';
        for (var i:int = 0;i<=oBody.length;i++){
            en+=String.fromCharCode(oBody.charCodeAt(i) ^ 20190101);

        }
        oSession.utilSetResponseBody(en);  
        
    }
}

```

## 0x04 解除观看限制

>以下返回数据包记录以及控制观影次数
![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/2019-05-25-03-32-14.png)
```json
{"code":0,"data":[{"readLevel":0,"preferenceCustom":"","gender":-1,"supUserId":5483824,"todayDownNum":0,"companion":"","userBrowCnt":39,"tagIds":"","icon":"","title":"","dailyViewNum":10,"myInviteCode":"XUPBUL","userCode":null,"tagNames":"","inviteCnt":0,"nextLevelNeed":1,"leftViewNum":9,"vipExpiredDate":"","aliasName":null,"userCls":2,"level":0,"limitDownNum":0,"exceedPercent":0,"birth":"","gmtCreate":"2019-01-01 23:04:27.000","userId":9216760,"isMaxLevel":0,"oldDriver":0,"phone":"15****35","name":null,"job":""}],"enumCode":"SUCCESS","msg":"1","success":true}
```
使用fiddler脚本功能捕获关键字插入正常观影次数的数据包

![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/2019-05-25-03-37-23.png)

>手机端发现观看次数变成9次了:)

![](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images/2019-05-25-03-42-42.png)

实际上只是表面显示上是突破了，服务端有作统计校验，**本次破解以失败告终。**