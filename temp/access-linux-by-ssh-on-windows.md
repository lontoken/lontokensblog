title: Windows下通过SSH远程访问Linux
date: 2015-04-07
tags: SSL,libssh2,Openssl
categories: SSL
---

Windows下通过SSH远程访问Linux  
====

# 前言
在Linux下的程序部署、程序运行时控制等情景中，希望能在Windows下以可视化的方式操作。  
在Windows下访问/控制Linux最好的方式无疑是通过SSH。 
本文将详细说明Windows下使用SSH访问Linux，SSH库选用开源的libssh2，其openssl和zlib库。下面将详细说明它们的编译和使用，并简单的演示对Linux的访问/控制。   

<!--more-->

#环境和版本#
>    操作系统：Windows 7 64位，Ubuntu14.04 32bit   
>    libssh2版本: libssh2-1.5.0  
>    OpenSSL版本: openssl-1.0.2a  
>    zlib版本: zlib-1.2.8  
>	 工具：Visual Studio 2010  


# OpenSSL安装
## 二进制安装包安装
下载地址：<a>http://slproweb.com/download/Win32OpenSSL-1_0_2a.exe</a>；  
如安装在 e:\OpenSSL-Win32 目录下；   
并选择将OpenSSL的DLL复制到Windows的系统目录下；   

## 源码安装
略，直接使用了下载的二进制安装包。 


# zlib源码编译
zlib下载地址：<a>http://zlib.net/zlib-1.2.8.tar.gz</a>；  
将源文件解压到指定目录，如 e:\WorkStation\HelloWorldForSSH\zlib-1.2.8\;   
打开"Visual Studio 命令行工具"，切换到上述目录；  
cd contrib/masmx86;   
bld_ml32.bat;   
使用VS2010打开 contrib\vstudio\vc10\zlibvc.sln；  
生成zlibvc；  
contrib/vstudio/vc10/x86/ZlibDllDebug/目录下会有 zlibwapid.dll和zlibwapid.lib；  


# zlib源码编译
libssh2下载地址：<a>http://www.libssh2.org/download/libssh2-1.5.0.tar.gz</a>； 
将源文件解压到指定目录，如 e:\WorkStation\HelloWorldForSSH\libssh2-1.5.0\;  
修改libssh2下的win32目录下config.mk，修改openssl和zlib的目录;  

```shell   
!if "$(OPENSSLINC)" == ""
OPENSSLINC=E:\OpenSSL-Win32\include
!endif

!if "$(OPENSSLLIB)" == ""
OPENSSLLIB=E:\OpenSSL-Win32\lib
!endif

!if "$(ZLIBINC)" == ""
ZLIBINC=e:\WorkStation\HelloWorldForSSH\zlib-1.2.8
!endif

!if "$(ZLIBLIB)" == ""
ZLIBLIB=e:\WorkStation\HelloWorldForSSH\zlib-1.2.8
!endif
```

打开"Visual Studio 命令行工具"，切换到上述目录；  
在此目录下执行 nmake /f NMakefile ；   
在根目录下，会生成 libssh2.dll ；  

此时只生成了 dll，若还需要生成 lib，可以使用VS2010打开 win32/libssh2.sln;   
将 e:\WorkStation\HelloWorldForSSH\zlib-1.2.8;E:\OpenSSL-Win32\include 添加到附加包含目录；  
将 e:\WorkStation\HelloWorldForSSH\zlib-1.2.8;E:\OpenSSL-Win32\lib 添加到附加库目录； 
将e:\WorkStation\HelloWorldForSSH\zlib-1.2.8\contrib\vstudio\vc10\x86\ZlibDllDebug/目录下的 zlibwapid.dll和zlibwapid.lib复制到e:\WorkStation\HelloWorldForSSH\zlib-1.2.8，并重命名为zlib.dll和zlib.lib；   
生成 libssh2 项目，此时win32\Debug_dll目录下已经生成libssh2.dll和libssh2.lib；  

至此使用SSH的依赖库均已经生成完成； 


# 简单使用SSH
程序详情参见<a>https://github.com/lontoken/HelloWorldForSSH</a>下的sshtest；  


# 参考资源
>	<a>http://developer.covenanteyes.com/building-openssl-for-visual-studio/</a>

>	<a>http://www.cnblogs.com/godboy1989/p/4064328.html</a>