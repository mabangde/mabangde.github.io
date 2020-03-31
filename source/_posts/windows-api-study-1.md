---
title: 从零开始学windows API
date: 2020-01-04 02:07:39
tags: c++学习
---

## 0x01 前置基础知识

### 宽窄字符

刚开始学编程的时候我们就接触了 `char`、`char*` 之类的，属于窄字节的，按照刚刚讲的，又多了一个 `wchar_t` 类型的字符，字符串指针的话就可以是 `wchar_t*` 类型。但是，在我们平时的编程过程中还会见到很多其他的复杂类型，如下：
-  窄字节：
char、char * 、const char *
CHAR、(PCHAR、PSTR、LPSTR)、LPCSTR

-  **Unicode** 宽字节：
wchar_t、wchar_t * 、const wchar_t *
WCHAR、(PWCHAR、PWSTR、LPWSTR)、LPCWSTR

-  **T** 通用类型：
TCHAR、(TCHAR * 、PTCHAR、PTSTR、LPTSTR)、LPCTSTR

其中：P代表指针的意思，STR代表字符串的意思，L是长指针的意思，在WIN32平台下可以忽略，**C**代表`const`常量的意思，W代表wide宽字节的意思，T大家可以理解为通用类型的意思，
就是可以根据工程中是否定义`_UNICODE` 宏，来判断当前工程的编码类型是宽字节还是窄字节，之后分别定义成不同的类型，比如：`TCHAR` 类型，如果工程中定义了`_UNICODE` 宏，那么就表明工程是宽字节编码，他最终就被定义成 `wchar_t` 类型，如果工程中没有定义_UNICODE 宏，就表明工程当前是窄字节编码，那么 `TCHAR` 被最终定义成 `char` 类型。

其方便性就是修改了工程的编码格式之后不用修改代码，所以还是建议大家在编写程序的时候使用通用类型！
[《实用VC编程之玩转字符串》第1课：宽窄字节的区别及重要性](https://www.cctry.com/thread-297534-1-1.html)

### 形参和实参
[C语言形参和实参的区别（非常详细）](http://c.biancheng.net/view/1853.html)

C语言函数的参数会出现在两个地方，分别是函数定义处和函数调用处，这两个地方的参数是有区别的。

**形参（形式参数）**

在函数定义中出现的参数可以看做是一个占位符，它没有数据，只能等到函数被调用时接收传递进来的数据，所以称为形式参数，简称形参。

**实参（实际参数）**

函数被调用时给出的参数包含了实实在在的数据，会被函数内部的代码使用，所以称为实际参数，简称实参。

形参和实参的功能是传递数据，发生函数调用时，实参的值会传递给形参。
形参和实参的区别和联系
1. 形参变量只有在函数被调用时才会分配内存，调用结束后，立刻释放内存，所以形参变量只有在函数内部有效，不能在函数外部使用。

2. 实参可以是常量、变量、表达式、函数等，无论实参是何种类型的数据，在进行函数调用时，它们都必须有确定的值，以便把这些值传送给形参，所以应该提前用赋值、输入等办法使实参获得确定值。

3. 实参和形参在数量上、类型上、顺序上必须严格一致，否则会发生“类型不匹配”的错误。当然，如果能够进行自动类型转换，或者进行了强制类型转换，那么实参类型也可以不同于形参类型。

4. 函数调用中发生的数据传递是单向的，只能把实参的值传递给形参，而不能把形参的值反向地传递给实参；换句话说，一旦完成数据的传递，实参和形参就再也没有瓜葛了，所以，在函数调用过程中，形参的值发生改变并不会影响实参。


### 指针和引用
函数参数传递中值传递、地址传递、引用传递的区别:

1.  值传递,会为形参重新分配内存空间,将实参的值拷贝给形参,形参的改变**不会影响实参的值**,函数调用结束,形参被释放.
2.  引用传递,**不会为形参重新分配内存**空间,形参只是实参的别名,形参的改变**会影响实参的值**,函数调用结束后,形参不会被释放.
3.  地址传递,形参为指针变量,将实参的地址传递给函数,可以在函数中**改变实参的值**.调用时为形参指针变量分配内存,结束时释放指针变量

#### 测试代码:

```cpp
#include <stdio.h>
void main()
{
    int i=888;
    int * p=&i;
    int & r=i;

    printf("i=%d &i=%d\n",i,&i);
    printf("p=%d &p=%d\n",p,&p);
    printf("r=%d &r=%d\n",r,&r);
}
```

运行结果:

```text
i=888 &i=8388272
p=8388272 &p=8388260
r=888 &r=8388272
```
![2020-01-01-20-33-30](https://wolvez.oss-cn-hangzhou.aliyuncs.com/5bc3e4be2b68b375c5faf6e6e585f4a7.png)



### 句柄
在程序设计中，句柄（handle）是Windows操作系统用来标识被应用程序所创建或使用的对象的整数。其本质相当于带有引用计数的智能指针。当一个应用程序要引用其他系统（如数据库、操作系统）所管理的内存块或对象时，可以使用句柄。

**句柄与指针**

句柄与普通指针的区别在于，指针包含的是引用对象的内存地址，而句柄则是由系统所管理的引用标识，该标识可以被系统重新定位到一个内存地址上。这种间接访问对象的模式增强了系统对引用对象的控制。（参见封装）。通俗的说就是我们调用句柄就是调用句柄所提供的服务，即句柄已经把它能做的操作都设定好了，我们只能在句柄所提供的操作范围内进行操作，但是普通指针的操作却多种多样，不受限制。

**句柄与安全性**
客户获得句柄时，句柄不仅是资源的标识符，也被授予了对资源的特定访问权限。

[句柄说明](https://zh.wikipedia.org/wiki/%E5%8F%A5%E6%9F%84)

以下所有案例均以`UNICODE`为例


### static关键字

`static` 关键字至少有下列N个作用:
1. 函数体内`static`变量的作用范围为该函数体,不同于auto变量,该变量的内存**只被分配一次**,因此其值在下次调用时任维持上次的值;
2. 在模块内的`static`全局变量可以被模块内所有函数访问,但不能被模块外其它函数访问;
3. 在模块内的`static`函数只可被这一模块内的其它函数调用,这个函数的使用范围被限制在声明它的模块内;
4. 在类中的`static`成员变量属于整个类所拥有,对所有对象只有一份拷贝;
5. 在类中的`static`成员函数属于整个类所拥有,这个函数不接受`this`指针,因而只能访问类的`staic`成员.

### 文件操作相关

>`fscanf()` 和 `fprintf()` 函数与前面使用的 `scanf()` 和 `printf()` 功能相似，都是格式化读写函数，两者的区别在于 `fscanf()` 和 `fprintf()` 的读写对象不是键盘和显示器，而是磁盘文件。


在C语言中，用一个指针变量指向一个文件，这个指针称为文件指针。通过文件指针就可对它所指的文件进行各种操作。


定义文件指针的一般形式为：
`FILE  *fp;`
这里的`FILE`，实际上是在`stdio.h`中定义的一个结构体，该结构体中含有文件名、文件状态和文件当前位置等信息，`fopen` 返回的就是`FILE`类型的指针。

注意：`FILE`是文件缓冲区的结构，`fp`也是指向文件缓冲区的指针。

## 0x02 windows 常用api学习

### ReadFile

函数说明:

```cpp
BOOL WINAPI ReadFile(
  _In_         HANDLE hFile,                // 读取的文件句柄
  _Out_        LPVOID lpBuffer,             // 保存读取缓冲字符数组
  _In_         DWORD nNumberOfBytesToRead,  // 缓冲数组的大小
  _Out_opt_    LPDWORD lpNumberOfBytesRead, // 实际读出的大小
  _Inout_opt_  LPOVERLAPPED lpOverlapped    // 异步IO文件结构体
);
```

用法:

```cpp
eadFile(
            hConsoleInput,  // 文件句柄
            Command_str,    // 获取内容的缓冲字符数组
            MAX_PATH,       // 缓冲数组大小
            &Command_len,   // 实际读出的大小
            NULL);
```

### CreateProcess

函数说明:
```cpp
BOOL WINAPI CreateProcess(
  _In_opt_     LPCTSTR lpApplicationName,   // 启动程序路径
  _Inout_opt_  LPTSTR lpCommandLine,        // 启动程序的命令行代码
  _In_opt_     LPSECURITY_ATTRIBUTES lpProcessAttributes,   // 进程属性
  _In_opt_     LPSECURITY_ATTRIBUTES lpThreadAttributes,    // 线程属性
  _In_         BOOL bInheritHandles,        // 是否继承句柄
  _In_         DWORD dwCreationFlags,       // 标识和优先级
  _In_opt_     LPVOID lpEnvironment,        // 环境变量设置
  _In_opt_     LPCTSTR lpCurrentDirectory,  // 当前目录设置
  _In_         LPSTARTUPINFO lpStartupInfo, // 启动信息设置
  _Out_        LPPROCESS_INFORMATION lpProcessInformation   // 新进程的信息
);
```

用法:

```cpp
CreateProcess(
        NULL,           // 不传程序路径, 使用命令行
        cmd_str,        // 命令行命令
        NULL,           // 不继承进程句柄(默认)
        NULL,           // 不继承线程句柄(默认)
        FALSE,          // 不继承句柄(默认)
        0,              // 没有创建标志(默认)
        NULL,           // 使用默认环境变量
        NULL,           // 使用父进程的目录
        &start_info,    // STARTUPINFO 结构
        &process_info );// PROCESS_INFORMATION 保存相关信息
```

### LogonUser

>`LogonUser` 方法只用于 本地计算机用户尝试本地登陆，不能用于登陆远程计算机。
登陆成功后，会返回这个用户的token，拥有这个token值，可以用来模拟这个用户的所有操作，也可以用来创建可以运行在登陆用户上下文的进程。

#### 语法

```cpp
BOOL LogonUserA(
  LPCSTR  lpszUsername,
  LPCSTR  lpszDomain,
  LPCSTR  lpszPassword,
  DWORD   dwLogonType,
  DWORD   dwLogonProvider,
  PHANDLE phToken
);
```

#### 参数说明
`lpszUsername` 

登录用户名,如果使用**用户主体名称**（UPN）格式`User @ DNSDomainName`，则`lpszDomain`参数必须为**NULL**。

`lpszDomain`

指向以空字符结尾的字符串的指针，该字符串指定其帐户数据库包含`lpszUsername`帐户的域或服务器的名称。如果此参数为**NULL*，则必须以UPN格式指定用户名。如果此参数为"."，则该函数仅使用本地帐户数据库来验证帐户。

`lpszPassword`

指向以空字符结尾的字符串的指针，用户密码

`dwLogonType`

要执行的登录操作类型

Value |	Meaning
--|--
LOGON32_LOGON_BATCH|此登录类型适用于批处理服务器，在批处理服务器中，进程可以代表用户执行而无需其直接干预. 此类型也适用于同时处理许多纯文本身份验证尝试的高性能服务器，例如邮件或Web服务器.
LOGON32_LOGON_INTERACTIVE|此登录类型适用于将使用计算机进行交互的用户，例如通过终端服务器，远程外壳程序或类似进程登录的用户. 这种登录类型的额外开销是为断开连接的操作缓存登录信息. 因此，它不适用于某些客户端/服务器应用程序，例如邮件服务器.
LOGON32_LOGON_NETWORK|此登录类型旨在用于高性能服务器来验证纯文本密码. LogonUser函数不缓存此登录类型的凭据.
LOGON32_LOGON_NETWORK_CLEARTEXT|此登录类型将名称和密码保留在身份验证包中 ，该名称和密码允许服务器在模拟客户端的同时建立与其他网络服务器的连接. 服务器可以接受来自客户端的纯文本凭据，调用LogonUser ，验证用户可以通过网络访问系统，并且仍然与其他服务器通信.
LOGON32_LOGON_NEW_CREDENTIALS|此登录类型允许调用方克隆其当前令牌，并为出站连接指定新的凭据. 新的登录会话具有相同的本地标识符，但对其他网络连接使用不同的凭据.仅LOGON32_PROVIDER_WINNT50登录提供程序支持此登录类型.
LOGON32_LOGON_SERVIC|表示服务类型的登录. 提供的帐户必须启用了服务特权.
LOGON32_LOGON_UNLOCK|不再支持GINA.Windows Server 2003和Windows XP：此登录类型用于GINA DLL，这些DLL登录将交互使用计算机的用户. 此登录类型可以生成唯一的审核记录，该记录显示工作站何时被解锁.

`dwLogonProvider`

指定登录提供程序. 此参数可以是下列值之一.

Value	| Meaning
--|--
LOGON32_PROVIDER_DEFAULT |使用系统的标准登录提供程序. 除非您为域名传递NULL且用户名不是UPN格式，否则默认的安全提供程序为协商. 在这种情况下，默认提供程序为NTLM.
LOGON32_PROVIDER_WINNT50 | 使用协商登录提供程序.
LOGON32_PROVIDER_WINNT40 | 使用NTLM登录提供程序.


`phToken`

指向句柄变量的指针，该变量接收代表指定用户的令牌的句柄.

您可以在对`ImpersonateLoggedOnUser`函数的调用中使用返回的句柄.

在大多数情况下，返回的句柄是主要令牌 ，您可以在对`CreateProcessAsUser`函数的调用中使用它. 但是，如果指定`LOGON32_LOGON_NETWORK`标志，则除非调用`DuplicateTokenEx`将其转换为主要令牌，否则`LogonUser`将返回模拟令牌 ，该令牌无法在`CreateProcessAsUser`中使用.

当您不再需要此句柄时，可以通过调用`CloseHandle`函数将其关闭 .

`Return Value`

如果函数成功，函数将返回非零值.

如果函数失败，则返回零. 要获取扩展的错误信息，请调用`GetLastError` .

**代码实例**

您可以使用以下代码生成LocalService令牌:

`LogonUser(L"LocalServer", L"NT AUTHORITY", NULL, LOGON32_LOGON_SERVICE, LOGON32_PROVIDER_DEFAULT, &hToken)`


### OpenProcessToken
获得进程访问令牌的句柄



语法:

```c
BOOL OpenProcessToken(
  HANDLE  ProcessHandle,
  DWORD   DesiredAccess,
  PHANDLE TokenHandle
);
```
第一参数是要修改访问权限的进程句柄；第三个参数就是返回的访问令牌指针；第二个参数指定你要进行的操作类型，如要修改令牌我们要指定第二个参数为TOKEN_ADJUST_PRIVILEGES

参数:
`ProcessHandle`
打开访问令牌的进程的句柄。该进程必须具有PROCESS_QUERY_INFORMATION访问权限。

指定[访问掩码](https://docs.microsoft.com/windows/desktop/SecGloss/a-gly),该[掩码](https://docs.microsoft.com/windows/desktop/SecGloss/a-gly)指定请求的访问令牌访问类型。将这些请求的访问类型与令牌的[自由访问控制列表](https://docs.microsoft.com/windows/desktop/SecGloss/d-gly)（DACL）进行比较，以确定允许或拒绝哪些访问。

有关访问令牌的访问权限的列表，请参阅 [访问令牌对象的访问权限](https://docs.microsoft.com/windows/desktop/SecAuthZ/access-rights-for-access-token-objects)。

`TokenHandle`

函数返回时指向标识新打开的访问令牌的句柄的指针。

返回值:

如果函数成功，则返回值为非零。
如果函数失败，则返回值为零。要获取扩展的错误信息，请调用 GetLastError。

备注:

通过调用CloseHandle关闭通过TokenHandle参数 返回的访问令牌句柄。


### ImpersonateLoggedOnUser

可以让当前线程模拟登陆用户进行操作
```
BOOL ImpersonateLoggedOnUser(
  HANDLE hToken
);
```
`hToken`

代表已登录用户的主要或模拟访问令牌的句柄。这可以是通过调用`LogonUser`， `CreateRestrictedToken`， `DuplicateToken`， `DuplicateTokenEx`， `OpenProcessToken`或 `OpenThreadToken`函数返回的令牌句柄 。如果`hToken`是主令牌的句柄，则该令牌必须具有`TOKEN_QUERY`和`TOKEN_DUPLICATE`访问权限。如果`hToken`是模拟令牌的句柄，则令牌必须具有`TOKEN_QUERY`和`TOKEN_IMPERSONATE` 访问。


## 以下内容后续更新

### malloc
### CreateProcessAsUser
### CreateProcessWithLogonW
### GetUserProfileDirectory
### DestroyEnvironmentBlock

- [获取进程信息](http://exp-blog.com/gitbook/book/markdown/notes/language/c/GetProcInfo.html)
- [木马，你好！](https://lellansin.wordpress.com/tutorials/hello_trojan/)



