title: CentOS7上安装web.py
date: 2015-10-31
tags: web.py,CentOS7,安装
categories: web.py
---


CentOS7上安装web.py  
====

# 环境和版本
>    操作系统：CentOS 7 64位  
>    python版本：2.7.5


# EasyInstall安装
用来下载安装Python一个公共资源库PyPI的相关资源包的。有了EasyInstall，python的包管理会简单很多。  

<!--more-->

```shell
#wget https://bootstrap.pypa.io/ez_setup.py
#python ez_setup.py
```


# web.py安装

```shell
#easy_install web.py
```


# web.py测试
创建文件 hello.py  


```python 
#!/usr/bin/env python

import web

urls = ("/.*", "hello")
app = web.application(urls, globals())

class hello:
    def GET(self):
        return 'Hello, world!'

if __name__ == "__main__":
    app.run()
```

使用命令

```shell
#python hello.py
```

访问站点： http://localhost:8080 即可。
