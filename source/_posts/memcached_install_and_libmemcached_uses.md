title: memcached安装和libmemcached的使用
date: 2014-11-24
tags: memcached,安装,libmemcached
categories: memcached
---

memcached安装和libmemcached的使用  
====

# 环境和版本
>    操作系统：Ubuntu14.04 32bit  
>    libevent版本: 2.0.21  
>    memdatach版本: v1.4.21  


# libevent安装

```shell
#wget http://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz
#tar -xvzf libevent-2.0.21-stable.tar.gz
#cd libevent-2.0.21-stable
#./configure -prefix=/usr
#make
#make install
```

<!--more-->

查看是否安装成功:  

```shell
#ls /usr/lib/ | grep  libevent
```


# memcached安装

```shell  
#wget wget http://www.memcached.org/files/memcached-1.4.21.tar.gz
#tar -xvzf memcached-1.4.21.tar.gz
#cd memcached-1.4.21
#./configure -with-libevent=/usr
#make
#make install
```

查看是否安装成功: 

```shell
#ll /usr/local/bin
```


#memcached启动#

```shell
#/usr/local/bin/memcached -d -u root -m 512 127.0.0.1 -p 11211
```

查看侦听端口和进程信息：  

```shell
#netstat -a |grep 11211
#ps -ef | grep memcached
```


#测试memcached#
连接memcached最简单的方法是通过telnet。  

```shell
#telnet 127.0.0.1 11211
```

查看memcached的状态(telnet下执行):    

```shell
stats
```

键值简单的设置、查看和删除(telnet下执行):  

```shell
set user_id 0 0 5
12345
get user_id
delete user_id
get user_id
```

PS:退出telnet，可以键入alt+] q 


# libmemcached安装

```shell  
#wget https://launchpad.net/libmemcached/1.0/1.0.18/+download/libmemcached-1.0.18.tar.gz
#tar -xvzf libmemcached-1.0.18.tar.gz
#cd libmemcached-1.0.18
#./configure
#make
#make install
```

查看libmemcached是否安装成功:  

```shell
#ls /usr/local/lib | grep libmemcached
```


# 使用C++通过libmemcached连接memcached#
C++源文件 libmemcachedtest.cpp  

```cpp
#include <iostream>
#include <string>
#include <libmemcached/memcached.h>

using namespace std;

int main(int argc, char *argv[])
{
    //connect server
    cout << "test start" << endl;
    memcached_st *memc;
    memcached_return rc;
    memcached_server_st *server;
    uint32_t  flags;

    memc = memcached_create(NULL);
    cout << "append start" << endl;
    server = memcached_server_list_append(NULL, "localhost", 11211, &rc);
    if(rc != MEMCACHED_SUCCESS){
        cout << "memcached_server_list_append failed. rc=" << rc << endl;
        return -1;
    }

    rc = memcached_server_push(memc, server);
    if(rc != MEMCACHED_SUCCESS){
        cout << "memcached_server_push failed. rc=" << rc << endl;
        memcached_server_free(server);
        return -2;
    };

    memcached_server_list_free(server);

    string key = "key";
    string value = "value";
    size_t value_length = value.length();
    size_t key_length = key.length();

    //Save data
    cout << "save data" << endl;
    rc = memcached_set(memc, key.c_str(), key_length, value.c_str(), value_length, 0, flags);
    if(rc == MEMCACHED_SUCCESS){
        cout << "save data sucessful, key=" << key << ",value=" << value <<endl;
    }else{
        cout << "save data faild, rc=" << rc <<endl;
    }

    //get data
    cout << "get data" << endl;
    char* result = memcached_get(memc, key.c_str(), key_length, &value_length, &flags, &rc);
    if(rc == MEMCACHED_SUCCESS){
        cout << "get value sucessful, result=" << result <<endl;
    }else{
        cout << "get value faild, rc=" << rc <<endl;
    }

    //delete data
    cout << "delete data" << endl;
    rc = memcached_delete(memc, key.c_str(), key_length, 0);
    if(rc == MEMCACHED_SUCCESS){
        cout << "delete key sucessful. key=" << key << endl;
    }else{
        cout << "delete key faild, rc=" << rc <<endl;
    }

    //free
    memcached_free(memc);
    cout << "test end." << endl;
    return 0;
}
```

编译前需要设置LD_LIBRARY_PATH环境变更，以使libmemcached能被找到。 

```shell
$export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib/
```
 
编译并执行：

```shell 
$g++ -std=c++11 -o libmemcachedtest libmemcachedtest.cpp -lmemcached
$./libmemcachedtest
``` 
 
如果一切顺利，输出如下： 

```shell
test start
append start
save data
save data sucessful, key=key,value=value
get data
get value sucessful, result=value
delete data
delete key sucessful. key=key
test end.
```

-----------------
本文结束，若有错误和疑问，欢迎交流(邮件：lontoken@gmail.com)。  
