[ ](/)

[ ](/newsend)

[ ](/wp-login.php?action=logout&redirect_to=http%3A%2F%2Fdrops.wooyun.org)

[

乌云知识库

](/)

# 域内渗透基本技巧

[ 碧云涛](/author/碧云涛) · 2013/08/07 18:50

## 0x00 什么是域

* * *

有人说域是一种组织结构，是个安全边界。域也有另为一个名字，“活动目录”。不管域是什么，它总有一些特点，有了它，对管理一个机构的组织单元，人员，特别是计算机就
方便了许多，特别是计算机，因为域就是建立在计算机上的。加入域里的计算机，共享文件，共享上网就会很方便。

域里有个比较重要的角色，域控制器(DC)。它有至高无上的权限，它可以在域里对其他计算机(DA)胡作非为。域控制器里还有域内所有用户的帐号密码(曾在一个机房的
机架上看到一张字条，上面写着"帐号密码是黑客攻击的首要目标"，当时偶就笑抽了，机架的锁上还写着"这里锁住的是企业的秘密")。

综上所诉，域就是我们在进行渗透测试时，会碰到的一种网络环境。对域比较直观的展示时，在登录3389时，在域里的机器会多一个下拉框，见图。

![enter image description
here](http://static.wooyun.org/20131018/2013101800462942715.png)

如果“登录到”这个下拉框没有显示，请点击“选项（0）”按钮显示。仔细看，这个下拉框有两个选项，一个是此计算机，本地登录。一个是TESTONE域。再看下图

![enter image description
here](http://static.wooyun.org/20131018/2013101800463033672.png)

这个3389登录界面里的“登录到”下拉框只有一个选项TESTONE。没有此计算机的选项。没错，这台机器就是DC，变身成DC的机器本地帐户会被禁用，所以不能登
录本机，只能登录到域中（一些书上是这样说的，偶学艺不精，没有准确验证过）。所以DC是伟大的。

## 0x01 寻找域控制

* * *

假设我们现在已经在一个域里，当然不是正常的存在。因为我们正在做渗透测试呢，而不是坐在公司里上班打dota。

一般情况下会有两种情况，第一：我们能登录一台域里服务器的3389。第二：我们用远程控制软件控制了一台域里的个人机器。只要有其中一种，你一定想，并很快拥有另一
种。

好，情况已经这样美好了，我们下一步该做点什么了，有人说赶紧把毛片和种子下回来，作为一个脱离低级趣味的人来说，我绝对我大声的对他说，俺硬盘里还有云存储里的资源
比他多的多，极品的多，还费的着下他的，切！

第一步我觉得不是导hash，而是再给对方种一个或者两个不同的远控，留条后路，然后导hash，查看各种信息。导hash的工具gsecdump,wce等，网上都
可以下到。

关于导hash，分为导处本地存储的hash和正在登录状态用户的hash，正在登录用户的hash就有可能导到域管理员的，假设你在一台域里的服务器上转悠，这时，
刚好管理员用域控管理帐户来视察这台服务器，那么内存里就有了他的hash，这时里运行一下工具，hash到手，不要说你破解不出来，没关系，还有那个法国神器，直接
给它显示明文吧,没有明文也没关系，还可以hash注入！

关于查看些有用的信息，下面引用网上广为流传的命令。

    
    
    net view                查看同一域/工作组的计算机列表
    net view /domain        查看域/工作组列表
    net view /domain:Secwing     查看Secwing域中 计算机列表
    net group /domain       查看所在域的组
    net user /domain        查看所在域用户
    net user /domain zerosoul 12345678      修改域用户密码，需要域管理员权限，或者Ctrl+Alt+Del点击修改则不需要域管理员权限
    net localgroup administrators SECWINGzerosoul /add         域Users组用户添加到本地Administrators组，需要本地管理员或域管理员在本机登陆域后进行
    

下面的命令 只能用于 域控制器:

    
    
    net group "Domain controllers"      查看域控制器(如果有多台)
    net group                       查看域的组
    net group "domain admins"       查看域管理员
    net group "domain users"        查看域管理员
    

PS:打开配置域控制器向导的命令

    
    
    dcpromo
    
    psexec /accepteula 绕过第一次验证窗口
    mstsc /admin 远程桌面登录到console会话解决hash无法抓出问题
    

我们查看信息的目的是找出域控制器，网上一般提出过很多方法参考地址

<http://hi.baidu.com/cao2109/item/6f7115687616e5166895e682>

不过偶觉得微软提供的方法更好用(一个朋友教的，在此感谢)。官方解说:

<http://msdn.microsoft.com/zh-cn/cc755655%28zh-cn,WS.10%29.aspx>

对，就是用dsquery。在2003自带，system32目录里。要在xp执行，把dsquery.exe和dsquery.dll拷贝出来放在同目录就可以用了
。dsquery不仅可以找域控制器，在一台普通域里的机器上就可以运行，列出域里的基本信息，包括组织单元，计算机，用户，子网，联系人（说实话，偶不懂这个耶）等
。

找到域控后，很多时候不止一台域控。现在不管你用什么方法，exp啊，0day啊，hash注入呀，弱口令呀，社工呀，Arp呀，苍井空啊。弄到域控的权限，有了域管
理员的帐号和密码，那么在域里YY就不愁了。

## 0x02 自由飞翔

* * *

假设现在我们拿到了域控管理员的帐号和密码，我们可以登录域内一台服务器，当然域控也没问题。不过管理会经常登域控，这样偷情肯定很容易被发现，所以最好找一台没有什
么作用，被遗弃的服务器上去操作。

有了帐号密码，IPC链接。内网为了共享啥的方便，server服务都不会关闭，要是我自己电脑server和workstation是肯定要关的。假设我们的域管理
员名字是CK，密码是123456，域的名字叫ALIYUN。我要到域里ip为192.168.4.6的机器上执行命令。我就去会敲出。

    
    
    net use 192.168.4.6c$ 123456 /user:ALIYUNCK
    

要是这个IP其实不在域里，那么把ALIYUN的地方改成192.168.4.6，用户名改成administrator，然后再试试，可能有惊喜哦。

要是命令执行成功，执行net use可以看到已经建立的映射。

然后用psexec来执行命令，第一次运行请记得跳过验证

    
    
    psexec /accepteula 绕过第一次验证窗口
    

不然坐在电脑面前的那个她会吓死的（你要在远控里的话）。

建立映射后，在交互式的cmdshell里，比如3389上去打开的cmdshell执行

    
    
    Psexec -s 192.168.4.6 cmd.exe
    

就可以得到一个192.168.4.6机器上的一个cmd.exe.前提是已经net use映射成功了，接下来要干什么，就看你的了。

这是在交互式的cmdshell里，要是在远控里自带的cmdshell呢？一般远控里的cmdshell会用到管道来实现的，而psexec在回显的时候或许也用了
管道，这样在执行下面的语句是可能会出现问题。

    
    
    Psexec -s 192.168.4.6 cmd.exe /c ipconfig
    

回显会出现时有时无的情况，弹一个交互式的cmdshell就不用想了，除非远控实现的是一个完全交互式的cmdshell，这个东西有吗？答案是有的。见下面链接:

<http://forum.eviloctal.com/thread-40208-1-1.html>

不过有方法可以弥补这个缺陷，net use成功后，在远控里是可以访问对方的文件系统的,192.168.4.6C$,这样就可以写文件进去,比如写到C盘，写批处
理，在里面写想执行的命令并把回显结果重定向到文件，一次写入多条效率比较高，还可以重复使用，就像这样：

    
    
    Ipconfig /all >> C:pp.txt
    

然后再执行

    
    
    psexec -s 192.168.4.6 cmd.exe /c C:a.bat
    

这样在远控里把回显的txt下回来就算行了，还便于保存，是不是很麻烦呀！

监视管理员：

虽然拿到域管理员的密码可以在域里玩，当跟域里有联系而且重要的机器可能并不在域里，比如帮助域里机器上网的代理服务器，邮件网关等。 要想在这些机器上玩，就只有监
视管理员了，找到管理员的机器，给他种个马，种个keyloger(键盘记录)，或者专门的3389登录记录器也可以。当然运气好，可能在他电脑找到一个passwo
rd.txt就发达了。管理员的浏览器也是藏密码的地方，他应该会经常登录一些需要密码验证的地方，然后他可能会保存密码，那么我们就要导出他的密码，那样会收获不小
哦。

## 0x03 问题与提高

* * *

如果你只有远控，对方网络在内网。你无法直接登录服务器上的3389,那就只有端口转发了。 如果对方内网不能直接上网，只能通过一个代理服务器上网，比如ISA，s
quid撒的，再加上一个会随时断你线的防火墙，那么麻烦就大了。就算一些木马能出来，一般的端口转发也出不来啊～。

这种情况我想到两种出路：

一 找到内网里可以直接出来的机器，比如管理员为了方便下片什么的开放的。其实可以直接找管理员。（还有一个思路，看对方内网里发邮件到外面的邮件头，一般都是邮件中
继信息，一步一步跟就从内网到外网了，要是有基于邮件协议的木马就爽YY了，中了以后直接劫持邮件客服端，隐蔽的发邮件，控制信息由邮件出去）

二 写程序，搞定代理，刺穿防火墙。

[ __收藏 ](javascript:void\(0\))

__[分享](javascript:void\(0\))

__新浪微博

  
![](/wooyun/captcha.php)  

__

![30](http://wooyun.b0.upaiyun.com/wooyun_job/avatar/default.png)

burlington coat factory online coupon 2014-08-21 06:11:57

**burlington coat factory online coupon...** 域内渗透基本技巧 | WooYun知识库...

[ __回复 ](javascript:void\(null\))

![30](http://zone.wooyun.org/upload/avatar/avatar_2238.png)

Wenien 2013-11-23 00:47:28

好。。。

[ __回复 ](javascript:void\(null\))

![30](http://zone.wooyun.org/upload/avatar/avatar_1860.jpg)

Aring 2013-08-23 10:36:07

好文章呀

[ __回复 ](javascript:void\(null\))

![30](http://zone.wooyun.org/upload/avatar/avatar_937.jpg)

煦阳。 2013-08-21 01:54:27

细节决定成败.. 好吧。我多嘴了

[ __回复 ](javascript:void\(null\))

![30](http://zone.wooyun.org/upload/avatar/avatar_937.jpg)

煦阳。 2013-08-21 01:53:51

`net use \192.168.4.6c$ 123456 /user:ALIYUNCK net use \\\192.168.4.6\c$
"123456" /user:ALIYUN\ck 执行命令用at是不是会比较好呢？ net time \\\192.168.4.6 at
\\\192.168.4.6 15：50（24小时） cmd /c "c:\wmpub\wce.exe -w>>c:/wmpub/w.txt"`

[ __回复 ](javascript:void\(null\))

![30](http://zone.wooyun.org/upload/avatar/avatar_2326.jpg)

Ivan 2013-08-15 17:47:22

很不错的文章

[ __回复 ](javascript:void\(null\))

![30](http://wooyun.b0.upaiyun.com/wooyun_job/avatar/default.png)

insight-labs 2013-08-10 11:33:50

windows防火墙或者路由ACL，ping不通不代表不能访问啊

[ __回复 ](javascript:void\(null\))

![30](http://zone.wooyun.org/upload/avatar/avatar_2349.jpg)

sdj 2013-08-10 10:44:50

已有域控权限，域管理员账号密码，有一台机器在域中，也开着机，但是ipc连不过去，也ping不通，在域控上和其他多台机器上ping此计算机都不通，这是为何？

[ __回复 ](javascript:void\(null\))

![30](http://wooyun.b0.upaiyun.com/wooyun_job/avatar/default.png)

possible 2013-08-08 10:19:50

学习了 值得收藏

[ __回复 ](javascript:void\(null\))

[ ![](http://static.wooyun.org/static/avatar_50_50.png) ](/author/碧云涛)
[碧云涛](/author/碧云涛)

感谢知乎授权页面模版
