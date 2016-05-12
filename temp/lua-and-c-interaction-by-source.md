title: Lua和C交互之Lua调用C--源码分析
date: 2014-01-05
tags: Lua,C,lua和C,lua源码
categories: Lua
---

# Lua和C交互--源码分析
最近在读Lua的源码，之前在Lua中调用C导出的函数和导出的userdata，觉得很方便，没去深究，不知道自己写的代码在栈层面是个如何情形，为什么lua中的一个函数调用，就能进入dll库的对应里。想弄明白这些问题，通过google搜索，大多没有深入的分析，不能令人满意。现在借此机会，将整个流程弄明白，顺便分享。
<!--more-->

Lua和C交互有两种：  
>   C访问Lua;  
>   Lua访问C;  
前者通过lua.h中lua_xxx的函数和宏，还有lauxlib.h中luaL_xxx的函数和宏来访问Lua，此问题相对来说简单此，以后有机会再详谈。  
对于后者，则涉及到Lua代码转换为Lua字节码，还待我们深入源码细细诉来。


## 示例代码
为便于说明，我们以如下场景来继续：  
>   新用户注册，我们需要录入用户的基本信息，并返回一个递增的用户ID。Lua将用户信息传入Dll中的C/C++的函数，函数将信息打印，并返回一个用户ID，Lua获取函数返回的ID，并打印。  

C++代码：  

```cpp
#include <stdio.h>
#include <lua.hpp>

#if defined(_WINDOWS) || defined(WIN32) || defined(_WIN32)
#include <windows.h>
#else
#include <linux/kernel.h>
#endif

#define MIN(x, y) min(x, y)
#define MAX(x, y) max(x, y)

#define PAI(L, n) (long)(lua_isnumber(L, n) ? luaL_checknumber((L), (n)) : 0)       //取整数的参数
#define PAD(L, n) (double)(lua_isnumber(L, n) ? luaL_checknumber((L), (n)) : 0.0)   //取浮点型参数
#define PAC(L, n) (char)(lua_isnumber(L, n) ? luaL_checknumber((L), (n)) : 0)       //取单字符的参数

#define PAS(L, n) (lua_isstring(L, n) ? luaL_checkstring((L), (n)) : "")            //取字符串参数
#define PASL(L, n) (lua_isstring(L, n) ? lua_strlen(L, n) : 0)                      //取字符串的长度
#define CS(f, L, n) memcpy(f, PAS(L, n), \
    MIN(sizeof(f), PASL(L, n))); f[sizeof(f) - 1] = 0      //复制字符串参数

#define PN(L, f) lua_pushnumber(L, f)
#define PINT(L, f) lua_pushinteger(L, f)
#define PS(L, f) lua_pushlstring(L, f, MIN(sizeof(f), strlen(f)))


//添加用户
//@param username, age
//@return id
extern "C" int AddUser(lua_State *L)
{
    static int id = 0;
    char name[20] = {0};
    int age = 0;
    CS(name, L, 1);
    age = PAI(L, 2);
    printf("name=%s, age=%d\n", name, age);

    PINT(L, ++id);
    return 1;
}

static const luaL_reg LuaCallCFunctions [] =
{
    {"AddUser", AddUser},
    {NULL, NULL}
};

extern "C" __declspec(dllexport) int luaopen_LuaCallC(lua_State* L) 
{
    const char* libName = "LuaCallC";
    luaL_register(L, libName, LuaCallCFunctions);
    return 1;
}
```

Lua代码:  

```cpp
package.cpath = ".\\?.dll"
local LuaCallC = require("LuaCallC");

local id = LuaCallC.AddUser("linmeimei", 18);
print("id=" .. id);
```

运行结果：  

```shell
>lua adduser.lua
name=linmeimei, age=18
id=1
```
