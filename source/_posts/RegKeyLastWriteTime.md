---
title: 查看注册表键值修改时间
date: 2019-10-23 15:37:07
tags: c++学习
---

```cpp
#include <stdio.h>
#include <string.h>
#include <Windows.h>
#include <iostream>
#include <tchar.h>

#define MAX_KEY_LENGTH 255
#define MAX_VALUE_NAME 16383

void __cdecl TestMain(void);

int wmain(int argc, TCHAR * argv[])
{
	TestMain();

}

void QueryKey(HKEY hKey) 
{ 
	char    achKey[MAX_KEY_LENGTH];   // buffer for subkey name
	DWORD    cbName;                   // size of name string 
	TCHAR    achClass[MAX_PATH] = TEXT("");  // buffer for class name 
	DWORD    cchClassName = MAX_PATH;  // size of class string 
	DWORD    cSubKeys=0;               // number of subkeys 
	DWORD    cbMaxSubKey;              // longest subkey size 
	DWORD    cchMaxClass;              // longest class string 
	DWORD    cValues;              // number of values for key 
	DWORD    cchMaxValue;          // longest value name 
	DWORD    cbMaxValueData;       // longest value data 
	DWORD    cbSecurityDescriptor; // size of security descriptor 
	FILETIME ftLastWriteTime;      // last write time 
 
	DWORD i, retCode; 
 
	TCHAR  achValue[MAX_VALUE_NAME]; 
	DWORD cchValue = MAX_VALUE_NAME; 
 
	// Get the class name and the value count. 
	retCode = RegQueryInfoKey(
		hKey,                    // key handle 
		achClass,                // buffer for class name 
		&cchClassName,           // size of class string 
		NULL,                    // reserved 
		&cSubKeys,               // number of subkeys 
		&cbMaxSubKey,            // longest subkey size 
		&cchMaxClass,            // longest class string 
		&cValues,                // number of values for this key 
		&cchMaxValue,            // longest value name 
		&cbMaxValueData,         // longest value data 
		&cbSecurityDescriptor,   // security descriptor 
		&ftLastWriteTime);       // last write time 
 
	// Enumerate the subkeys, until RegEnumKeyEx fails.
	
	if (cSubKeys)
	{
		printf( "\nNumber of subkeys: %d\n", cSubKeys);

		for (i=0; i<cSubKeys; i++) 
		{ 
			cbName = MAX_KEY_LENGTH;

			retCode = RegEnumKeyExA(hKey, i,
					 achKey, 
					 &cbName, 
					 NULL, 
					 NULL, 
					 NULL, 
					 &ftLastWriteTime); 
			if (retCode == ERROR_SUCCESS) 
			{
				char szLocalTime[255];
				char szLocalDate[255];
				SYSTEMTIME sSTYM;
				 FileTimeToSystemTime(&ftLastWriteTime, &sSTYM);  
				 GetTimeFormatA( LOCALE_USER_DEFAULT, 0, &sSTYM, NULL, szLocalTime, 255 );
				 GetDateFormatA( LOCALE_USER_DEFAULT, DATE_LONGDATE, &sSTYM, NULL,
     szLocalDate, 255 );
				printf("%s [%s %s]\n",achKey,szLocalDate,szLocalTime);
			}
		}
	} 
 

	if (cValues) 
	{
		printf( "\nNumber of values: %d\n", cValues);

		for (i=0, retCode=ERROR_SUCCESS; i<cValues; i++) 
		{ 
			cchValue = MAX_VALUE_NAME; 
			achValue[0] = '\0'; 
			retCode = RegEnumValue(hKey, i, 
				achValue, 
				&cchValue, 
				NULL, 
				NULL,
				NULL,
				NULL);
 
			if (retCode == ERROR_SUCCESS ) 
			{ 
				_tprintf(TEXT("(%d) %s\n"), i+1, achValue); 
			} 
		}
	}
}

void __cdecl TestMain(void)
{
   HKEY hTestKey;

   if( RegOpenKeyEx( HKEY_LOCAL_MACHINE,
		TEXT("SYSTEM\\CurrentControlSet\\services"),
		0,
		KEY_READ,
		&hTestKey) == ERROR_SUCCESS
	  )
   {
	  QueryKey(hTestKey);
   }
   
   RegCloseKey(hTestKey);
}
```