title: DuiVision学习笔记
date: 2016-03-04
tags: DuiVision,学习
categories: DuiVision
---


# DuiVision学习笔记
## DuiVision说明
DirectUI 技术一般是指将所有的界面控件都绘制在一个窗口上，这些控件的逻辑和绘图方式都必须自己进行编写和封装，而不是使用 Windows 控件，所以这些控件都是无句柄的。
DirectUI 技术需要解决的主要问题如下：
1. 窗口的子类化，截获窗口的消息。
2. 封装自己的控件，并将自己的控件绘制到该窗口上。
3. 封装窗口的消息，并分发到自己的控件上，让自己的控件根据消息进行响应和绘制。
4. 根据不同的行为发送自定义消息给窗口，以便程序进行调用。
5. 一般窗口上控件的组织使用 XML 来描述。通常 DirectUI 的界面库都采用 XML 配置文件+图片+控制脚本（Lua、 Javascript 等）的开发方式，非常类似于 Web 程序的开发方式，当然这里面控制脚本也可以直接使用 C++代码来实现。
<!--more-->

## XML资源文件定义
基于 DuiVision 界面库的程序，需要有一个默认的资源定义 XML 文件，此文件默认的位置是exe 文件所在路径下的 xml\resource.xml 文件，如果使用了 zip 压缩文件来保存所有资源文件，则此文件的位置是在压缩包中的 xml\resource.xml 文件。此文件中可以定义程序的全局配置、XML 文件、字体、图片、文字等资源，示例如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<root>
    <!--系统配置-->
    <res type="cfg" name="defaultStyle" value=""/>
    <res type="cfg" name="logfile" value="demoui.log"/>
    <res type="cfg" name="loglevel" value="1"/>
    <res type="cfg" name="appMutex" value="MUTEX_DUIVISION_DEMO"/>
    <res type="cfg" name="enableDragFile" value="1"/>
    <res type="cfg" name="trayDbClickMsg" value="0"/>
    <!--风格设置-->
    <res type="style" name="default" value=""/>
    <res type="style" name="qq" value="qq"/>
    <!--嵌套的XML资源定义文件-->
    <res type="res" lang="zh-cn" file="xml\def_string_zh-cn.xml"/>
    <res type="res" lang="en-us" file="xml\def_string_en-us.xml"/>
    <!--XML资源-->
    <res type="xml" name="dlg_main" file="xml\dlg_main.xml"/>
    <res type="xml" name="dlg_skin" file="xml\dlg_skin.xml"/>
    <res type="xml" name="dlg_about" file="xml\dlg_about.xml"/>
    <res type="xml" name="dlg_login" file="xml\dlg_login.xml"/>
    <res type="xml" name="dlg_msgbox" file="xml\dlg_msgbox.xml"/>
    <!--字体资源-->
    <res type="font" lang="zh-cn" name="default" font="微软雅黑" size="12" bold="false"/>
    <res type="font" lang="zh-cn" name="big" font="微软雅黑" size="14" bold="true" italic="false" underline="false" strikeout="false"/>
    <!--图片资源-->
    <res type="img" name="IDB_MAIN_FRAME" file="skins\WindowsBack.png"/>
    <res type="img" name="IDB_BT_CLOSE" file="skins\BT_CLOSE.png"/>
    <res type="img" name="IDB_BT_MIN" file="skins\BT_MIN.png"/>
    <res type="img" name="IDB_BT_MENU" file="skins\BT_MENU.png"/>
    <res type="img" name="IDB_BT_SKIN" file="skins\BT_SKIN.png"/>
    <res type="img" name="IDB_TAB_1" file="skins\Tab1.png"/>
    <res type="img" name="IDB_TAB_2" file="skins\Tab2.png"/>
    <res type="img" name="IDB_ICON_INFO" file="skins\info.png"/>
    <res type="img" name="IDB_ICON_WARN" file="skins\warning.png"/>
    <res type="img" name="IDB_ICON_ERROR" file="skins\error.png"/>
    <res type="img" name="IDB_MENU_UPDATE" file="skins\MENU_UPDATE.png"/>
    <!--字符串资源-->
    <res type="str" lang="zh-cn" name="APP_NAME" value="DUI测试程序"/>
    <res type="str" lang="zh-cn" name="APP_VER" value="1.0.0.1"/>
    <res type="str" lang="zh-cn" name="OK" value="确定"/>
    <res type="str" lang="zh-cn" name="CANCEL" value="放弃"/>
    <res type="str" lang="zh-cn" name="LOGIN" value="登录"/>
</root>
```

说明：
1. 全局配置定义
Xml 的 type 是 cfg，目前支持的配置如下：
logfile – 日志文件名，是相对 exe 的路径的文件名，如果未定义，则不会生成日志文件
loglevel – 日志级别，1 表示调试级别，2 表示信息级别，4 表示错误级别，8 表示致命级别
defaultStyle – 默认的风格，resource.xml 中的每个资源定义都可以加一个 style 属性，通过
style 属性指定这条资源定义是针对哪种风格的，只有和当前的风格相同的资源或者默认风格的资源才会被加载，资源定义中指定风格的例子如下：

```xml
<res type="xml" style="qq" name="dlg_main" file="xml\dlg_wnd.xml" />
```

appMutex – 应用程序互斥量的名字，如果指定了此变量，则应用程序只能创建一个运行的进程，第二个进程运行时候判断如果存在此名字的互斥量，则退出；
enableDragFile – 是否允许拖拽一个图片文件到程序窗口来指定当前使用的背景图片；
trayDbClickMsg – 在托盘图标双击是否发送消息，用于重新定义托盘图标双击的行为，托盘图标双击的默认行为是打开程序的主窗口，如果此变量设置为 1，则双击托盘图标会给应用程序发送一个 DUI 消息，消息类型为 MSG_TRAY_DBCLICK，消息的发送方 ID 和名字分别是TRAY_ICON 和 NAME_TRAY_ICON，可以在 DUI 消息处理类中增加对此消息的处理，来实现自定义的双击动作；

2. 风格定义
type 是 style，name 是风格的名字，仅用于说明，value 是风格的值，每个资源定义中的 style对应的是风格的 value 部分。
资源 XML 中所有的定义都可以加一个 style 属性来指定当前定义是针对特定风格的，如果和当前的风格一致，则使用此定义覆盖之前的未指定风格的同名的定义。

3. 资源文件定义
Type 是 res，用于加载嵌套的资源定义文件，file 是对应的文件路径名，是相对 exe 所在的路径，通过 lang 属性可以指定此资源文件仅针对哪种语言。

4. xml 文件定义
Type 是 xml，name 是其他地方引用时候的名字，file 是对应的文件路径名，是相对 exe 所在的路径。

5. 字体定义
Type 是 font，lang 表示是针对哪种语言的字体。其他属性说明：
> name – 字体定义名，在其他地方用名字进行引用
> font – 字体名
> size – 文字大小
> bold – 是否粗体（true|false）
> italic– 是否斜体（true|false）
> underline – 显示下划线（true|false）
> strikeout – 显示删除线（true|false）
> os – 表示此字体定义适用于哪些操作系统，例如 os="winxp,win7"表示此定义仅针对 xp 和win7 操作系统，可用的操作系统名字符串包括 win98、 winme、 winnt、 win2000、 winxp、win2003、 vista、 win7、 win8。

6. 图片资源定义
Type 是 img，name 是其他地方引用时候的名字，file 是对应的文件路径名，是相对 exe 所在的路径。

7. 字符串资源定义
Type 是 str，lang 表示是针对哪种语言的字体，name 是其他地方引用时候的名字，value 是字符串内容。
定义的字符串资源可以在各个空间的 title 属性中引用，引用时候要用[]包围，例如title=”[APP_NAME]”就可以在 title 中引用 APP_NAME 对应的字符串。

## XML对话框文件定义
程序中所有界面都是基于对话框或菜单等窗口的，每个对话框都需要有一个 XML 定义文件，用于描述对话框中的内容，对话框中主要是组成对话框的各个控件的定义，对话框的 XML定义示例如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<dlg name="dlg_about" title="MsgBox" width="450" height="230" appwin="1" resize="1" translucent="245" frame="" bkimg="skin:SKIN_PIC_7" crbk="000000">
    <base>
        <imgbtn name="button.close" pos="-45,0,-0,29" skin="IDB_BT_CLOSE" show="1"/>
        <text name="title" crtext="FFFFFF" crmark="800000" font="big" pos="10,5,200,25" title="关于[APP_NAME]" mask="[APP_NAME]" response="0" show="1"/>
    </base>
    <body>
        <area name="area-1" pos="0,0,-0,40" begin-transparent="100" end-transparent="30"/>
        <img name="icon" pos="25,45" width="128" height="128" image="skins\scriptnet.jpg" mode="normal" framesize="0" response="0" show="1"/>
        <text crtext="000000" pos="170,45,-25,65" title="[APP_NAME]
[APP_VER]"/>
        <text crtext="000000" pos="170,65,-25,85" title="2013-2014"/>
        <linkbtn name="linkbtn1" crtext="800000" pos="170,100,-25,130" show="1" DuiVision 开发手册 9 01. 1 title="http://www.blueantstudio.net" href="http://www.blueantstudio.net"/>
        <button name="button.ok" skin="IDB_BT_DEFAULT" title="[OK]" pos="-100,-30,-20,-6" show="1"/>
    </body>
</dlg>
```

说明：
dlg 标签是对话框自身一些属性的描述，可以设置对话框的大小、 背景图片、 蒙版图片、透明度、应用程序窗口属性、改变大小属性等；
base 标签下面的内容都是属于对话框的基础控件，一般可以把对话框的标题和关闭按钮等放在基础控件部分定义；
body 标签下面的内容是属于对话框的普通控件，除了基础控件之外，其他内容都放在 body标签下面定义。
base 和 body 下面都是具体控件的定义描述，可以参考后面关于每个控件的属性说明。

## XML菜单文件定义


## 从ZIP压缩文件中加载资源
DuiVision 支持将所有的图片和 XML 资源文件放在一个 zip 格式的压缩文件中，如果使用 zip
格式的资源文件，需要在主程序代码中初始化部分指定使用的压缩文件的文件名。
如果使用 zip 资源文件，则 resource.xml 文件的位置默认是放在 zip 文件中的 xml 子目录下。
建议 zip 文件按照 xml、 skins 这样的子目录来压缩，见下面的压缩文件示例：

<img src="{{site.url}}/assets/11ab6a76-84f5-4e5d-81a7-d2eab7c26cea.png" />

有 zip 资源文件的情况下，资源文件的加载并不一定是加载的 zip 文件中的内容，加载的优先级如下：
1） 如果只有 zip 压缩文件，没有非压缩的 xml 和 skins 目录，则只会加载 zip 文件中的内容；
2） 如果 zip 压缩文件和非压缩的 xml 和 skins 目录同时存在， 则优先加载非压缩的 xml 和 skins目录中的文件，对应的文件不存在的情况下才去 zip 文件中查找是否存在并加载。
之所以这样定义是方便通过非压缩的文件替换压缩文件中部分内容，以及方便调试和发布工程，调试阶段可以直接修改非压缩的目录中文件，不用每次修改之后都要再打一次压缩包。
说明：zip 资源文件中仅支持包含 xml、 png、 bmp 类型的文件，其他文件无法加载，如果有其他类型的文件，请不要放在 zip 文件中，应该单独放在外部目录中加载。

## 创建主程序的方法（人工创建）
1. 创建一个基于 DuiVision 的界面程序是比较简单的，在 VC 中创建一个 MFC 对话框工程，注意工程要使用 Unicode 库：

<img src="{{site.url}}/assets/a7ae290c-b98b-4220-a072-c664c4081e60.png" />

<img src="{{site.url}}/assets/81a43411-0be4-4111-a3dd-6359601612aa.png" />

工程创建之后，需要将默认对话框资源中的几个按钮和文字都删除，变成一个干净的对话框

2. 设置 DuiVision 的头文件和 lib 文件目录
将 DuiVision 的头文件和 lib 文件放在某个位置，并在工程的头文件和 lib 文件路径定义部分添加相应的目录。
然后在 stdafx.h 文件中添加如下几行对 DuiVision 的头文件和 lib 文件的引用。
`#include "DuiVision.h"`

3. DuiVision 库的初始化以及主窗口的定义
在主程序的 App 类 InitInstance()函数中添加 DuiVision 库的引用代码，示例代码如下：

```cpp 
// 初始化DuiVision界面库,可以指定语言,dwLangID为表示自动判断当前语言
// 1116是应用程序ID，每个DUI应用程序应该使用不同的ID
// ID主要用于进程间通信传递命令行时候区分应用
// IDD_DUIVISIONDEMO_DIALOG是工程中创建的那个对话框资源ID，所有窗口都是共用此ID的
// 第三个参数可以指定资源文件名，如果不指定默认使用xml\resource.xml的定义进行资源的加载，资源文件名的定义方式参考下面的说明
DWORD dwLangID = 0;
new DuiSystem(m_hInstance, dwLangID, _T("DuiVisionDemo.ui"), 1116, IDD_DUIVISIONDEMO_DIALOG, "");
// 创建主窗口
CDlgBase* pMainDlg = DuiSystem::CreateDuiDialog(_T("dlg_main"), NULL, _T(""), TRUE);
// 给主窗口注册事件处理对象
CDuiHandlerMain* pHandler = new CDuiHandlerMain();
pHandler->SetDialog(pMainDlg);
DuiSystem::RegisterHandler(pMainDlg, pHandler);
// 初始化提示信息窗口
DuiSystem::Instance()->CreateNotifyMsgBox(_T("dlg_notifymsg"));
// 按照非模式对话框创建主窗口,可以设置为默认隐藏
pMainDlg->Create(pMainDlg->GetIDTemplate(), NULL);
//pMainDlg->ShowWindow(SW_HIDE);
INT_PTR nResponse = pMainDlg->RunModalLoop();
// 如果是按照模式对话框运行主窗口,只要改为如下代码就可以
//INT_PTR nResponse = pMainDlg->DoModal();
// 释放DuiVision界面库的资源
DuiSystem::Release();
```

如果已经定义了主窗口的 XML 定义文件，添加上面的代码之后应该就可以创建出主窗口了。
这段代码做的事情主要是：
1、DuiVision 库的初始化，并指定资源文件的位置（不指定则使用默认的位置）
2、根据 dlg_main 定义加载主窗口界面
3、创建主窗口的事件处理对象，并注册给主窗口（注册之后主窗口的事件都会自动发送给此事件处理对象的 OnDuiMessage 处理函数进行处理）
4、显示主窗口，使用非模态方式运行主窗口的消息循环
5、主界面关闭之后进行 DuiVision 库的释放
注意运行之前要把图片、 xml 定义都放在对应的位置，默认图片都在 skins 目录，xml 定义文件都在 xml 目录。

DuiSystem 构造函数第三个参数（资源文件名）的说明，资源文件名可以用下面几种格式：
1、不指定（为空），则使用默认的资源文件名 xml\resource.xml，表示加载 exe 路径下的xml\resource.xml 文件，根据此文件加载其余的资源；
2、 指定 xml 文件，例如 xml\resource.xml，表示加载指定的资源 xml 文件，根据此 xml 文件加载其余的资源，如果指定了 xml 文件路径则使用绝对路径，如果没有指定 xml 路径，则查找 exe 路径和 exe 下面的 xml 路径看有没有此文件，有的话就加载 xml 文件；
3、指定后缀是.ui 的文件，后缀是.ui 的文件表示是 zip 压缩文件，则首先加载此 zip 压缩文件到内存中，然后再从 zip 压缩文件中解压出相对路径是 xml\resource.xml 的文件进行加载，后续其余文件的加载过程和上面两种情况是相同的，如果 zip 压缩文件已经加载了，则加载其余文件的时候，会先查找物理路径下有没有文件，有的话就优先加载，物理路径下没有找到文件，则查找内存中的 zip 压缩文件中有没有文件，有的话就解压到内存并加载，加载内存文件时候会对内存文件进行缓存，下一次再加载相同的文件时候直接用内存缓存，不需要再进行解压缩；
4、指定前缀是 res:的文件，表示从程序的资源中首先获取到资源 zip 压缩文件，后续的流程和.ui 方式的 zip 压缩文件是相同的，这种情况下，参数 res:后面跟的是程序中的资源 ID，例如 res:1116 表示从程序资源中获取资源 ID 是 1116 的资源进行解压缩。


## 事件处理类编写
除了界面的描述之外，最主要的工作就是业务逻辑的处理，为了将业务逻辑和界面展示能够更好的分离，DuiVision 中定义了事件处理基类，所有的业务逻辑都应该写在派生的事件处理类中，并把事件处理对象注册到相应的对话框或控件上，这样对应的子控件有事件需要处理的时候，就会自动调用注册的事件处理对象的相应函数。事件处理类只要在处理函数中根据控件的 ID 或名字决定该做什么事情，写相应的处理代码就可以。时间处理类中同时提供了一些函数方便根据 ID 或名字获取到对应的控件对象，并对控件进行操作，例如改变控件文字、获取控件的某个状态等。
事件处理基类是CDuiHandler，这个类的一些函数如下，提供了获取控件对象、设置控件的一些参数的函数，方便在事件处理类中对控件的操作：

```cpp 
void SetDuiObject(CDuiObject* pDuiObject);
CControlBase* GetControl(UINT uControlID);
CControlBase* GetControl(CString strControlName);
CDlgBase* GetControlDialog(UINT uControlID);
void SetVisible(CString strControlName, BOOL bIsVisible);
void SetDisable(CString strControlName, BOOL bIsDisable);
void SetTitle(CString strControlName, CString strTitle);
CString GetTitle(CString strControlName);
virtual void OnInit();
virtual LRESULT OnDuiMessage(UINT uID, CString strName, UINT Msg, WPARAM wParam, LPARAM lParam);
```

下面这段代码是在派生的事件处理类的 OnDuiMessage 函数中判断如果点击了某个按钮，就修改一个进度条进度的代码：

```cpp 
if(strName == _T("button_normal_3"))
{
    CDuiProgress* pControl = (CDuiProgress*)GetControl(_T("progress"));
    if(pControl && (Msg == BUTTOM_UP))
    {
        static int g_nProgress = 0;
        g_nProgress += 10;
        if(g_nProgress > 100)
        {
            g_nProgress = 0;
        }
        pControl->SetProgress(g_nProgress);
    }
}
```

为了简化事件处理类的编写，使事件处理的代码看起来更清晰一些，DuiHandler.h 中定义了一些事件处理函数的消息映射宏，如下表所示：

| 宏 | 参数 | 说明 |
| ---------- | ---------- | ---------- |
| DUI_DECLARE_MESSAGE_BEGIN | 类名 | 事件处理类的消息映射宏开始 |
| DUI_DECLARE_MESSAGE_END | 无 | 事件处理类的消息映射宏结束 |
| DUI_CONTROL_ID_MESSAGE | 控件 ID、处理函数 | 根据控件 ID，执行相应的处理函数 |
| DUI_CONTROL_IDMSG_MESSAGE | 控件 ID、消息、处理函数 | 根据控件 ID 和消息，执行相应的处理函数 |
| DUI_CONTROL_NAME_MESSAGE | 控件名、处理函数 | 根据控件名，执行相应的处理函数 |
| DUI_CONTROL_NAMEMSG_MESSAGE |控件名、消息、处理函数 | 根据控件名和消息，执行相应的处理函数 |

实际定义的样例如下：

```cpp 
// 消息处理定义
DUI_DECLARE_MESSAGE_BEGIN(CDuiHandlerMain)
DUI_CONTROL_ID_MESSAGE(APP_IPC, OnDuiMsgInterprocess)
DUI_CONTROL_NAME_MESSAGE(NAME_SKIN_WND, OnDuiMsgSkin)
DUI_CONTROL_NAMEMSG_MESSAGE(NAME_TRAY_ICON, MSG_TRAY_DBCLICK,
OnDuiMsgTrayIconDClick)
DUI_CONTROL_NAMEMSG_MESSAGE(L"notify_button_1", BUTTOM_UP, OnDuiMsgNotifyButton1)
DUI_CONTROL_NAMEMSG_MESSAGE(L"notify_button_2", BUTTOM_UP, OnDuiMsgNotifyButton2)
DUI_CONTROL_NAMEMSG_MESSAGE(L"notify_button_3", BUTTOM_UP, OnDuiMsgNotifyButton3)
DUI_CONTROL_NAMEMSG_MESSAGE(L"listctrl_1", BUTTOM_DOWN, OnDuiMsgListCtrl1Click)
DUI_CONTROL_NAMEMSG_MESSAGE(L"listctrl_2", BUTTOM_DOWN, OnDuiMsgListCtrl2Click)
DUI_DECLARE_MESSAGE_END()
```

在编写自己的 Handler 事件处理类时候可以参考样例，定义消息映射宏，每个需要处理的消息定义一行内容，定义出哪个控件的什么消息需要处理，由哪个函数处理，消息处理函数必须按照固定的参数格式来编写，消息处理函数的样例如下：

```cpp 
// 显示信息对话框消息处理
LRESULT CDuiHandlerMain::OnDuiMsgMsgBoxButton1(UINT uID, CString strName, UINT Msg, WPARAM wParam, LPARAM lParam)
{
    DuiSystem::DuiMessageBox(NULL, _T("演示对话框！"));
    return TRUE;
}
```

每个消息处理函数必须按照以上样例中的参数格式， 函数的返回值表示此消息是否不再需要向下传递继续处理了，如果返回 TRUE，则消息处理结束，如果返回 FALSE，则此消息还可以继续被后面定义的其他函数处理。

##控件的唯一标识和控件查找
DuiVision 库的一个特点就是可以通过 ID 和 name 两种方式灵活的进行控件的查找，每个控件对象创建的时候都会自动分配一个唯一 ID，同时也可以给控件命名，查找一个控件也可以通过 ID 和 name 两种方式进行查找，因为 ID 方式查找不够灵活，所以一般情况下都建议用 name 的方式进行查找，控件的 name 有两种方式可以设置，一种方式是在 xml 文件中定义控件的 name 属性，另一种方式是控件创建之后调用 SetName 函数进行设置。
控件查找可以通过 GetControl 函数进行查找，这个函数在事件处理基类、控件基类、对话框中都有定义。其中控件基类中 GetControl 函数的查找方式是递归查找当前控件的子控件，看是否有对应名字或 ID 的子控件；事件处理类中 GetControl 函数的查找方式是查找此事件处理对象关联的控件对象，然后查找此控件对象或子控件中是否有对应名字或 ID 的控件；对话框类中的 GetControl 函数的查找方式是递归查找当前对话框中的所有控件，看是否有对应名字或 ID 的控件。

## **系统预定义控件、动作和事件**
DuiVision 库中预定义了一些控件名、动作和事件，这些定义可以参考 duiid.h。
对于预定义的控件名，只要某个控件定义的名字是这个名字，就会被看做为特定的控件，系统会对其事件作出响应，预定义控件如下：

| 控件名 | 定义 | 说明 |
| ---------- |
| tray.icon | NAME_TRAY_ICON | 托盘图标，系统托盘图标事件中的控件名都是这个名字 |
| button.min | NAME_BT_MIN | 最小化按钮，按钮定义成这个名字点击会最小化窗口 |
| button.max | NAME_BT_MAX | 最大化按钮，按钮定义成这个名字点击会最大化/恢复窗口 |
| button.close | NAME_BT_CLOSE | 关闭按钮，按钮定义成这个名字点击会关闭窗口 |
| button.skin | NAME_BT_SKIN | 换肤按钮，按钮定义成这个名字点击会显示换肤窗口 |
| button.setup | NAME_BT_SETUP | 设置按钮，建议打开系统设置窗口的按钮控件用这个名字 |
| frame.mainwnd | NAME_FRAME_MAINWND | 主窗口的透明度渐变层蒙板图片，这个控件是每个窗口自动创建的 |
| button.ok | NAME_BT_OK | 确定按钮，用于对话框中的按钮，点击会指定对话框的DoOK函数 |
| button.cancel | NAME_BT_CANCEL | 取消按钮，用于对话框中的按钮，点击会指定对话框的DoCancel函数 |
| button.yes | NAME_BT_YES | 是按钮，用于对话框中的按钮，点击会指定对话框的DoYes函数 |
| button.no | NAME_BT_NO | 否按钮，用于对话框中的按钮，点击会指定对话框的DoNo函数 |
| skin.wnd | NAME_SKIN_WND | 皮肤选择窗口 |

系统预定义的动作如下，这些预定义动作用于定义在控件 xml 的 action 属性中：

| action名 | 定义 | 说明 |
| ---------- |
| close-window | ACTION_CLOSE_WINDOW | 关闭窗口，action 中以此开头，后面跟的是窗口名字，表示动作为关闭指定的窗口
| hide-window | ACTION_HIDE_WINDOW | 隐藏窗口，表示隐藏当前窗口 |
| show-window | ACTION_SHOW_WINDOW | 显示窗口，action 中以此开头，后面跟的是窗口名字，表示动作为显示指定的窗口 |

预定义的控件消息：

| 消息定义 | 说明 |
| ---------- |
| MSG_TRAY_DBCLICK | 托盘双击消息 |
| MSG_TRAY_LBUTTONDOWN | 托盘左键单击消息 |
| MSG_BUTTON_DOWN | 鼠标在控件按下 |
| MSG_BUTTON_UP | 鼠标在控件放开 |
| MSG_BUTTON_DBLCLK | 鼠标在控件双击 |
| MSG_BUTTON_CHECK | 检查框点击 |
| MSG_SCROLL_CHANGE | 滚动条位置变更事件 |
| MSG_CONTROL_BUTTON | 控件内的按钮点击事件， 例如 tabctrl 控件中可以在每个 tab页增加一个可点击的按钮，用于关闭 tab 等操作，这种控件内部按钮点击时候会发送此消息 |
| MSG_MOUSE_MOVE | 鼠标在控件范围内移动的事件 |
| MSG_MOUSE_LEAVE | 鼠标离开控件范围的事件（离开之后只会发送一次此消息） |
| MSG_KEY_DOWN | 控件的键盘事件处理，消息中的 wParam 表示键盘码，lParam 表示 Flasgs 状态 |

## 控件的快捷键和焦点的支持
每个控件可以设置快捷键，设置方法是在 xml 中设置 shortcut 属性，例如下面这个控件设置快捷键为 ESC 键：

```xml
<imgbtn name="button.close" pos="-45,0,-0,29" skin="IDB_BT_CLOSE" shortcut="ESC" />
```

快捷键的写法是 flag+char 的形式，flag 可以是 CTRL、 ALT、 SHIFT，分别表示几个控制键是否按下，char 是键的名字，可以用的包括字母、数字，以及下面这些键：
RETURN – 回车
ESC – 取消
BACK – 回退
TAB – TAB 键
SPACE – 空格键
PRIOR – 上翻页
NEXT – 下翻页
END – 到文件尾
HOME – 到文件头
LEFT、 UP、 RIGHT、 DOWN – 几个方向键
PRINT – 打印
INSERT – 插入
DELETE – 删除
F1-F12 – 功能键
如果没有 flag，也可以直接写键的名字。
某个控件如果设置了快捷键，当按下快捷键时候，相当于在控件上点击了一次鼠标，实际动作就是自动触发针对这个控件的一次鼠标按下消息和鼠标放开消息。
DuiVision 的控件支持焦点状态，如果一个控件要支持焦点的话，可以通过设置控件的 tabstop属性来实现，tabstop 为 1 表示此控件可以处于焦点状态，tabstop 属性也可以通过 API 查询，就是控件的 IsTabStop 函数。一些控件默认是会设置为 tabstop 为 1 的状态的，这些控件包括按钮、检查框、 RadioButton、编辑框。
焦点的切换有两种方式，一种是通过键盘操作，TAB 键和 SHIFT+TAB 键分别表示切换到下一个或上一个可以获取焦点的控件上面，另一种方式是鼠标点击了一个控件之后，这个控件就会成为当前的焦点控件。为了区分焦点控件，DuiVision 提供了默认的焦点控件显示方式，就是在控件的内部靠近控件边框位置画灰色的虚线框，有些控件不会采用这样的方式画焦点，例如编辑框处于焦点状态时候会显示编辑框的输入光标，而编辑框失去焦点时候会取消光标的显示，同时会在编辑框内显示出编辑框控件的 tip 信息。

## 控件的tip的支持
控件可以设置 tip 属性，也可以通过 API 函数 SetTooltip 来设置。
设置了 tip 之后，当鼠标移动到此控件并停留一段时间，就会显示此控件的 tip 信息.

## 动态创建界面控件

```cpp 
CDuiButton* pToolBtn = static_cast<CDuiButton*>(DuiSystem::CreateControlByName(L"button", m_pDlg->GetSafeHwnd(), m_pDuiObject));
if(pToolBtn)
{
    ((CControlBase*)m_pDuiObject)->AddControl(pToolBtn);
    pToolBtn->SetName(L"btnName");
    pToolBtn->SetTitle(L"动态按钮");
    pToolBtn->SetPosStr(L"10,10,110,35");
    pToolBtn->OnAttributeSkin(L"skin:IDB_BT_ICON", FALSE);
    pToolBtn->OnAttributeImageBtn(L"skin:IDB_ICON_TOOL", FALSE);
    pToolBtn->SetMaxIndex(4);
    pToolBtn->SetAction(L"dlg:dlg_login");
    pToolBtn->OnPositionChange();
}
```

创建之后再调用控件的函数来设置控件的一些属性，需要特别注意的是控件的位置信息的设置，必须使用位置字符串的方式来设置，因为位置字符串是可以根据父控件的位置信息计算出控件的实际位置的，如果不采用这种方式，直接指定控件的位置，则可能会因为位置自动计算时候没有相关的信息导致最后计算不出真正的位置。
位置字符串设置需要调用 SetPosStr 函数进行设置，注意最后一定要调用 OnPositionChange函数，这个函数会真正进行一次位置计算，这样才能使控件显示在正确的位置。

## 日志文件定义
DuiVision 提供了日志文件的操作函数，在任何地方都可以通过如下代码调用日志函数来写日志：

```cpp 
DuiSystem::LogEvent(LOG_LEVEL_DEBUG, _T("CDuiHandlerTab3::OnDuiMessage:uID=%d, name=%s, msg=%d, wParam=%d, lParam=%d"), uID, strName, Msg, wParam, lParam);
```

此函数的第一个参数是日志的级别，后面的参数类似于 C 语言的 printf 的写法。级别定义如下：

```cpp 
#define LOG_LEVEL_DEBUG 0x0001 //调试信息
#define LOG_LEVEL_INFO 0x0002 //一般信息
#define LOG_LEVEL_ERROR 0x0004 //错误信息
#define LOG_LEVEL_CRITICAL 0x0008 //致命信息
```

日志文件默认是在 exe 所在路径下面，日志文件名和日志级别可以在全局资源定义（resource.xml）中定义，定义如下：

```xml
<!--系统配置-->
<res type="cfg" name="logfile" value="demoui.log" />
<res type="cfg" name="loglevel" value="1" />
```

其中 logfile 表示日志文件的名字，是对应的 exe 文件的路径，loglevel 表示记录的日志的最低级别，日志级别有 4 种级别，分别是 1-调试级别、 2-一般级别、 4-错误级别、 8-致命级别。
如果这里的 loglevel 定义的是 1，就表示调试和以上级别的日志都会记录。

## 任务类
基于 MFC 的界面程序中，如果存在多线程，一般情况下只有主线程（界面线程）可以调用Windows 窗口相关的函数，否则如果在其他线程中调用了界面函数，很可能会造成异常。
为此 DuiVision 库提供了一个任务队列和相应的调度机制，可以将各种任务对象放到任务队列中按顺序执行，通过任务队列，可以做到其他线程和界面线程之间的中转调用，方法是创建任务对象时候指定是需要界面线程处理的任务，则任务调用过程中会通过跨进程的消息将此任务通知界面线程来执行。
DuiVision 中定义了任务的基类和几种派生类，基类是 IBaseTask 类，派生类目前有两种，分别是 DuiSystem 中定义的 CDuiActionTask 类（用于执行菜单等动作），另一种是 DuiSystem 中定义的 CDuiNotifyMsgTask 类，用于执行桌面右下角的弹出提示窗口，这两个任务类的核心执行代码都在 DoAction 函数中，如果需要派生其他的任务类，可以参考者两个类的实现方式。

## 界面皮肤定义
默认的 9 张背景图片是放在 exe 所在路径的 bkimg 子目录下，图片文件名分别是SKIN_PIC_0.png-SKIN_PIC_8.png，如果要更改默认的图片，只要在制作安装包时候替换这个目录中相应名字的图片就可以。
皮肤选择窗口是通过皮肤按钮来打开的，只要窗口的某个控件定义的名字是皮肤按钮的名字，就自动具有皮肤按钮的功能，皮肤按钮的名字是 button.skin，例如下面这段对话框 xml中的定义，就自动会支持皮肤窗口的功能：

```xml
<imgbtn name="button.skin" pos="-140,0,-111,29" skin="IDB_BT_SKIN" tip="皮肤" show="1"/>
```

通过皮肤窗口更改了背景皮肤之后，这个改动并不会保存下来，如果下次运行这个程序，就会又回到最初的状态（默认显示的是第一张背景图片），为了将选择的皮肤保存下来，需要在主程序中写一些代码来实现，例如将选择的皮肤信息保存在注册表中，具体实现方式可以参考 DuiVision 界面库 demo 程序中的 DuiHandler_main.cpp 的CDuiHandlerTest::ProcessSkinMsg 函数的实现。

## 托盘图标
DuiVision 界面库封装了 Windows 托盘图标的相关操作，可以创建托盘图标，并设置图标文件、托盘的 tip 信息，也可以处理托盘的单击、双击、右键菜单的事件。
通过调用下面的函数可以进行托盘的初始化：

```cpp 
DuiSystem::Instance()->InitTray();
```

初始化一般放在主的事件处理类 OnInit 函数中，可以参考 demo 程序的代码。 设置托盘的图标文件盒 tip 信息可以调用 DuiSystem 的 SetTrayIcon、 SetTrayTip 函数。
托盘的右键操作是打开右键菜单，右键菜单在 resource.xml 中通过 menu_tray 名字的资源项定义具体的菜单 xml 文件。
托盘的左键双击默认动作时打开主窗口，也可以更改为自定义的处理方式，resource.xml 中下面的配置项用于定义托盘双击的动作，如果为 0 就表示执行默认的打开主窗口的动作，如果为 1，则会发送 MSG_TRAY_DBCLICK 消息，通过在事件处理类中响应这个消息，就可以处理双击事件。

```cpp 
<res type="cfg" name="trayDbClickMsg" value="0" ></res>
```

托盘左键的单击事件也会发送一个消息，消息 ID 为 MSG_TRAY_LBUTTONDOWN，通过在事件处理类中响应这个消息，就可以处理单击事件。 可以参考 Demo 程序单击和双击事件响应函数。

## 界面插件介绍
DuiVision 界面库支持界面插件，每个界面插件是一个独立的动态库，里面可以封装各种界面，界面插件会对外暴露标准的插件接口，主应用程序可以加载插件，将插件中界面集成到主程序指定的位置，插件中可以有自己的事件处理。通过插件可以将一个大的界面应用程序分离成多个小程序，DuiVision 界面插件的一个优点是基于接口进行集成，所以主程序和界面插件动态库可以使用不同版本的 DuiVision 库进行编译，减少了主程序和插件的版本依赖性，甚至基于其他界面库开发的界面，只要是符合界面插件规范也可以进行集成。
DuiVision 界面插件目前只支持通过 div 控件进行集成，因为 div 控件是一个容器控件，通过设置 div 控件的 plugin 和 plugin-debug 属性，指定插件的动态库文件，主程序在初始化这个div 时候就会自动加载插件动态库，并将插件中的顶层 div 和主程序中这个 div 关联起来，插件显示的区域就是主程序这个 div 指定的区域。


# 控件说明
## DUI 基类
类名：CDuiObject
控件名：无（基类，没有实体控件）
说明：所有 DUI 的基类（包括控件、对话框、菜单等）

| 函数 | 是否虚函数 | 说明 |
| ---------- | ---------- |
| IsClass | 是 | 判断是否此类 |
| GetObjectClass | 是 | 获取类名 |
| BaseObjectClassName | 是 | 获取基类名 |
| GetID | 否 | 获取对象 ID |
| GetName | 否 | 获取对象名字 |
| IsThisObject | 否 | 根据 ID 或名字判断是否此对象，ID 或名字任意一个匹配上就可以 |
| RegisterHandler | 否 | 注册事件处理对象 |
| GetDuiHandler | 否 | 获取事件处理对象指针 |
| GetRect | 否 | 获取控件的左上角、右下角坐标 |
| ParseDuiString | 否 | 解析字符串，替换其中的替换内容（用[]包围的定义内容），替换内容是在 resource.xml 或字符串定义文件中定义的字符串内容 |
| ParseKeyCode | 否 | 根据传入的字符串获取对应的键盘码，例如CTRL+F1 会被转换为 0x11,0x70 |

## DUI 控件基础类
类名：CControlBase
控件名：无（基类，没有实体控件）
说明：所有界面控件的基类

**属性**：

属性名 | 类型 | 说明
-------- | ---------- | -----------------------------------------------
show | 1,0 | 控件是否可见
disable | 1,0 | 控件是否被禁用
pos | 位置 | 控件的位置坐标，可以是左上角坐标，例如 10,10，也可以是左上角+右下角坐标，例如10,10,200,100。支持正值和负值，正值表示从父控件左上角开始计算的值，负值表示从父控件右下角开始计算的值，例如-10,10 表示从右边往左 10 像素，从上往下 10像素的位置。也可以支持从父控件中间开始计算的坐标，用&iota;表示中间位置，例如&iota;10,&iota;-10表示横向中间向右10像素，纵向中间向上 10 像素的位置。
width | 数字 | 控件宽度
height | 数字 | 控件高度
action | 字符串 | 控件的动作字符串，表示控件点击之后执行的动作，有几种类型：1、 dlg:xxxx，表示显示指定的对话框，显示的对话框可以是一个 xml 定义文件，也可以是一个定义（从resource 资源定义文件中查找具体的 xml 定义文件）2、 menu:xxxx.xml，表示显示指定的菜单，菜单定义文件是指定的 xml 文件3、 link:url，表示使用浏览器或其他默认程序打开指定的链接或文件4、run:xxx.exe&iota;param，表示运行指定的程序，可以传递命令行，执行的程序后面用&iota;分隔，&iota;之后的表示传递的命令行参数
menupos | 位置 | 菜单显示的位置坐标，必须是 x,y 的形式，表示菜单左上角坐标，例如 10,10。支持正值和负值，正值表示从控件左上角开始计算的值，负值表示从父控件右下角开始计算的值，例如-10,10表示从右边往左 10 像素，从上往下 10像素的位置。也可以支持从控件中间开始计算的坐标，用&iota;表示中间位置，例如&iota;10,&iota;-10 表示横向中间向右 10 像素，纵向中间向上 10 像素的位置。tip 字符串 定义鼠标移动到空间上过一段时间会出现的 tip 提示信息，tip 信息只有对话框的基础控件可以定义，其他控件即使定义了也没有效果
tip-width | 数字 | Tooltip 的宽度，默认为 0，这个值只有设置为大于0 才会有效
response | 1,0 | 控件是否可以响应鼠标事件，如果不希望控件响应鼠标事件，可以设置此属性为 0
tabstop | 1,0 | 控件是否可以响应焦点切换的事件，也就是按键盘tab 键能否将焦点切换到此控件
taskmsg | 1,0 | 调用事件响应函数时候是否采用任务方式，对于可能产生阻塞或耗时比较长的处理操作应该采用任务方式处理，避免造成界面不响应或界面异常
img-ecm | 1,0 | 是否使用图片自身的颜色管理信息，默认为 0，表示加载图片文件时候使用系统的颜色管理信息而不使用图片自身的颜色管理信息，因为 XP SP1 以前的操作系统自带的 GDI+模块可能不支持图片自身的颜色管理信息，因此如果设置为 1 的话，在 XPSP1 以前的系统下运行图片可能无法正常显示
shortcut | 字符串 | 定义控件的快捷键，快捷键字符串参考第二章的相应章节说明，当按下对应快捷键的时候，会自动触发此控件产生一个鼠标按下和鼠标放开的事件，模拟点击了此控件cursor 字符串定义控件的鼠标光标，如果定义，则鼠标移动到控件范围内会显示指定的鼠标光标，目前支持系统预定义的几个鼠标光标，分别是：arrow - 箭头；wait – 沙漏等待；cross – 十字；sizewe –双箭头指向东西；sizens – 双箭头指向南北；hand – 手型；help – 箭头+问号
duimsg | 字符串 | 指定控件会发送哪些 DUI 消息，消息名之间用&iota;分隔，例如：” msg1&iota;msg2&iota;msg3” 。目前支持的消息名包括：mousemove – 发送鼠标移动和鼠标离开控件的消息。keydown – 发送键盘按下的消息。

**函数**：

函数 | 是否虚函数 | 说明
---- | ----
GetParent | 否 | 获取父控件对象指针
GetControl | 否 | 根据 ID 或 name 获取控件对象指针
AddControl | 否 | 添加控件
RemoveControl | 否 | 删除控件
GetControls | 否 | 获取所有控件对象的列表
GetParentDialog | 否 | 获取控件所在的对话框的指针
OnMessage | 是 | 控件的消息处理函数
SendMessage | 否 | 发送通知消息
SetRect | 是 | 设置控件的位置
GetRect | 是 | 获取控件的位置
OnPositionChange | 是 | 刷新控件的位置信息
SetPosStr | 否 | 设置控件的位置字符串
SetVisible | 否 | 设置控件的可见性
GetVisible | 否 | 获取控件的可见性
SetControlWndVisible | 是 | 设置控件中的Windows原生控件是否可见的状态，由 Panel 对象中调用，对于 edit、activex等使用了Windows原生控件的类需要重载此函数，并正确的进行原生控件的显示和隐藏
SetDisable | 否 | 设置控件是否禁用
GetDisable | 否 | 获取控件是否禁用
SetTabStop | 否 | 设置控件是否能响应 tab 键
IsTabStop | 否 | 获取控件是否能响应 tab 键
SetTooltip | 否 | 设置控件的 tip 信息
GetTooltip | 否 | 获取控件的 tip 信息
SetAction | 否 | 设置控件的动作字符串
GetAction | 否 | 获取控件的动作字符串
SetRresponse | 否 | 获取控件是否可响应鼠标事件
GetRresponse | 否 | 设置控件是否可响应鼠标事件
PtInRect | 是 | 判断坐标是否在控件范围内
UpdateControl | 否 | 刷新控件显示
SetWindowFocus | 是 | 设置窗口焦点

# DUI 文字控件基础类
类名：CControlBaseFont
控件名：无（基类，没有实体控件）
说明：所有界面控件中有显示文字信息的控件类的基类
**属性**：

属性名 | 类型 | 说明
---- | ----
title | 字符串 | 控件的显示标题
font | 字体 | 控件的字体，可以引用资源定义中定义的某个字体，默认字体是default
fontname | 字符串 | 直接指定某种字体
fontwidth | 数字 | 直接指定字体宽度
height | 数字 | 控件高度
valign | 枚举 | 文字的垂直对齐模式，top、middle、bottom
align | 枚举 | 文字的水平对齐模式，left、center、right
skin | 皮肤 | 控件的皮肤名，引用资源定义中的统一皮肤定义
image | 图片 | 控件的图片，有3种定义方式：1、图片文件：xxx.png，xxx.jpg 等，是相对exe的路径。2、图片资源：如果 image 不是文件格式，则认为是资源 ID，到程序的内嵌资源中去查找对应的图片资源。3、皮肤方式：skin:xxxx，如果是 skin:开始，则认为是皮肤格式，后面是皮肤名，到全局皮肤定义中查找具体图片img-count数字 定义图片的切片个数，如果一个图片文件中横向包含了多个等宽的小图片，根据这个定义，控件可以知道到底有几个小图片，并按照图片个数进行正确的切片

**函数**：

函数 | 是否虚函数 | 说明
---- | ----
SetTitle | 否 | 设置标题文字
GetTitle | 是 | 获取标题文字
SetAlignment | 是 | 设置控件的水平对齐方式
SetVAlignment | 是 | 设置控件的垂直对齐方式
SetImage | 否 | 设置控件的图片
SetBitmapCount | 否 | 设置控件图片的水平方向小图片个数

## DUI 矩形区域控件
类名：CArea
控件名：area
说明：可以设置区域的渐变透明度，不能响应鼠标事件，此控件的原理是画一个填充的透明度渐变矩形区域，透明度从矩形区域顶部到底部均匀渐变。
属性：

属性名 | 类型 | 说明
---- | ----
color | 颜色 | 矩形区域的颜色
begin-transparent | 字体 | 矩形区域顶部的起始透明度
end-transparent | 字符串 | 矩形区域底部的终止透明度

## DUI 矩形边框控件
类名：CDuiFrame
控件名：frame
说明：可以画矩形边框，可以指定边框颜色、宽度、类型，边框内部可以采用渐变填充，可以之内内部区域的颜色和渐变透明度。

**属性**：

属性名 | 类型 | 说明
---- | ----
color | 颜色 | 矩形区域的颜色
crframe | 颜色 | 边框颜色
dashstyle | 数字 | 边框类型
Frame-width | 数字 | 边框宽度
begin-transparent | 字体 | 矩形区域顶部的起始透明度
end-transparent | 字符串 | 矩形区域底部的终止透明度

## DUI 对话框
类名：CDlgBase
控件名：无
说明：对话框类，所有对话框都直接创建一个 CDlgBase 对象就可以。代码中如果需要创建一个对话框，一般建议使用 DuiSystem 类中封装的若干对话框相关的函数来操作，包括创建对话框、删除对话框、根据对话框名获取对话框指针、显示通用对话框。

**属性**：

属性名 | 类型 | 说明
---- | ----
width | 数字 | 窗口宽度
height | 数字 | 窗口高度
resize | 0,1 | 1表示窗口可以改变大小
frame | 字符串 | 窗口的frame层图片，frame层是一个可选的半透明Alpha图片层，一般设置的这个图片是用于和背景图片进行Alpha混合，这一层的图片中每个像素都包含了自身颜色和透明度属性，通过透明度属性可以将背景图片进行半透明处理，默认只有主窗口设置了这个frame层图片，并且默认的frame图片是一个透明度渐变的PNG图片，从顶端的100%透明到底端的完全不透明
framesize | 数字 | 窗口的frame层图片的边框宽度，非九宫格方式有效
width-lt | 数字 | 窗口的frame层图片的九宫格左上角位置距离边框的宽度
height-lt | 数字 | 窗口的frame层图片的九宫格左上角位置距离边框的高度
width-rb | 数字 | 窗口的frame层图片的九宫格右下角位置距离边框的宽度
height-rb | 数字 | 窗口的frame层图片的九宫格右下角位置距离边框的高度
bkimg | 字符串 | 窗口的背景图片，如果指定了就使用指定的背景图片，否则使用全局设置的背景图片crbk颜色窗口的背景颜色，如果未指定背景图片，但指定了背景颜色，就使用指定的背景颜色，否则使用全局设置的背景图片
appwin | 0,1 | 此窗口是否会显示在 Windows任务栏中显示，见下面的截图说明
translucent | 数字 | 窗口的整体透明度，取值范围是1-255,1表示全透明，255表示不透明

说明：
1） 九宫格方式 frame 层的说明：对于复杂的背景 frame 层图片，其所有边框宽度并不是固定的，但一般都可以用九宫格方式来切分，就是把背景 frame 图片横向、纵向各用两条线切分，一共切分成九部分，应用时候四个角的图片大小是按照原始大小应用到窗口中的，其余几部分都会进行拉伸，对于这种方式，只要描述出九宫格的左上角和右下角坐标位置就可以，对应的就是 width-lt、 height-lt、 width-rb、 height-rb 这 4 个属性。

<img src="{{site.url}}/assets/4b3e84a1-52cd-44e9-90bd-03f6d4fea8e0.png" />

**函数**：

函数 | 是否虚函数 | 说明
---- | ----
SetXmlFile | 否 | 设置对话框加载的 | xml | 文件
GetControl | 否 | 根据ID或name获取对应的控件指针
DoOK | 否 | 对话框的确定
DoCancel | 否 | 对话框的取消
DoClose | 否 | 对话框的关闭
SetControlVisible | 是 | 设置指定控件的可见性
SetControlDisable | 是 | 设置指定控件是否禁用
OpenDlgPopup | 否 | 打开一个弹出框
CloseDlgPopup | 否 | 关闭弹出框

## DUI 菜单
类名：CDuiMenu
控件名：无
说明：菜单有两种显示的位置，一种是在窗口顶部某个按钮点击后可以下拉一个菜单，另一种是托盘图标的右键菜单。
窗口中的菜单定义方式是 xml 文件中设置某个按钮的 action 属性，以 menu:开头，后面是菜单的 XML 文件名或 XML 定义名，例如下面这样定义：

```xml
<imgbtn name="button.menu" pos="-110,0,-77,29" skin="IDB_BT_MENU" tip="菜单" action="menu:mainmenu.xml"/>
```

属性名 | 类型 | 说明
---- | ----
width | 数字 | 菜单窗口宽度
item-height | 数字 | 每个菜单项的高度
left | 数字 | 菜单左侧图标区的宽度
sep-height | 数字 | 菜单分隔线的高度
font | 字符串 | 字体
fontwidth | 数字 | 字体宽度
frame-width | 数字 | 菜单项距离边框的宽度
top-height | 数字 | 菜单项顶部距离边框的高度
bottom-height | 数字 | 菜单项底部距离边框的高度
crrowhover | 颜色 | 菜单项背景颜色(鼠标移动到菜单项时候的颜色)，如果不设置则使用默认颜色
img-rowhover | 图片 | 菜单项背景图片(鼠标移动到菜单项时候的背景图片)，优先级比背景颜色高
img-popuparrow | 图片 | 弹出菜单箭头图片

函数 | 是否虚函数 | 说明
---- | ----
LoadXmlFile | 否 | 加载菜单 | XML | 文件
AddMenu | 否 | 动态添加菜单项
AddSeparator | 否 | 动态添加菜单分隔线
SetItemTitle | 否 | 预设值菜单项的标题
SetItemVisible | 否 | 预设值菜单项的可见性
SetItemDisable | 否 | 预设值菜单项的禁用状态
SetItemCheck | 否 | 预设值菜单项的检查标志
SetMenuPoint | 否 | 刷新所有菜单项的位置信息
GetParentMenu | 否 | 获取父菜单对象
GetHoverMenuItem | 否 | 获取当前激活菜单项对象

## DUI菜单项
类名：CMenuItem
控件名：menuitem
说明：菜单中加载的每个菜单项的控件，CControlBaseFont中的所有属性和函数都可以使用。

属性名 | 类型 | 说明
---- | ----
seperator | 0,1 | 是否分隔线
select | 0,1 | 是否选择（如果是checkbox或radiobutton类型的菜单项，此属性必须设置为1）
check | 0,1 | 是否处于选中状态group字符串广播按钮所属的组名，相同组名的广播按钮是属于一组的，可以联动，一组中只有一个会处于选中状态
value | 字符串 | 广播按钮的值，一组广播按钮中的多个按钮值是不一样的，当获取这一组广播按钮的值时候，获取的就是选中的按钮的值
menu | 字符串 | 引用其他的菜单的名字（通过资源定义可以找到的菜单的名字），设置了这个属性，则会将对应的菜单嵌入当前菜单中
image | 图片 | 菜单项左侧的小图片，如果是弹出菜单，并且没有设置菜单的img-popuparrow属性，则image属性表示菜单项右侧的箭头图片
img-count | 数字 | 设置菜单项左侧图片是由几个并列的小图片组成的
taskmsg | 0,1 | 是否通过任务方式执行菜单处理函数，如果弹出菜单的处理函数中有阻塞或等待的操作（例如打开一个对话框），则执行过程中弹出菜单可能会因为失去焦点而将自身的对象删除，这种情况下就需要通过任务方式执行菜单处理函数，任务方式是将操作插入任务队列，由任务队列线程再去执行菜单处理函数

函数 | 是否虚函数 | 说明
---- | ----
SetCheck | 否 | 设置是否选择
GetCheck | 否 | 获取是否选择的状态
IsSeparator | 否 | 判断是否分隔线
SetGroupName | 否 | 设置广播按钮组的名字
GetGroupName | 否 | 获取广播按钮组的名字
GetValue | 否 | 获取广播按钮的值
GetGroupValue | 否 | 获取广播按钮组的值
ResetGroupCheck | 否 | 刷新父控件下面所有同一个组的RadioButton控件的状态

## DUI TabCtrl控件
类名：CDuiTanCtrl
控件名：tabctrl
说明：多个Tab页的切换控件。

属性名 | 类型 | 说明
---- | ----
image | 字符串 | Tab页的图标图片，可以试并排多个图片，通过img-count可以设置图片个数
img-sep | 字符串 | Tab页之间的分割线图片
img-hover | 字符串 | 鼠标移动到tab页时候的tab状态图片，并排的两张图片，分别是鼠标移动和鼠标点击状态的图片
img-tabbtn | 字符串 | tab页签内部按钮图片，并排的四张图片，分别是正常状态、鼠标移动、鼠标点击、禁用状态的图片
tabbtnpos | 位置 | Tab页签内部按钮图片显示的位置坐标，必须是x,y的形式，表示按钮图片左上角坐标（相对于每一个tab页签），例如10,10.支持正值和负值，正值表示从tab页签左上角开始计算的值，负值表示从tab页签右下角开始计算的值，例如-10,10表示从右边往左10像素，从上往下10像素的位置。也可以支持从tab页签中间开始计算的坐标，用&iota;表示中间位置，例如&iota;10,&iota;-10表示横向中间向右10像素，纵向中间向上10像素的位置。
item-width | 数字 | 每个tab页的宽度，如果没有设置则按照
img-hover | 数字 | 的图片宽度
tab-height | 数字 | Tab页图标部分高度，如果没有设置则按照
img-hover | 的图片高度
animate | 0,1 | Tab页切换时候是否显示动画效果
animate-count | 数字 | Tab页切换动画的帧数，默认是10帧
crtext | 颜色 | Tab页标题的文字颜色
tab-left-pading | 数字 | Tab标签最左侧的空白宽度


函数 | 是否虚函数 | 说明
---- | ----
InsertItem | 否 | 添加一个tab页
GetItemIndex | 否 | 根据tab页名字获取索引
GetItemInfo | 否 | 根据tab页索引获取tab页信息
RefreshItems | 否 | 刷新所有tab页，重新计算位置信息
DeleteItem | 否 | 根据索引或名字删除tab页
SetSelectItem | 否 | 设置当前激活的tab页
SetItemVisible | 否 | 设置tab页是否可见
LoadTabXml | 否 | 从XML文件加载tab页

每个 Tab 页签中可以放一个图片按钮，例如实现用于关闭 Tab 页签的按钮，通过 img-tabbtn属性可以设置这个图片按钮的图片，每个 tab 页签都是使用相同的图片，通过 tabbtnpos 属性可以设置按钮图片相对于每个 Tab 页签的位置，如果不设置位置，默认会显示在 Tab 页签的右上角，使用页签按钮实现的带关闭按钮的 Tab 页效果如下：

<img src="{{site.url}}/assets/ca72e0ea-8b4f-46c4-9d68-d4b487a1f2b6.jpg" />

页签按钮点击之后会发送一个消息到事件处理类，消息类型是 MSG_CONTROL_BUTTON，消息中的 wParam 参数表示 tab 页签的索引。

## DUI Tab 控件
类名：CDuiTanCtrl
控件名：tab
说明：Tab 页控件的下层控件，每个 tab 表示一个页面。

属性名 | 类型 | 说明
---- | ----
name | 字符串 | 可以根据名字获取到tab页对应的DuiPanle控件对象
title | 字符串 | Tab页标题
active | true,false | 此Tab页是否激活
show | 0,1或true,false | 此Tab页是否显示
visible | 0,1或true,false | 和show属性相同
image | 字符串 | 此tab页的页签图片，是一个3张横向切片图组成图片，3张图分别表示正常、鼠标移动、鼠标点击3种状态下的tab页签
img-index | 数字 | 如果是多张图片组成的大图，则表示图片的索引
img-count | 数字 | 如果是多张图片组成的大图，则表示图片的个数
div | 字符串 | 如果一个tab页的内容想写在一个单独的xml文件中，可以在此处定义xml文件名
pos | 字符串 | Tab页的位置信息字符串
outlink | 0,1或true,false | 表示此tab页是否是外部链接，外部链接是指点击之后打开一个网页或独立的窗口或程序，对于外部链接，是不会显示鼠标点击到此tab页的状态的
action | 字符串 | 点击此 | tab | 页时候执行的动作
scroll | 0,1 | 此tab页是否支持滚动

## DUI Panel 控件
类名：CDuiPanel
控件名：div
说明：Panel 控件是一种虚拟的容器控件，本身不会有任何显示界面，但可以在下层包含若干其他的控件，通过设置 Panel 的可见性可以控制下层所有控件的可见性。 Panel 类是从CControlBaseFont 派生的，所以基类控件具有的属性和函数也可以使用。

属性名 | 类型 | 说明
---- | ----
xml | 字符串 | Panel控件可以通过加载一个xml文件来加载下层控件，此处是指xml文件名
img-scroll | 字符串 | 滚动条图片
scroll-width | 数字 | 滚动条宽度
scroll | 0,1 | 是否允许滚动
plugin | 0,1 | DuiVision界面插件的release版本动态库文件，具体插件说明请参考插件章节，主程序是release版本编译时候此属性才有效
Plugin-debug | 0,1 | DuiVision界面插件的debug版本动态库文件，具体插件说明请参考插件章节，主程序是debug版本编译时候此属性才有效

函数 | 是否虚函数 | 说明
---- | ----
LoadXmlFile | 否 | 加载XML文件
SetVirtualHeight | 否 | 设置div的虚拟高度
CalcVirtualHeight | 否 | 自动计算div的虚拟高度，按照div中所有子控件的位置进行计算，找到位置最大值作为div的虚拟高度
SetEnableScroll | 否 | 设置div是否允许滚动
GetEnableScroll | 否 | 获取div是否允许滚动

## DUI 文字控件
类名：CDuiText
控件名：text
说明：显示指定的文字，支持文字中指定的字符串设置不同的颜色，是从 CControlBaseFont派生的，所以基类控件具有的属性和函数也可以使用。

属性名 | 类型 | 说明'
---- | ----
crtext | 颜色 | 文字颜色
crhover | 颜色 | 鼠标移动时候的颜色
crshadow | 颜色 | 文字阴影颜色
crback | 颜色 | 背景颜色
crmark | 颜色 | 特殊显示的文字颜色
mask | 字符串 | 特殊显示的字符串
img-scroll | 字符串 | 滚动条图片
scroll-width | 数字 | 滚动条宽度
bk-transparent | 数字 | 背景透明度，未指定背景颜色时才有效，0表示全透明，100表示不透明，默认为0

函数 | 是否虚函数 | 说明
---- | ----
SetTitleMarkText | 否 | 设置整个字符串和特殊显示部分字符串

## DUI 图片控件
类名：CDuiPicture
控件名：img
说明：显示指定的图片，是从 CControlBaseFont 派生的，所以基类控件具有的属性和函数也可以使用。

属性名 | 类型 | 说明
---- | ----
mode | normal,tile,extrude,frame,mid | 图片的现实模式，枚举类型，分别表示正常显示图片、平铺、拉伸、显示图片边框、九宫格方式显示图片
framesize | 数字 | 显示图片边框的模式下，表示边框的宽度
width-lt | 数字 | 九宫格模式下的九宫格左上角位置距离边框的宽度
height-lt | 数字 | 九宫格模式下的九宫格左上角位置距离边框的高度
width-rb | 数字 | 九宫格模式下的九宫格右下角位置距离边框的宽度
height-rb | 数字 | 九宫格模式下的九宫格右下角位置距离边框的高度


函数 | 是否虚函数 | 说明
---- | ----
SetShowMode | 否 | 设置图片显示模式
SetShowModeMID | 否 | 设置九宫格方式的图片显示模式

## DUI 动画图片控件
类名：CDuiAnimateImage
控件名：animateimg
说明：显示动画图片，是从 CControlBaseFont 派生的，所以基类控件具有的属性和函数也可以使用。此控件设置的图片按照横向若干张小图片拼接的方式，小图片的宽度必须相同，控件可以设置小图片的个数，控件内可以自动启动一个定时器，按照顺序循环定时显示小图片，达到动画的效果。

属性名 | 类型 | 说明
---- | ----
index | 数字 | 当前显示第几张图片
maxindex | 数字 | 小图片的个数
timer-count | 数字 | 图片切换的定时器周期数，每个周期的时间是30毫秒，也就是图片切换的时间是多少个30毫秒
run | 0,1 | 表示是否启动动画，启动之后动画图片可以自己按照设置的定时周期进行变化

函数 | 是否虚函数 | 说明
---- | ----
SetRun | 否 | 设置是否启动动画
SetTimerCount | 否 | 设置动画的图片切换周期

## DUI 按钮控件
类名：CDuiButton、 CImageButton
控件名：button、 imgbtn
说明：显示按钮，是从 CControlBaseFont 派生的，所以基类控件具有的属性和函数也可以使用。按钮的背景图片是由 4 张切图拼接成的大图片，分别是普通状态、鼠标移动状态、鼠标按下状态、禁用状态的图片，背景图片设置可以通过 skin 或 image 属性设置。
按钮控件有两种控件名，button 和 imgbtn，唯一的差别是 imgbtn 默认会启用渐变效果定时器，并且渐变效果的帧数默认设置为 10 帧。

属性名 | 类型 | 说明
---- | ----
crtext | 颜色 | 按钮文字的颜色
animate | 0,1 | 是否启用渐变效果定时器
maxindex | 0,1 | 渐变效果的最大帧数
img-btn | 字符串 | 图片按钮的图片文件
showfocus | 0,1 | 控件处于焦点状态时是否显示焦点虚线框（默认是显示）

函数 | 是否虚函数 | 说明
---- | ----
SetMaxIndex | 否 | 设置渐变效果的最大帧数

## DUI DUI 文字按钮控件
类名：CTextButton
控件名：textbtn
说明： 文字按钮是从 CControlBaseFont 派生的，所以基类控件具有的属性和函数也可以使用。
此控件显示效果是一段可点击的文字，文字可以设置普通状态、鼠标移动状态、鼠标按下状态、禁用状态的颜色。

属性名 | 类型 | 说明
---- | ----
crtext | 颜色 | 按钮文字的颜色
crhover | 颜色 | 鼠标移动时候的颜色
crpush | 颜色 | 鼠标点击时候的颜色
crdisable | 颜色 | 禁用状态的颜色

## DUI 链接按钮控件
类名：CLinkButton
控件名：linkbtn
说明： 链接按钮是从 CControlBaseFont 派生的，所以基类控件具有的属性和函数也可以使用。
此控件显示效果是一段可点击的文字，文字可以设置普通状态、鼠标移动状态、鼠标按下状态、禁用状态的颜色，文字点击之后可以打开一个网页或文件。

属性名 | 类型 | 说明
---- | ----
crtext | 颜色 | 按钮文字的颜色
crhover | 颜色 | 鼠标移动时候的颜色
crpush | 颜色 | 鼠标点击时候的颜色
crdisable | 颜色 | 禁用状态的颜色
href | 字符串 | 链接URL

函数 | 是否虚函数 | 说明
---- | ----
SetLink | 否 | 设置链接URL
GetLink | 否 | 获取链接URL

##DUI 隐藏按钮控件
类名：CHideButton
控件名：hidebtn
说明： 文字按钮是从 CControlBaseFont 派生的，所以基类控件具有的属性和函数也可以使用。
此控件显示效果是一段文字右边有一段默认隐藏的文字链接，正常情况下看不到右边的文字链接，鼠标移动到左边的文字时候才能看到右边的文字链接。文字可以设置普通状态、鼠标移动状态、鼠标按下状态、禁用状态的颜色。

属性名 | 类型 | 说明
---- | ----
crtext | 颜色 | 按钮文字的颜色
crhover | 颜色 | 鼠标移动时候的颜色
crpush | 颜色 | 鼠标点击时候的颜色
crdisable | 颜色 | 禁用状态的颜色
crtip | 颜色 | 隐藏链接的文字颜色
text | 字符串 | 隐藏链接的字符串显示内容

## DUI 检查框控件
类名：CCheckButton
控件名：chkbtn
说明：显示检查框，是从 CControlBaseFont 派生的，所以基类控件具有的属性和函数也可以使用。

属性名 类型 说明
---- | ----
check true,false 检查框是否处于选择状态
crtext 颜色 文字颜色
showfocus 0,1 控件处于焦点状态时是否显示焦点虚线框（默认是显示）

函数 是否虚函数 说明
---- | ----
SetCheck 否 设置检查框的值
GetCheck 否 获取检查框的值
SetTextColor 否 设置文字颜色

## DUI 广播按钮控件
类名：CDuiRadioButton
控件名：radiobtn
说明：显示广播按钮，是从 CControlBaseFont 派生的，所以基类控件具有的属性和函数也可以使用。

属性名 | 类型 | 说明
---- | ----
check | true,false | 广播按钮是否处于选中状态
crtext | 颜色 | 文字颜色
group | 字符串 | 广播按钮所属的组名，相同组名的广播按钮是属于一组的，可以联动，一组中只有一个会处于选中状态
value | 字符串 | 广播按钮的值，一组广播按钮中的多个按钮值是不一样的，当获取这一组广播按钮的值时候，获取的就是选中的按钮的值
showfocus | 0,1 | 控件处于焦点状态时是否显示焦点虚线框（默认是显示）

函数 | 是否虚函数 | 说明
---- | ----
SetCheck | 否 | 设置检查框的值
GetCheck | 否 | 获取检查框的值
SetGroupName | 否 | 设置广播按钮组的名字
GetGroupName | 否 | 获取广播按钮组的名字
GetValue | 否 | 获取广播按钮的值
GetGroupValue | 否 | 获取广播按钮组的值
ResetGroupCheck | 否 | 刷新父控件下面所有同一个组的RadioButton控件的状态
SetTextColor | 否 | 设置文字颜色

## DUI 进度条控件
类名：CDuiProgress
控件名：progress
说明：显示进度条，是从 CControlBaseFont 派生的，所以基类控件具有的属性和函数也可以使用。

属性名 | 类型 | 说明
---- | ----
value | 数字 | 进度值，最大值不能超过max-value的值
max-value | 数字 | 最大的进度值，默认为100
timer-count | 数字 | 定时器定时多少次，自动增长的进度值加1
run | true,false | 进度条是否自动运行（循环增长）
img-back | 字符串 | 背景图片
img-fore | 字符串 | 前景图片
head-len | 数字 | 左右两侧圆角部分长度，如果此长度不为0，表示头部是圆角的进度条，画图时候头部圆角按照实际图片画图，其余部分则拉伸显示
show-text | 0,1 | 是否显示进度条的文字，如果显示文字，可以通过align和valign属性设置文字的对齐方式，显示的文字内容是前缀+当前进度值，如果进度最大值是100，则自动在当前进度值后面加%
crtext | 颜色 | 进度条文字的颜色
title | 字符串 | 进度条文字的前缀，例如title设置为” | 进度：” | ，则显示出来的整体的进度条文字是” | 进度：xx%”

函数 | 是否虚函数 | 说明
---- | ----
SetProgress | 否 | 设置进度条的值
GetProgress | 否 | 获取进度条的值
SetMaxProgress | 否 | 设置进度条的最大值
GetMaxProgress | 否 | 获取进度条的最大值
SetRun | 否 | 设置自动运行

## DUI 输入框控件
类名：CDuiEdit
控件名：edit
说明：输入框控件，是从 CControlBaseFont 派生的，所以基类控件具有的属性和函数也可以使用。

属性名 | 类型 | 说明
password | 0,1 | 是否是密码输入框
multiline | 0,1 | 是否多行输入框
autohscroll | 0,1 | 水平宽度超出是否可以滚动显示
autovscroll | 0,1 | 垂直高度超出是否可以滚动显示
number | 0,1 | 是否只能输入数字
readonly | 0,1 | 是否只读
maxchar | 数字 | 最多可以输入多少个字符
small-image | 字符串 | 输入框右边显示的小图片

函数 | 是否虚函数 | 说明
GetEditText | 否 | 获取编辑框文字
SetSmallBitmap | 否 | 设置小图片

## DUI 下拉列表控件
类名：CDuiComboBox
控件名：combobox
说明：下拉列表控件，是从 CDuiEdit 派生的，所以基类控件具有的属性和函数也可以使用。

属性名 | 类型 | 说明
---- | ----
value | 字符串 | 下拉列表当前选择项的值
head-image | 字符串 | 下拉列表项中左边部分图片的掩码图片
del-image | 字符串 | 下拉列表项中每一行的删除按钮，如果没有设置则不能由用户删除列表项
xml | 字符串 | 指定下拉列表内容定义的xml文件，xml文件中的下拉列表项描述的格式和非独立xml文件方式定义的格式完全相同，如果不指定xml文件，则使用当前控件属性的下级节点定义的xml内容
crtext | 颜色 | 列表项的颜色，如果是名字和描述两行内容，则表示名字的颜色
crdesc | 颜色 | 名字和描述两行内容的情况下，表示描述部分的颜色
crhover | 颜色 | 当前选中的行（鼠标移动的行）的颜色

Combobox | 的下来列表内容有三种定义的方式，分别是在控件xml的下级节点定义、通过独立xml文件定义、调用控件的AddItem函数通过程序添加，xml定义方式中下拉项的xml节点名是item，节点属性如下表：

属性名 | 类型 | 说明
---- | ----
name | 字符串 | 列表项名字
desc | 字符串 | 列表项描述。如果所有列表项的描述字符串都为空，则下拉列表自动调整为单行显示；否则当鼠标移动到某一项时候此列表项显示为两行（名字和描述各一行），其他列表项则只显示一行（名字）
value | 字符串 | 列表项的值
image | 字符串 | 列表项中左边部分图片
crtext | 颜色 | 列表项的颜色，如果是名字和描述两行内容，则表示名字的颜色
crdesc | 颜色 | 名字和描述两行内容的情况下，表示描述部分的颜色

函数 | 是否虚函数 | 说明
---- | ----
SetComboValue | 否 | 设置下拉列表的值
GetComboValue | 否 | 获取下拉列表的值
GetItemCount | 否 | 获取下拉列表项个数
AddItem | 否 | 添加列表项
ClearItems | 否 | 清空列表项

## DUI 列表控件
类名：CDuiListCtrl
控件名：listctrl
说明：列表控件，是从 CDuiPanel 派生的，所以基类控件具有的属性和函数也可以使用。

属性名 | 类型 | 说明
---- | ----
img-scroll | 字符串 | 滚动条图片
img-sep | 字符串 | 行分隔线图片
img-check | 字符串 | 行左侧的检查框图片
font-title | 字符串 | 标题行字体
crtext | 颜色 | 正文文字颜色
crhover | 颜色 | 鼠标移动时候的文字颜色
crpush | 颜色 | 鼠标点击的行的文字颜色
crtitle | 颜色 | 标题颜色
crsep | 颜色 | 分割线颜色
crrowhover | 颜色 | 鼠标移动时候行的背景颜色，可以设置透明度
img-width | 数字 | 图片宽度
row-height | 数字 | 行高度
bk-transparent | 数字 | 背景透明度，0-100，0表示全透明，100表示不透明
wrap | 0,1 | 文字是否可以换行
down-row | 0,1 | 当前点击的行是否特殊显示
row-tip | 0,1 | 是否显示行的tooltip(如果title部分超出则tip显示title，否则如果content部分超出则tip显示content内容)

函数 | 是否虚函数 | 说明
---- | ----
InsertItem | 否 | 添加行
GetItemInfo | 否 | 根据行索引号获取行的数据结构
ClearItems | 否 | 清空所有行

## DUI 表格控件
类名：CDuiGridCtrl
控件名：gridctrl
说明：表格控件，是从 CDuiPanel 派生的，所以基类控件具有的属性和函数也可以使用。

属性名 | 类型 | 说明
---- | ----
img-scroll | 字符串 | 滚动条图片
img-sep | 字符串 | 行分隔线图片
img-check | 字符串 | 行左侧的检查框图片
font-title | 字符串 | 标题行字体
crtext | 颜色 | 正文文字颜色
crhover | 颜色 | 鼠标移动时候的文字颜色
crpush | 颜色 | 鼠标点击的行的文字颜色
crtitle | 颜色 | 标题颜色
crsep | 颜色 | 分割线颜色
crrowhover | 颜色 | 鼠标移动时候行的背景颜色，可以设置透明度
img-width | 数字 | 图片宽度
row-height | 数字 | 行高度
bk-transparent | 数字 | 背景透明度，0-100，0表示全透明，100表示不透明
row | 子节点 | 表示表格的行节点，具体属性参见行节点表
left-pos | 数字 | 左侧起始位置
wrap | 0,1 | 文字是否可以换行
down-row | 0,1 | 当前点击的行是否特殊显示
grid-tip | 0,1 | 是否显示单元格的tip信息


列属性：

属性名 | 类型 | 说明
---- | ----
title | 字符串 | 列标题（暂不支持）
crtext | 颜色 | 列文字颜色（暂不支持）
width | 数字 | 列宽度
valign | 枚举 | 文字的垂直对齐模式，top、middle、bottom
align | 枚举 | 文字的水平对齐模式，left、center、right

行属性：

属性名 | 类型 | 说明
---- | ----
id | 字符串 | 行ID，可用于定位
check | 0,1 | 是否显示行左侧的检查框
image | 字符串 | 行左侧的图片
right-img | 字符串 | 行右侧的图片
crtext | 颜色 | 正文文字颜色
item | 子节点 | 表示此行的单元格节点，具体属性参见单元格节点表


单元格属性：

属性名 | 类型 | 说明
---- | ----
title | 字符串 | 单元格标题
content | 字符串 | 单元格内容
image | 字符串 | 单元格图片
link | 字符串 | 单元格链接文字
linkaction | 字符串 | 单元格链接动作
crtext | 颜色 | 正文文字颜色
font-title | 0,1 | 是否使用标题字体显示单元格文字

单元格可以添加子控件，XML中只要在单元格的item节点下面添加控件子节点就可以。

函数：

函数 | 是否虚函数 | 说明
---- | ----
InsertItem | 否 | 添加行
GetItemInfo | 否 | 根据行索引号获取行的数据结构
ClearItems | 否 | 清空所有行
AddSubItemControl | 否 | 添加单元格子控件
DeleteSubItemControl | 否 | 删除单元格子控件

##DUI 树控件
类名：CDuiTreeCtrl
控件名：treectrl
说明：树控件，是从 CDuiPanel 派生的，所以基类控件具有的属性和函数也可以使用。

属性：

属性名 | 类型 | 说明
---- | ----
img-scroll | 字符串 | 滚动条图片
img-sep | 字符串 | 行分隔线图片
img-check | 字符串 | 节点左侧的选择框图片
img-collapse | 字符串 | 行缩放图片
img-toggle | 字符串 | 树节点收缩图片
font-title | 字符串 | 标题行字体
crtext | 颜色 | 正文文字颜色
crhover | 颜色 | 鼠标移动时候的文字颜色
crpush | 颜色 | 鼠标点击的行的文字颜色
crtitle | 颜色 | 标题颜色
crsep | 颜色 | 分割线颜色
img-width | 数字 | 图片宽度
row-height | 数字 | 行高度
bk-transparent | 数字 | 背景透明度，0-100，0表示全透明，100表示不透明
node | 子节点 | 树节点
left-pos | 数字 | 左侧起始位置
wrap | 0,1 | 文字是否可以换行
down-row | 0,1 | 当前点击的行是否特殊显示
grid-tip | 0,1 | 是否显示单元格的tip信息

树节点属性：

属性名 | 类型 | 说明
---- | ----
id | 字符串 | 行ID，可用于定位
check | 0,1 | 是否显示行左侧的检查框
image | 字符串 | 行左侧的图片
right-img | 字符串 | 行右侧的图片
crtext | 颜色 | 正文文字颜色
collapse | 0,1 | 当前是否处于收缩状态
item | 子节点 | 表示此行的单元格节点，具体属性参见单元格节点表

单元格属性：

属性名 | 类型 | 说明
---- | ----
title | 字符串 | 单元格标题
content | 字符串 | 单元格内容
image | 字符串 | 单元格图片
link | 字符串 | 单元格链接文字
linkaction | 字符串 | 单元格链接动作
crtext | 颜色 | 正文文字颜色
font-title | 0,1 | 是否使用标题字体显示单元格文字
collapse | 0,1 | 此单元格是否可以用于收缩和展开子节点单元格可以添加子控件，XML中只要在单元格的item节点下面添加控件子节点就可以。

函数：

函数 | 是否虚函数 | 说明
---- | ----
InsertNode | 否 | 添加树节点
GetNodeInfo | 否 | 根据节点句柄获取节点的数据结构
ClearNodes | 否 | 清空所有节点
AddSubItemControl | 否 | 添加单元格子控件
DeleteSubItemControl | 否 | 删除单元格子控件
