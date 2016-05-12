title: 编译器产生的输出文件的MD5值与生成代码的关系
date: 2013-04-02
tags: 编译器,MD5,输出文件
categories: 编译器
---

<h3 id="md5">编译器产生的输出文件的MD5值与生成代码的关系</h3>
<h4>背景</h4>
<p>因发布给用户的产品需要升级，每次升级使用的是版本号加&rdquo;增量更新&ldquo;的方法，自动更新服务器保存当前版本号与所有文件的MD5值，用户本地保存本地的版本号。登录时，若用户本地的版本号与服务器上的版本号不一致，则根据嗠器上文件的MD5与本地所有文件计算的MD5值比较，若不同，则更新。但现在在Delphi6中，程序的代码没有作更新，前后再次产生的文件的MD5却不同。Delphi6将生成文件的当前时间戳添加到了输出文件中。从二进制进的角度来看，代码相同的程序，只是编译的时间不一样，产生的文件却是不同的。Delphi6的这种画蛇添足的做法，实在叫人费解。</p>
<!--more-->
<p>既然Delphi6这么做，难道这是&ldquo;业内行规&rdquo;，为了弄明白，那就只有一一探个明白了。</p>
<h4 id="delphi6md5">Delphi6输出文件MD5</h4>
<p>已经知道了，代码不一样编译时间不同，其输出文件的MD5也不同，但若是在输出文件中，将编译时间戳的影响去除了，是其它因素对它的影响是怎样的呢。</p>
<p>程序如下：&nbsp;<br />program BuildOutput_6;</p>
<pre><code>{$APPTYPE CONSOLE}  

uses  
  SysUtils;  

var  
  aNum: Integer;  

begin
  Writeln('==Delphi 6的输出文件==');  

  aNum := 0;

  Writeln(IntToStr(aNum)); 

  Writeln('按回车键退出。');
  readln;
end.
</code></pre>
<p>在不修改代码的情况下，编译两次产生的文件：BuildOutput<em>6</em>1.exe、BuildOutput<em>6</em>2.exe，另外将时间戳强制改为2008-08-08 08:08:08之后分别产生的文件为：BuildOutput<em>6</em>1<em>0.exe和BuildOutput</em>6<em>2</em>0.exe；&nbsp;<br />4个文件的MD5值:&nbsp;<br />BuildOutput<em>6</em>1.exe 7C4574F2C7614273076D78BC06C5824F&nbsp;<br />BuildOutput<em>6</em>2.exe 5BF6B4BF7DE37A71BC4060F95C544FC5&nbsp;<br />BuildOutput<em>6</em>2<em>0.exe 68B6114E66DC2912656810278C616A94&nbsp;<br />BuildOutput</em>6<em>2</em>0.exe 68B6114E66DC2912656810278C616A94</p>
<p>将代码做略微调整：&nbsp;<br />program BuildOutput_6;</p>
<pre><code>{$APPTYPE CONSOLE} 

uses
  SysUtils; 

var
  aNum: Integer;

begin
  Writeln('==Delphi 6的输出文件==');

  aNum := 1;

  Writeln(IntToStr(aNum));

  Writeln('按回车键退出。');
  readln;
end.
</code></pre>
<p>此次产生的BuildOutput<em>6</em>3<em>0.exe的MD5值：&nbsp;<br />BuildOutput</em>6<em>3</em>0.exe 4B44FC3E49A35081004D1D7BCE4BDAB4&nbsp;<br />BuildOutput<em>6</em>3_0.exe的MD5与前面2对应文件的MD5值不同，代码的改动，已经影响的最后的输出文件。</p>
<p>将代码的语句顺序调整： program BuildOutput_6;</p>
<pre><code>{$APPTYPE CONSOLE}

uses
  SysUtils;

var
  aNum: Integer;

begin
  aNum := 0;

  Writeln('==Delphi 6的输出文件==');

  Writeln(IntToStr(aNum));

  Writeln('按回车键退出。');
  readln;
end.
</code></pre>
<p>此次产生的BuildOutput<em>6</em>4<em>0.exe的MD5值：&nbsp;<br />BuildOutput</em>6<em>4</em>0.exe DB2DA5F6F3AE5B3F71330DB17A116A98&nbsp;<br />BuildOutput<em>6</em>4_0.exe的MD5与前面2对应文件的MD5值不同，代码的改动，已经影响的最后的输出文件。</p>
<p>将代码的空白行去除：&nbsp;<br />program BuildOutput_6;</p>
<pre><code>{$APPTYPE CONSOLE}

uses
  SysUtils;

var
  aNum: Integer;

begin
  Writeln('==Delphi 6的输出文件==');
  aNum := 0;

  Writeln(IntToStr(aNum));

  Writeln('按回车键退出。');
  readln;
end.
</code></pre>
<p>此次产生的BuildOutput<em>6</em>5<em>0.exe的MD5值：&nbsp;<br />BuildOutput</em>6<em>5</em>0.exe 68B6114E66DC2912656810278C616A94&nbsp;<br />BuildOutput<em>6</em>5_0.exe的MD5与前面2对应文件的MD5值一样同。</p>
<h4 id="javamd5">JAVA输出文件MD5</h4>
<p>JAVA代码如下: public class BuildOutput_6 {</p>
<pre><code>     public static void main(String[] args){
         System.out.println("==Java 6的输出文件==");

         int i = 0;

         System.out.println("i = " + i);

     }
 }
</code></pre>
<p>两次编译产生的.class文件的MD5如下： BuildOutput<em>6</em>1.class 5F50C342B17C8DBFE22E3E71311CB358&nbsp;<br />BuildOutput<em>6</em>2.class 5F50C342B17C8DBFE22E3E71311CB358&nbsp;<br />两个MD5值相同。</p>
<h4 id="vc90md5">VC9.0输出文件MD5</h4>
<p>代码如下： public class BuildOutput_6 {</p>
<pre><code>     public static void main(String[] args){
         System.out.println("==Java 6的输出文件==");

         int i = 0;

         System.out.println("i = " + i);

     }
 }
</code></pre>
<p>两次编译产生的.exe文件的MD5如下： BuildOutput<em>1.exe EB77754D7216FD4AC3FF62EB7A0E3A2B&nbsp;<br />BuildOutput</em>2.exe 17BFC8E72EEC200E5E792673C56BB236&nbsp;<br />两个MD5值不同，应该与Delphi6类似。</p>
<p>似乎事情有些线索了，输出为可执行文件时，相同代码两次产生的MD5值会不同。但学需要进一步验证。下次有时间之后继续。</p>
<p>原始博文连接：<a href="http://www.lontoken.com/blog/?p=28">http://www.lontoken.com/blog/?p=28</a></p>
<p>&nbsp;</p>