---
title:  "获取http信息小工具"
date:   2018-09-19 13:05:14 +0100
author: wolvez
tags: http
---

功能: 获取网站titele信息 服务器信息
批量的话结合 for 命令使用
<!--more-->

![hinfo](https://wolvez.oss-cn-hangzhou.aliyuncs.com/images%2F7788811111.PNG)

---

```cpp
#define CURL_STATICLIB  //必须在包含curl.h前定义
#include <io.h> 
#include<iostream>
#include"curl/curl.h"
#include<ctime>  
#include<string>
#include <cctype>
#include <regex>
#include "libiconv\iconv.h"
#include <algorithm> 
//std::string my_str="sssss";

//my_str.erase(std::remove(my_str.begin(), my_str.end(), '\t'), my_str.end());

using namespace std;  
using namespace std::regex_constants;
//以下四项是必须的
#pragma comment ( lib, "libcurl.lib" )
#pragma comment (lib, "libiconv.lib")
#pragma comment ( lib, "ws2_32.lib" )
#pragma comment ( lib, "winmm.lib" )
#pragma comment ( lib, "wldap32.lib" )

static int writer(char *, size_t, size_t, string *);
//string HostToIp(const std::string& host);
static bool init(CURL *&, char *,string *,string *);
using std::string;

/*!
https://github.com/svenyao/libiconv/blob/master/docs/libiconv.example.test.txt
	对字符串进行语言编码转换
	param from  原始编码，比如"GB2312",的按照iconv支持的写
	param to    转换的目的编码
	param save  转换后的数据保存到这个指针里，需要在外部分配内存
	param savelen  存储转换后数据的内存大小
	param src      原始需要转换的字符串
	param srclen   原始字符串长度
*/

int convert(const char *from, const char *to, char* save, int savelen, const char *src, int srclen)
{
    iconv_t cd;
    const char   *inbuf = src;
    char *outbuf = save;
    size_t outbufsize = savelen;
    int status = 0;
    size_t  savesize = 0;
    size_t inbufsize = srclen;
    const char* inptr = inbuf;
    size_t      insize = inbufsize;
    char* outptr = outbuf;
    size_t outsize = outbufsize;
    
    cd = iconv_open(to, from);
    iconv(cd, NULL, NULL, NULL, NULL);
    if (inbufsize == 0) {
        status = -1;
        goto done;
    }
    while (insize > 0) {
        size_t res = iconv(cd, (const char**)&inptr, &insize, &outptr, &outsize);
        if (outptr != outbuf) {
            int saved_errno = errno;
            int outsize = outptr - outbuf;
            strncpy(save+savesize, outbuf, outsize);
            errno = saved_errno;
        }
        if (res == (size_t)(-1)) {
            if (errno == EILSEQ) {
                int one = 1;
                iconvctl(cd, ICONV_SET_DISCARD_ILSEQ,&one);
                status = -3;
            } else if (errno == EINVAL) {
                if (inbufsize == 0) {
                    status = -4;
                    goto done;
                } else {
                    break;
                }
            } else if (errno == E2BIG) {
                status = -5;
                goto done;
            } else {
                status = -6;
                goto done;
            }
        }
    }
    status = strlen(save);
done:
    iconv_close(cd);
    return status;
}


/*
 *  convert(...) string封装
 *  [in|out]put string : src_str
 *  s[from|to]: 编码名称
 */

int convert_str(const std::string& sfrom, const std::string& sto, std::string& src_str)
{
	int ilen = src_str.length();
	int ioutlen = ilen * 3;
	char *outbuf = (char *)malloc(ioutlen);
	memset(outbuf, 0, ioutlen);

	int nret = convert(sfrom.c_str(), sto.c_str(), outbuf, ioutlen, src_str.c_str(), ilen);
	if (nret < 0)
	{
		free(outbuf);
		outbuf = NULL;
		return nret;
	}

	outbuf[nret] = '\0';
	src_str = outbuf;
	free(outbuf);
	outbuf = NULL;

	return nret;
}

void Usage(char* prog)
{
    printf("[*]:%s Usage-> http://www.360.net \r\n",prog);
    printf("[*]:code by: lostwolf \r\n");
}

int main(int argc,char* argv[])
{
    CURL *conn = NULL;
    CURLcode code;
	string buffer;
	string h_buffer;
	string XPowered;
	string content;
	string charset;
	char  *ip;
	long response_code;
	string title;
	string header;
	string  server;
   if (argc != 2)
    {
        Usage(argv[0]);
        return 0;
    }
    //lstrcpyW(tp.Host,argv[1]);
    curl_global_init(CURL_GLOBAL_DEFAULT);

    char* url=argv[1];

   init(conn,url,&buffer,&h_buffer);
    code = curl_easy_perform(conn);
	
	content=buffer.c_str();
	header=h_buffer.c_str();
   regex re_title("<title>([^<]+)</title>");
   regex re_charset("content=\".*charset=(.*?)\""); 
   regex re_server("Server:\\s(.*?)\\r");
   regex re_XPowered("X-Powered-By:\\s(.*?)\\r");
   smatch match_XPowered;
   smatch match_charset;
   smatch match_title;
   smatch match_server;

   	if ( regex_search(content, match_charset, re_charset) ) {
		charset=match_charset[1].str();
	}else{
	charset="utf-8";
	}
	if ( regex_search(content, match_title, re_title) ) {
		title=match_title[1].str();
		convert_str(charset, "gb2312", title);
   }
	if ( regex_search(header, match_server, re_server) ) {
		server=match_server[1].str();
   }
	if ( regex_search(header, match_XPowered, re_XPowered) ) {
		XPowered=match_server[1].str();
   }
	if(server==XPowered){
	XPowered="";
	}

	if((code == CURLE_OK) &&
     !curl_easy_getinfo(conn, CURLINFO_PRIMARY_IP, &ip) && ip) {
		 curl_easy_getinfo(conn, CURLINFO_RESPONSE_CODE, &response_code);
		 cout <<"Target->["<<url<<']'<<' '; 
		 cout <<"Code->["<<response_code<<']'<<' '; 
	cout <<"IP->["<<ip<<']'<<' '; 
	cout <<"Server->["<<server<<']'<<' '; 
	cout <<XPowered<<' ';
	cout <<"Title->["<<title<<']'<<' ';
	cout <<'\n';
	curl_easy_cleanup(conn);
  }else{
  
  cout <<"[!] "<<url<<" connect failed!"<<' ';
  cout << '\n';
  }

	return 0;
}

static bool init(CURL *&conn, char *url,string *p_buffer,string *h_buffer)
{
    CURLcode code;
	conn = curl_easy_init();
    code = curl_easy_setopt(conn, CURLOPT_URL, url);
	//code = curl_easy_setopt(conn, CURLOPT_VERBOSE, 1L);  //详细
    code = curl_easy_setopt(conn, CURLOPT_TIMEOUT, 3);
    code = curl_easy_setopt(conn, CURLOPT_FOLLOWLOCATION, 1);
	curl_easy_setopt(conn, CURLOPT_USERAGENT, "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.90 Safari/537.36");
    code = curl_easy_setopt(conn, CURLOPT_WRITEFUNCTION, writer);
	code = curl_easy_setopt(conn, CURLOPT_HEADERDATA, h_buffer);
    code = curl_easy_setopt(conn, CURLOPT_WRITEDATA, p_buffer);


    return true;
}

static int writer(char *data, size_t size, size_t nmemb, string *writerData)
{
    unsigned long sizes = size * nmemb;
    if (writerData == NULL) return 0;
    writerData->append(data, sizes);
    return sizes;
}

```

下载:
[hinfo.exe](https://wolvez.oss-cn-hangzhou.aliyuncs.com/files%2Fhtml.zip)

