---
title: RedHat6 升级 GLIBC
date: 2019-01-17 21:13:09
categories: Server
tags: [Server, ]
---

>在安装 Nanopore 测序平台需要的 Guppy 等软件时，发现集群上系统是 RHEL6.3，自带 gcc 4.4.6和 glibc 2.12，已经远远落后于时代了，新软件编译时经常报错，提示需要 GLIBC 2.17，因此今天着手升级了一下 gcc 和 glibc 的版本

<!-- more -->

## 升级 glibc 2.17

查看 RedHat 发行版本

```
[root@node5 ~]# cat /etc/redhat-release
Red Hat Enterprise Linux Server release 6.3 (Santiago)
```

查看现在 GLIBC 的版本，支持最高只到 2.12

```
[root@node5 ~]# strings /lib64/libc.so.6 | grep GLIBC
GLIBC_2.2.5
GLIBC_2.2.6
GLIBC_2.3
GLIBC_2.3.2
GLIBC_2.3.3
GLIBC_2.3.4
GLIBC_2.4
GLIBC_2.5
GLIBC_2.6
GLIBC_2.7
GLIBC_2.8
GLIBC_2.9
GLIBC_2.10
GLIBC_2.11
GLIBC_2.12
GLIBC_PRIVATE
```

在安装之前，先在[确认内核版本支持 glibc_2.17](http://man7.org/tlpi/api_changes/#glibc-2.17)

> glibc 2.17 (25 Dec 2012)
> Note: the minimum Linux kernel version to run with this and later glibc versions is Linux 2.6.16.
>
> API changes include the following:
>
> A new secure_getenv() function allows secure access to the environment. It is similar to getenv(3), but returns NULL if running in a set-user-ID/set-group-ID process. Documentation can be found in the secure_getenv(3) manual page.
> The functions clock_getres(), clock_gettime(), clock_settime(), clock_getcpuclockid(), and clock_nanosleep(), moved from the realtime library (librt to the main C library. Consequently, it is no longer necessary to link against the realtime library (cc -lrt) when using these functions. The rationale for this change is explained in glibc bug 14743 ("clock_gettime et al from -lrt always bring in libpthread").

可以看到，系统内核版本为 2.6.32，而 glibc_2.17 支持 2.6.16 以上的内核，因此应该可以安装

```
[root@node5 ~]# uname -a
Linux node5 2.6.32-279.el6.x86_64 #1 SMP Wed Jun 13 18:24:36 EDT 2012 x86_64 x86_64 x86_64 GNU/Linux
```

查看需要安装的 glibc 相关模块

```
[root@node5 ~]# rpm -qa | grep glibc
compat-glibc-headers-2.5-46.2.x86_64
glibc-2.12-1.80.el6.x86_64
compat-glibc-2.5-46.2.x86_64
glibc-devel-2.12-1.80.el6.x86_64
glibc-headers-2.12-1.80.el6.x86_64
glibc-2.12-1.80.el6.i686
glibc-common-2.12-1.80.el6.x86_64
```

下载对应的 glibc 2.17 版本安装包，参考 [(https://gist.github.com/harv/f86690fcad94f655906ee9e37c85b174)](https://gist.github.com/harv/f86690fcad94f655906ee9e37c85b174)，将需要用到的5个安装包，放在同一文件夹内

[x86_64 下载地址](https://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-x86_64/glibc-2.17-55.fc20/) | [i386 下载地址](https://copr-be.cloud.fedoraproject.org/results/mosquito/myrepo-el6/epel-6-i386/glibc-2.17-55.fc20/)

```
[root@node5 glibc]# ll
total 24767
-rw-r--r-- 1 root root  4835780 Feb 17  2015 glibc-2.17-55.el6.i686.rpm
-rw-r--r-- 1 root root  4181172 Feb 17  2015 glibc-2.17-55.el6.x86_64.rpm
-rw-r--r-- 1 root root 14624176 Feb 17  2015 glibc-common-2.17-55.el6.x86_64.rpm
-rw-r--r-- 1 root root  1043692 Feb 17  2015 glibc-devel-2.17-55.el6.x86_64.rpm
-rw-r--r-- 1 root root   677944 Feb 17  2015 glibc-headers-2.17-55.el6.x86_64.rpm
```

**同时安装glibc相关的包**，如果单独安装会出现依赖问题

```
[root@node14 glibc]# rpm -Uvh ./*
warning: ./glibc-2.17-55.el6.i686.rpm: Header V3 RSA/SHA1 Signature, key ID 73ec361c: NOKEY
Preparing...                ########################################### [100%]
   1:glibc-common           ########################################### [ 20%]
/usr/sbin/build-locale-archive: /lib64/libc.so.6: version `GLIBC_2.14' not found (required by /usr/sbin/build-locale-archive)
/usr/sbin/build-locale-archive: /lib64/libc.so.6: version `GLIBC_2.14' not found (required by /usr/sbin/build-locale-archive)
   2:glibc                  ########################################### [ 40%]
   3:glibc-headers          ########################################### [ 60%]
   4:glibc-devel            ########################################### [ 80%]
   5:glibc                  warning: /etc/nsswitch.conf created as /etc/nsswitch.conf.rpmnew
warning: /etc/rpc created as /etc/rpc.rpmnew
########################################### [100%]
warning: /etc/localtime saved as /etc/localtime.rpmsave
```

安装完成后，检查是否已经支持 GLIBC_2.17

```
[root@node5 glibc]# strings /lib64/libc.so.6 | grep GLIBC
GLIBC_2.2.5
GLIBC_2.2.6
GLIBC_2.3
GLIBC_2.3.2
GLIBC_2.3.3
GLIBC_2.3.4
GLIBC_2.4
GLIBC_2.5
GLIBC_2.6
GLIBC_2.7
GLIBC_2.8
GLIBC_2.9
GLIBC_2.10
GLIBC_2.11
GLIBC_2.12
GLIBC_2.13
GLIBC_2.14
GLIBC_2.15
GLIBC_2.16
GLIBC_2.17
GLIBC_PRIVATE
```

**升级成功！**

P.S. RPM 升级会导致时区设置清空，重新设置为当前时区后，和管理节点同步时间即可

```
[root@node5 ~]# ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
[root@node5 ~]# ntpdate node78
17 Jan 20:49:05 ntpdate[26445]: adjust time server 192.168.0.78 offset -0.010161 sec
```

## 升级 gcc

当前 gcc 版本为 4.4.7

```
[root@node5 ~]# gcc --version
gcc (GCC) 4.4.7 20120313 (Red Hat 4.4.7-18)
Copyright (C) 2010 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

为了不影响其他依赖的软件包，我们使用 devtoolset-2 单独安装一份 gcc 4.8.2

```
[root@node5 ~]# wget -O /etc/yum.repos.d/slc6-devtoolset.repo http://linuxsoft.cern.ch/cern/devtoolset/slc6-devtoolset.repo
```

在可以联网的节点下载 devtoolset-2 的安装包

```
[root@manager ~]#  yum install --downloadonly --downloaddir=/histor/software/yum_download devtoolset-2-gcc-c++-4.8.2-15.1.el6
Loaded plugins: aliases, changelog, downloadonly, kabi, presto, product-id, security, subscription-
              : manager, tmprepo, verify, versionlock
Updating certificate-based repositories.
Unable to read consumer identity
Loading support for Red Hat kernel ABI
Setting up Install Process
Resolving Dependencies
--> Running transaction check
---> Package devtoolset-2-gcc-c++.x86_64 0:4.8.2-15.1.el6 will be installed
--> Processing Dependency: devtoolset-2-libstdc++-devel = 4.8.2-15.1.el6 for package: devtoolset-2-gcc-c++-4.8.2-15.1.el6.x86_64
--> Processing Dependency: devtoolset-2-gcc = 4.8.2-15.1.el6 for package: devtoolset-2-gcc-c++-4.8.2-15.1.el6.x86_64
--> Running transaction check
---> Package devtoolset-2-gcc.x86_64 0:4.8.2-15.1.el6 will be installed
--> Processing Dependency: devtoolset-2-runtime for package: devtoolset-2-gcc-4.8.2-15.1.el6.x86_64
---> Package devtoolset-2-libstdc++-devel.x86_64 0:4.8.2-15.1.el6 will be installed
--> Running transaction check
---> Package devtoolset-2-runtime.noarch 0:2.1-4.el6 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

========================================================================================================
 Package                             Arch          Version                 Repository              Size
========================================================================================================
Installing:
 devtoolset-2-gcc-c++                x86_64        4.8.2-15.1.el6          slc6-devtoolset        7.2 M
Installing for dependencies:
 devtoolset-2-gcc                    x86_64        4.8.2-15.1.el6          slc6-devtoolset         21 M
 devtoolset-2-libstdc++-devel        x86_64        4.8.2-15.1.el6          slc6-devtoolset        1.9 M
 devtoolset-2-runtime                noarch        2.1-4.el6               slc6-devtoolset        1.0 M

Transaction Summary
========================================================================================================
Install       4 Package(s)

Total download size: 32 M
Installed size: 71 M
Is this ok [y/N]: y
Downloading Packages:
Setting up and reading Presto delta metadata
Processing delta metadata
Package(s) data still to download: 32 M
(1/4): devtoolset-2-gcc-4.8.2-15.1.el6.x86_64.rpm                                                               |  21 MB     03:40
(2/4): devtoolset-2-gcc-c++-4.8.2-15.1.el6.x86_64.rpm                                                           | 7.2 MB     01:37
(3/4): devtoolset-2-libstdc++-devel-4.8.2-15.1.el6.x86_64.rpm                                                   | 1.9 MB     00:19
(4/4): devtoolset-2-runtime-2.1-4.el6.noarch.rpm                                                                | 1.0 MB     00:08
---------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                   92 kB/s |  32 MB     05:51


exiting because --downloadonly specified
```

切换到计算节点，安装下载的 devtoolset-2

```
[root@node5 yum_download]# rpm -Uvh ./*.rpm
warning: ./devtoolset-2-gcc-4.8.2-15.1.el6.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 1d1e034b: NOKEY
Preparing...                ########################################### [100%]
   1:devtoolset-2-runtime   ########################################### [ 25%]
libsemanage.get_home_dirs: mails homedir /var/mails or its parent directory conflicts with a file context already specified in the policy.  This usually indicates an incorrectly defined system account.  If it is a system account please make sure its uid is less than 500 or its login shell is /sbin/nologin.
   2:devtoolset-2-gcc       ########################################### [ 50%]
   3:devtoolset-2-libstdc++-########################################### [ 75%]
   4:devtoolset-2-gcc-c++   ########################################### [100%]
```

启用 devtoolset-2

```
[root@node5 ~]# source /opt/rh/devtoolset-2/enable
```

查看 gcc 版本

```
[root@node5 ~]# gcc --version
gcc (GCC) 4.8.2 20140120 (Red Hat 4.8.2-15)
Copyright (C) 2013 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

**成功！**