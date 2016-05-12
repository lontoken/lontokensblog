title: A template class for binding C++ to Lua(翻译)
date: 2014-01-06
tags: lua,翻译,C++
categories: lua
---


# A template class for binding C++ to Lua
英文原文：[A template class for binding C++ to Lua](http://www.lua.org/notes/ltn005.html)

最近在研究C/C++和Lua的交互问题，顺便看了下luna，自己尝试着翻译了此文，以供分享，初始翻译，权当练习。  

## 摘要
此文介绍了一种将C++类绑定到Lua的方法。Lua没有直接提供此方法，而是通过底层的C接口和扩展机制来实现。我所描述的方法使用了Lua的C接口、C++模板和Lua的扩展机制，构建了一个小巧、简单且高效的提供类注册服务的静态模板类。这个方法对你的类只有一个小小的要求，即只有签名为 int(T::\*)(lua_State\*) 的成员函数能被注册。但是，正如我将要展示的，这个限制也能被克服。The end result is a clean interface to register classes, and familiar Lua table semantics of classes in Lua。此处描述的解决方案依赖于一个我命名为[Luna](http://lua-users.org/files/wiki_insecure/users/lpalozzi/luna.tar.gz)的模板类。
<!--more-->

## 问题
Lua的接口的设计，不能注册C++类到Lua中，只提供了注册签名为 int()(lua_State\*) 的C函数。实际上，这是Lua支持注册的唯一C数据类型。为了注册其它类型，你需要使用Lua提供的扩展机制，如tag methods、closures等。为了创建注册C++类到Lua的方案，必须使用这些扩展机制。

## 方案
此方案主要有4个元素：类注册、对象实例化、成员函数调用和垃圾回收。

类注册是通过以类的名字注册一个表构造函数(a table constructor function with the name of the class)。表构造函数是一个在lua_State中注册一个表的静态模板函数。

注释：静态类成员函数是和C函数相兼容的，如果它们的签名相同，则我们可以在Lua中注册它们。下面的代码片段是一个模板类的成员函数，T 是待注册的类。

```cpp
static void Register(lua_State* L) {
    lua_pushcfunction(L, &Luna<T>::constructor);
    lua_setglobal(L, T::className);

    if (otag == 0) {
        otag = lua_newtag(L);
        lua_pushcfunction(L, &Luna<T>::gc_obj);
        lua_settagmethod(L, otag, "gc"); /* tm to release objects */
    }
}
```

对象实例化是通过passing any arguments the user passed to the table constructor function to the constructor of the C++ object，创建一个代表对象的表，注册一些类的成员函数到表上，最后将表返回给Lua。对象的指针做为userdata保存在表中，其对应的索引为0。成员函数的序号做为闭包的值保存在所有函数中。More on the member function map later。

```cpp
static int constructor(lua_State* L) {
    T* obj= new T(L); /* new T */
    /* user is expected to remove any values from stack */

    lua_newtable(L); /* new table object */
    lua_pushnumber(L, 0); /* userdata obj at index 0 */
    lua_pushusertag(L, obj, otag); /* have gc call tm */
    lua_settable(L, -3);

    /* register the member functions */
    for (int i=0; T::Register[i].name; i++) {
        lua_pushstring(L, T::Register[i].name);
        lua_pushnumber(L, i);
        lua_pushcclosure(L, &Luna<T>::thunk, 1);
        lua_settable(L, -3);
    }
    return 1; /* return the table object */
}
```

不像C函数，C++成员函数需要类的对象来调用。成员函数的调用是通过thunks函数，它由一个对象指针和成员函数指针来做实际的调用。成员函数指针是保存在函数的闭包值里，对应成员函数的哈希表中，对象的指针是表的序号0对应的值。注意，在Lua中所有的类函数都是通过下面的函数来注册。

```cpp
static int thunk(lua_State* L) {
    /* stack = closure(-1), [args...], 'self' table(1) */
    int i = static_cast<int>(lua_tonumber(L,-1));
    lua_pushnumber(L, 0); /* userdata object at index 0 */
    lua_gettable(L, 1);
    T* obj = static_cast<T*>(lua_touserdata(L,-1));
    lua_pop(L, 2); /* pop closure value and obj */
    return (obj->*(T::Register[i].mfunc))(L);
}
```

垃圾回收是通过在设置表中userdata的垃圾回收标志方法。当垃圾回收被触发时，'gc'标志方法就会被调用以删除对象。'gc'标志方法是在类注册时通过一个新的标志注册的。在上面的实例化时，userdata就通过一个标签来标志。

```cpp
static int gc_obj(lua_State* L) {
    T* obj = static_cast<T*>(lua_touserdata(L, -1));
    delete obj;
    return 0;
}
```

注意，有一些规则类需要遵守：  
1. 必须一个公开的构造函数，它有一个lua_State*参数;  
2. 被注册的成员函数签名必须是 int(T::*)(lua_State*);  
3. 必须有一个 public static const char[] className成员;  
4. 必须有一个 public static const Luna<T>::RegType[] Register成员；  

注释：这些要求是我所选择的设计的要求，你可以使用不同的接口，只需要代码做很小的改动。  

Luna<T>::RegType 是一个函数哈希表。name 是被注册的 mfunc 函数的名字。  

```cpp
struct RegType {
    const char* name;
    const int(T::*mfunc)(lua_State*);
};
```

这里有一个注册C++类到Lua中的例子。Luna<T>::Register() 调用会注册这个类，它也是这个模板类唯一要求的公开接口。要在Lua中使用此类，你可通过调用表的构造函数来创建此类的一个实例。  

```cpp
class Account {
    double m_balance;
    public:
    Account(lua_State* L) {
        /* constructor table at top of stack */
        lua_pushstring(L, "balance");
        lua_gettable(L, -2);
        m_balance = lua_tonumber(L, -1);
        lua_pop(L, 2); /* pop constructor table and balance */
    }

    int deposit(lua_State* L) {
        m_balance += lua_tonumber(L, -1);
        lua_pop(L, 1);
        return 0;
    }
    int withdraw(lua_State* L) {
        m_balance -= lua_tonumber(L, -1);
        lua_pop(L, 1);
        return 0;
    }
    int balance(lua_State* L) {
        lua_pushnumber(L, m_balance);
        return 1;
    }
    static const char[] className;
        static const Luna<Account>::RegType Register
    };

    const char[] Account::className = "Account";
    const Luna<Account>::RegType Account::Register[] = {
        { "deposit",  &Account::deposit },
        { "withdraw", &Account::withdraw },
        { "balance",  &Account::balance },
        { 0 }
    };

    //[...]

/* Register the class Account with state L */
Luna<Account>::Register(L);
```

```lua
-- In Lua
-- create an Account object
local account = Account{ balance = 100 }
account:deposit(50)
account:withdraw(25)
local b = account:balance()
```

Account实例的表如下：  

```shell
0 = userdata(6): 0x804df80
balance = function: 0x804ec10
withdraw = function: 0x804ebf0
deposit = function: 0x804f9c8
```


## 说明
也许有些人不喜欢使用C++模板，但在此处却是合适的。它们提供了一个起初看起来复杂但快速严密的解决方案。作为使用模板的回报，类是类型安全的，例如，它不可能在成员函数哈希表中混合不同类型的成员函数，因为编译器会报怨的。此外，静态模板类的设计，使它容易使用，在你做完事后没有模板实例需要清除。  

thunk机制是类的核心，因为它转换了函数调用。它通过将表调用的函数对应的对象指针索引到成员函数哈希表中。(Lua表的函数调用 table:function() 是 table.function(table) 调用的语法糖)。这个调用会将表最先压入栈中，接着是参数。成员函数的索引做为一个闭包值压入栈的最后。最开始，我将对象指针也做为一个闭包值，这意味着有2个闭包值，一个void *的对象指针，一个成员函数的索引，这样花费更多，但访问更快。当然，表中用于垃圾回收的userdata对象还是需要的。最后，我选择了将对象指针索引到表中，以节省资源，但增加调用的时间消耗。

实际上，这个实现只利用了Lua扩展机制很少的特性，闭包来保持成员函数的索引，'gc'标志成员用于垃圾回收，表的构造函数做函数注册和成员函数的调用。

为什么只允许签名为 int(T::\*)(lua_State\*) 成员函数被注册？这允许你的成员函数和Lua直接交互，接收Lua传入的参数并返回值到Lua，调用任何的Lua Api函数等。此外，它提供了与注册到Lua中的C函数相同的签名，这使得想使用C++的人更方便。  

## 不足
这个模板类方案只能绑定特定签名的成员函数。如果你有之前写的类，或想在Lua和C++的环境中都能使用，这个方案对你来说可能不是最好的。理论上这不是一个问题。使用代理模式，我们封装实际的类，并且代理任何对目标对象的调用。代理类的成员函数强迫Lua的参数和返回值，并且代理对目标对象的调用。你将在Lua中注册代理类，而不是实际类。Additionally, you may use inheritance as well where the proxy class inherits from the base class and delegates the function calls up to the base class, but with one caveat, the base class must have a default constructor; you cannot get the constructor arguments from Lua to the base class in the proxy's constructor initializer list。代理模式解决了我们想在Lua和C++环境中使用的问题，但它要求我们写代理类并且维护它们。

对象都是简单的创建，但需要给用户创建对象时更多的控制。比如，用户也许希望注册一个单例类。一个单例需要用户实现一个返回一个对象的静态 create() 成员函数。这种方式，用户可以实现单例模式，简单的通过new分配对象或者其它方法。constructor函数需要做修改来调用create()而不是使用new来获取一个对象的指针。这对类要求了更多的约束但更灵活。 A "hook" for garbage collection may be of use to some as well.

## 总结
这个文章说明了一个绑定C++类到Lua的简单方法。这个实现如此简单，你可以选择修改它来满足你的目的，也满足了一般的需求。也有一些其它的工具来绑定C++类到Lua中，如 tolua、SWIGLua和一些其它像本文的简单实现。根据它们的优点、缺点来解决你特定的问题。希望本文能给你一些解决此类问题的提示。

模板类的所有源码大概在70行左右，可以Lua的[add-ons](http://www.lua.org/addons.html)面面获取。

## 参考
[1] R. Hickey, [Callbacks in C++ using template functors](http://www.bestweb.net/~rhickey/functor.html), C++ Report February 95  

Last update: Wed Mar 12 11:51:13 EST 2003 by lhf.

***
欢迎转载，原文地址：[http://www.lontoken.com/a-template-class-for-binding-cpp-to-lua-translate](http://www.lontoken.com/a-template-class-for-binding-cpp-to-lua-translate)
***

