title: Delphi中由TComponent.Owner引发的资源重复释放问题
date: 2012-09-24
tags: TComponent.Owner,资源重复释放
categories: Delphi
---

<p>案例情形：在通过控件的构造函数Create(AOwner: TComponent)创建对象a时传入Application，之后又自作多情的主动调用FreeAndNil释放此对象a，在程序退出时问题就会来了，由于Application会主动释放自己的Components内的元素，而我们自己再次调用FreeAndNil时就会出现对象的多次释放，导致程序无法正常退出！！！</p>
<!--more-->
<p>反例代码：</p>
<div class="cnblogs_code" style="background-color: #f5f5f5; border: #cccccc 1px solid; padding: 5px;">
<pre>//<span style="color: #000000;">在Create时创建对象
FFoolPan :</span>=<span style="color: #000000;"> TPanel.Create(Application);
 
</span>//<span style="color: #000000;">在Destroy时释放资源
</span>//<span style="color: #000000;">旁白：不要以为做了Assigned判断就万事大吉了，遇到&rdquo;悬空指定&rdquo;你会死得很难看 
</span><span style="color: #0000ff;">if</span> Assigned(FFoolPan ) <span style="color: #0000ff;">then</span> FreeAndNil(FFoolPan );</pre>
</div>
<p>&nbsp;</p>
<p>好了，现在开始分析问题的原因，为了刨根问底，我们只有深入Delphi的VCL去探险了&hellip;<br />要知道TPanel.Create()到底做了什么，我们得去问TPanel的祖先类TComponent，因因它有了组件列表的概念，列表中的元素必须是TComponent的实例，且属于此TComponent，它会在其构造函数中主动的释放列表中的实例，注意，它只会调用列表中元素实例的Destroy方法，而不将其置为nil，&ldquo;悬空指针&rdquo;从此诞生。</p>
<div class="cnblogs_code" style="background-color: #f5f5f5; border: #cccccc 1px solid; padding: 5px;">
<pre>//------在Create时将自己插入到组件列表Components当中  Start------//
<span style="color: #0000ff;">constructor</span><span style="color: #000000;"> TComponent.Create(AOwner: TComponent);
</span><span style="color: #0000ff;">begin</span><span style="color: #000000;">
  FComponentStyle :</span>=<span style="color: #000000;"> [csInheritable];
  </span><span style="color: #0000ff;">if</span> AOwner &lt;&gt; <span style="color: #0000ff;">nil</span> <span style="color: #0000ff;">then</span> AOwner.InsertComponent(Self);       //<span style="color: #000000;">将自己插入到AOwner的组件列表中
</span><span style="color: #0000ff;">end</span><span style="color: #000000;">;

</span>//<span style="color: #000000;">我们来看看InsertComponent到底做了什么    (PS:由贴出了与此问题相关的代码，下同)
</span><span style="color: #0000ff;">procedure</span><span style="color: #000000;"> TComponent.InsertComponent(AComponent: TComponent);
</span><span style="color: #0000ff;">begin</span><span style="color: #000000;">
  AComponent.ValidateContainer(Self);
  ValidateRename(AComponent, </span><span style="color: #800000;">''</span><span style="color: #000000;">, AComponent.FName);
  Insert(AComponent);       </span>//<span style="color: #000000;">Insert就发生在此时
</span><span style="color: #0000ff;">end</span><span style="color: #000000;">;

</span><span style="color: #0000ff;">procedure</span><span style="color: #000000;"> TComponent.Insert(AComponent: TComponent);
</span><span style="color: #0000ff;">begin</span>
  <span style="color: #0000ff;">if</span> FComponents = <span style="color: #0000ff;">nil</span> <span style="color: #0000ff;">then</span> FComponents :=<span style="color: #000000;"> TList.Create;
  FComponents.Add(AComponent);
  AComponent.FOwner :</span>=<span style="color: #000000;"> Self;
</span><span style="color: #0000ff;">end</span><span style="color: #000000;">;
</span>//------在Create时将自己插入到组件列表Components当中  Edn------//</pre>
</div>
<p>&nbsp;</p>
<div class="cnblogs_code" style="background-color: #f5f5f5; border: #cccccc 1px solid; padding: 5px;">
<pre>//------TComponent释放组件列表Components中的实例  Start------//
<span style="color: #0000ff;">destructor</span><span style="color: #000000;"> TComponent.Destroy;
</span><span style="color: #0000ff;">begin</span><span style="color: #000000;">
  Destroying;
  DestroyComponents;            </span>//<span style="color: #000000;">释放Components
  </span><span style="color: #0000ff;">if</span> FOwner &lt;&gt; <span style="color: #0000ff;">nil</span> <span style="color: #0000ff;">then</span><span style="color: #000000;"> FOwner.RemoveComponent(Self);
  </span><span style="color: #0000ff;">inherited</span><span style="color: #000000;"> Destroy;
</span><span style="color: #0000ff;">end</span><span style="color: #000000;">;

</span><span style="color: #0000ff;">procedure</span><span style="color: #000000;"> TComponent.DestroyComponents;
</span><span style="color: #0000ff;">var</span><span style="color: #000000;">
  Instance: TComponent;
</span><span style="color: #0000ff;">begin</span>
  <span style="color: #0000ff;">while</span> FComponents &lt;&gt; <span style="color: #0000ff;">nil</span> <span style="color: #0000ff;">do</span>
  <span style="color: #0000ff;">begin</span><span style="color: #000000;">
    Instance :</span>=<span style="color: #000000;"> FComponents.Last;
    </span><span style="color: #0000ff;">if</span> (csFreeNotification <span style="color: #0000ff;">in</span><span style="color: #000000;"> Instance.FComponentState)
      </span><span style="color: #0000ff;">or</span> (FComponentState * [csDesigning, csInline] = [csDesigning, csInline]) <span style="color: #0000ff;">then</span><span style="color: #000000;">
      RemoveComponent(Instance)
    </span><span style="color: #0000ff;">else</span><span style="color: #000000;">
      Remove(Instance);
    Instance.Destroy;           </span>//<span style="color: #000000;">只调用了Destroy，却没置为nil，引入悬空指针，情何以堪...
  </span><span style="color: #0000ff;">end</span><span style="color: #000000;">;
</span><span style="color: #0000ff;">end</span><span style="color: #000000;">;
</span>//------TComponent释放组件列表Components中的实例  End------//</pre>
</div>
<p>现在我们明白了Create(Application)和Create(nil)的一个重要的区别了：使用Create(Application)所创建的对象的释放由Application来做，Create(nil)构造的对象需要自己来做资源的释放。</p>
<p>那程序退出时，Delphi都做了些什么呢？</p>
<p>我们从简单的情况入手，看看在系统的主窗体关闭时，我们的程序都执行了些什么操作。我们来看TCustomForm的WMClose，它的声明如下：</p>
<p>procedure WMClose(var Message: TWMClose); message WM_CLOSE;<br />既然接收了WM_CLOSE消息，那到底做了什么呢？</p>
<div class="cnblogs_code" style="background-color: #f5f5f5; border: #cccccc 1px solid; padding: 5px;">
<pre><span style="color: #0000ff;">procedure</span> TCustomForm.WMClose(<span style="color: #0000ff;">var</span><span style="color: #000000;"> Message: TWMClose);
</span><span style="color: #0000ff;">begin</span><span style="color: #000000;">
  Close;      </span>//<span style="color: #000000;">很简单，只是调用了Close而已
</span><span style="color: #0000ff;">end</span>;</pre>
</div>
<p>真像会在Close里面吗？</p>
<div class="cnblogs_code" style="background-color: #f5f5f5; border: #cccccc 1px solid; padding: 5px;">
<pre><span style="color: #0000ff;">procedure</span><span style="color: #000000;"> TCustomForm.Close;
</span><span style="color: #0000ff;">var</span><span style="color: #000000;">
  CloseAction: TCloseAction;
</span><span style="color: #0000ff;">begin</span>
  <span style="color: #0000ff;">if</span> fsModal <span style="color: #0000ff;">in</span> FFormState <span style="color: #0000ff;">then</span><span style="color: #000000;">
    ModalResult :</span>=<span style="color: #000000;"> mrCancel
  </span><span style="color: #0000ff;">else</span>
    <span style="color: #0000ff;">if</span> CloseQuery <span style="color: #0000ff;">then</span>
    <span style="color: #0000ff;">begin</span>
      <span style="color: #0000ff;">if</span> FormStyle = fsMDIChild <span style="color: #0000ff;">then</span>
        <span style="color: #0000ff;">if</span> biMinimize <span style="color: #0000ff;">in</span> BorderIcons <span style="color: #0000ff;">then</span><span style="color: #000000;">
          CloseAction :</span>= caMinimize <span style="color: #0000ff;">else</span><span style="color: #000000;">
          CloseAction :</span>=<span style="color: #000000;"> caNone
      </span><span style="color: #0000ff;">else</span><span style="color: #000000;">
        CloseAction :</span>=<span style="color: #000000;"> caHide;
      DoClose(CloseAction);
      </span><span style="color: #0000ff;">if</span> CloseAction &lt;&gt; caNone <span style="color: #0000ff;">then</span>
        <span style="color: #0000ff;">if</span> Application.MainForm = Self <span style="color: #0000ff;">then</span> Application.Terminate   //<span style="color: #000000;">我们找到Application.Terminate了，不错
        </span><span style="color: #0000ff;">else</span> <span style="color: #0000ff;">if</span> CloseAction = caHide <span style="color: #0000ff;">then</span><span style="color: #000000;"> Hide
        </span><span style="color: #0000ff;">else</span> <span style="color: #0000ff;">if</span> CloseAction = caMinimize <span style="color: #0000ff;">then</span> WindowState :=<span style="color: #000000;"> wsMinimized
        </span><span style="color: #0000ff;">else</span><span style="color: #000000;"> Release;
    </span><span style="color: #0000ff;">end</span><span style="color: #000000;">;
</span><span style="color: #0000ff;">end</span>;</pre>
</div>
<p>Application.Terminate的作用时让程序终止执行，即退出应用程序。更详细的说明是Terminate会通过调用PostQuitMessage(0)发送消息WM_QUIT面终止程序。</p>
<div class="cnblogs_code" style="background-color: #f5f5f5; border: #cccccc 1px solid; padding: 5px;">
<pre><span style="color: #0000ff;">function</span> TApplication.ProcessMessage(<span style="color: #0000ff;">var</span><span style="color: #000000;"> Msg: TMsg): Boolean;
</span><span style="color: #0000ff;">var</span><span style="color: #000000;">
  Handled: Boolean;
</span><span style="color: #0000ff;">begin</span><span style="color: #000000;">
  Result :</span>=<span style="color: #000000;"> False;
  </span><span style="color: #0000ff;">if</span> PeekMessage(Msg, <span style="color: #800080;">0</span>, <span style="color: #800080;">0</span>, <span style="color: #800080;">0</span>, PM_REMOVE) <span style="color: #0000ff;">then</span>   //<span style="color: #000000;">在消息队列中获取消息
  </span><span style="color: #0000ff;">begin</span><span style="color: #000000;">
    Result :</span>=<span style="color: #000000;"> True;
    </span><span style="color: #0000ff;">if</span> Msg.Message &lt;&gt; WM_QUIT <span style="color: #0000ff;">then</span>
    <span style="color: #0000ff;">begin</span><span style="color: #000000;">
      Handled :</span>=<span style="color: #000000;"> False;
      </span><span style="color: #0000ff;">if</span> Assigned(FOnMessage) <span style="color: #0000ff;">then</span><span style="color: #000000;"> FOnMessage(Msg, Handled);
      </span><span style="color: #0000ff;">if</span> <span style="color: #0000ff;">not</span> IsHintMsg(Msg) <span style="color: #0000ff;">and</span> <span style="color: #0000ff;">not</span> Handled <span style="color: #0000ff;">and</span> <span style="color: #0000ff;">not</span> IsMDIMsg(Msg) <span style="color: #0000ff;">and</span>
        <span style="color: #0000ff;">not</span> IsKeyMsg(Msg) <span style="color: #0000ff;">and</span> <span style="color: #0000ff;">not</span> IsDlgMsg(Msg) <span style="color: #0000ff;">then</span>
      <span style="color: #0000ff;">begin</span><span style="color: #000000;">
        TranslateMessage(Msg);
        DispatchMessage(Msg);
      </span><span style="color: #0000ff;">end</span><span style="color: #000000;">;
    </span><span style="color: #0000ff;">end</span>
    <span style="color: #0000ff;">else</span>    //<span style="color: #000000;">如果接收到WM_QUIT消息，将退出标志置为true 
      FTerminate :</span>=<span style="color: #000000;"> True;           
  </span><span style="color: #0000ff;">end</span><span style="color: #000000;">;
</span><span style="color: #0000ff;">end</span>;</pre>
</div>
<p>在中我们可以在TApplication.Run看到，系统会通过HandleMessage调用ProcessMessage处理消息，直到退出标志为true时，才终止。</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<div class="cnblogs_code" style="background-color: #f5f5f5; border: #cccccc 1px solid; padding: 5px;">
<pre><span style="color: #0000ff;">procedure</span><span style="color: #000000;"> TApplication.Run;
</span><span style="color: #0000ff;">begin</span><span style="color: #000000;">
  FRunning :</span>=<span style="color: #000000;"> True;
  </span><span style="color: #0000ff;">try</span><span style="color: #000000;">
    AddExitProc(DoneApplication);    </span>//<span style="color: #000000;">将DoneApplication添加到TApplication.Run退出之后执行列表中
    </span><span style="color: #0000ff;">if</span> FMainForm &lt;&gt; <span style="color: #0000ff;">nil</span> <span style="color: #0000ff;">then</span>
    <span style="color: #0000ff;">begin</span>
      <span style="color: #0000ff;">case</span> CmdShow <span style="color: #0000ff;">of</span><span style="color: #000000;">
        SW_SHOWMINNOACTIVE: FMainForm.FWindowState :</span>=<span style="color: #000000;"> wsMinimized;
        SW_SHOWMAXIMIZED: MainForm.WindowState :</span>=<span style="color: #000000;"> wsMaximized;
      </span><span style="color: #0000ff;">end</span><span style="color: #000000;">;
      </span><span style="color: #0000ff;">if</span> FShowMainForm <span style="color: #0000ff;">then</span>
        <span style="color: #0000ff;">if</span> FMainForm.FWindowState = wsMinimized <span style="color: #0000ff;">then</span><span style="color: #000000;">
          Minimize </span><span style="color: #0000ff;">else</span><span style="color: #000000;">
          FMainForm.Visible :</span>=<span style="color: #000000;"> True;
      </span><span style="color: #0000ff;">repeat</span>
        <span style="color: #0000ff;">try</span><span style="color: #000000;">
          HandleMessage;
        </span><span style="color: #0000ff;">except</span><span style="color: #000000;">
          HandleException(Self);
        </span><span style="color: #0000ff;">end</span><span style="color: #000000;">;
      </span><span style="color: #0000ff;">until</span> Terminated;         //<span style="color: #000000;">退出标志为true时退出
    </span><span style="color: #0000ff;">end</span><span style="color: #000000;">;
  </span><span style="color: #0000ff;">finally</span><span style="color: #000000;">
    FRunning :</span>=<span style="color: #000000;"> False;
  </span><span style="color: #0000ff;">end</span><span style="color: #000000;">;
</span><span style="color: #0000ff;">end</span>;</pre>
</div>
<p>&nbsp;</p>
<p>Application.Ran退出后，我们看看DoneApplication会做些什么。</p>
<div class="cnblogs_code" style="background-color: #f5f5f5; border: #cccccc 1px solid; padding: 5px;">
<pre><span style="color: #0000ff;">procedure</span><span style="color: #000000;"> DoneApplication;
</span><span style="color: #0000ff;">begin</span>
  <span style="color: #0000ff;">with</span> Application <span style="color: #0000ff;">do</span>
  <span style="color: #0000ff;">begin</span>
    <span style="color: #0000ff;">if</span> Handle &lt;&gt; <span style="color: #800080;">0</span> <span style="color: #0000ff;">then</span><span style="color: #000000;"> ShowOwnedPopups(Handle, False);
    ShowHint :</span>=<span style="color: #000000;"> False;      
    Destroying;             
    DestroyComponents;      </span>//<span style="color: #000000;">调用Application.DestroyComponents方法
  </span><span style="color: #0000ff;">end</span><span style="color: #000000;">;
</span><span style="color: #0000ff;">end</span>;</pre>
</div>
<p>&nbsp;</p>
<p>我们现在又回到了DestroyComponents方法，很熟悉的感觉，Application的DestroyComponents会有什么不同呢？</p>
<p>情况并没有不同，它没有重写DestroyComponents，还是使用的TComponent.DestroyComponents方法。好了，现在我们也该明白为什么在TPanel.Create(Application)之后，不会再手动调用FreeAndNil(FFoolPan )了。</p>
<p>切忌：内存的重复释放引发的危害，远远比内存泄漏来得大来得猛烈。</p>
<p>有一篇博文就是讲&ldquo;为什么重复free()比内存泄漏危害更大&rdquo;，有兴趣的同学可以过去瞧瞧。</p>
<p>说了这么多，我们也该休息下了 :)</p>
<p>&nbsp;</p>
<p>------------仅以此文，献给我自己、HOMS开发的同学们，还有深受客户端退出无响应的受害者------------</p>
<p>新博客地址连接：<a href="http://www.lontoken.com/blog/?p=5">http://www.lontoken.com/blog/?p=5</a></p>