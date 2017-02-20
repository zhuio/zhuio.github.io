---
layout: "post"
title: "ubuntu terminal trick"
date: "2017-02-20 21:49"
tags:
- ubuntu
comments: true
---

#### 一、快捷键


    Shift&#43;Ctrl&#43;T:新建标签页

    Shift&#43;Ctrl&#43;W:关闭标签页

    Ctrl&#43;PageUp:前一标签页

    Ctrl&#43;PageDown:后一标签页

    Shift&#43;Ctrl&#43;PageUp:标签页左移

    Shift&#43;Ctrl&#43;PageDown:标签页右移

    Alt&#43;1:切换到标签页1

    Alt&#43;2:切换到标签页2

    Alt&#43;3:切换到标签页3


    Shift&#43;Ctrl&#43;N:新建窗口

    Shift&#43;Ctrl&#43;Q:关闭终端


    终端中的复制／粘贴:

    Shift&#43;Ctrl&#43;C:复制

    Shift&#43;Ctrl&#43;V:粘贴


    终端改变大小：

    F11：全屏

    Ctrl&#43;plus:放大

    Ctrl&#43;minus:减小

    Ctrl&#43;0:原始大小

#### 二、组合键


1、在终端显示时间使用命令date

如果想让时间显示为一定大&#26684;式，可以使用带参数带date

如，date &#43;%Y/%m/%d就会显示形如2010/7/30的形式

date &#43;%H:%M 会显示11:35


2、显示日历：cal

1）直接输入显示的是当前月份带日历

2）也可以显示某一年的日历cal 2010,也就是cal ［年］

3）总带来说cal的语法&#26684;式如下：

cal [month] [year]

所以还可以显示置顶月份的日历，如 2010年8月的

cal  2010


3、简单的计算器：bc

在输入bc后会显示版本号，然后就可以输入相应的指令了。

运算符： 除了加减乘除以外，还有^指数，%余数运算。

》但在默认时，是不保留小数位的，想要保留小数位，需要输入命令scale=number以指定小数位数。


4、Tab 按键

不用多说，这事Linxu最棒带功能之一。具有［命令补全］和［档案补全］的功能。可以避免我们打错指令或文件名称！


5、Ctrl &#43; c 组合键

如果输入错误的指令或参数，有时候这个指令或程序会在系统下一直不停的运行。这时可以这个组合键以“中断目前程序”。


6、Ctrl &#43; d 组合键。

这个组合键有“End of File，EOF”或“End of Input”的意思。 即［键盘输入结束］。相当于‘exit’。

如想退出终端就可以直接按此组合键。

#### 三、linux图形界面切换到字符界面

1. X-Window图形界面和字符界面自由切换



     一、图形界面切换到字符界面

     ①在X-Window图形操作界面中按“Alt&#43;Ctrl&#43;Fn（n=1~6）”就可以进入Console字符操作界面。

       这就意味着你可以同时拥有X-Window加上6个Console字符操作界面。

      ②如果不行，就加上Backspace键：（同时按住Alt&#43;Ctrl，在按一下Backspace并松开，再按Fn）

       在X-Window图形操作界面中按“Alt&#43;Ctrl&#43;Backspace&#43;Fn（n=1~6）”就可以进入Console字符操作界面。

      二、字符界面切换到图像界面

      ①按“Alt&#43;Ctrl&#43;F7”或者“Alt&#43;Ctrl&#43;Backspace&#43;F7”即可。

         这时Linux默认打开7个屏幕，编号为tty1~tty7。X-Window启动后，占用的是tty7号屏幕，tty1~tty6仍为字符界面屏幕。也就是说，用“Alt&#43;Ctrl&#43;Fn”组合键即可实现字符界面与X Window界面的快速切换。

2.开机进入字符界面设置

    为了在Linux开机启动时直接进入Console字符界面，我们可以编辑/etc/inittab文件。

    找到id:5: initdefault:这一行，将它改为id:3:initdefault:后重新启动系统即可。

    我们看到，简简单单地将5改为3，就能实现启动时进入X-Window图形操作界面或Console字符界面的转换，这是因为Linux操作系统有六种不同的运行级（run level），在不同的运行级下，系统有着不同的状态，这六种运行级分别为：

      0 ：停机（记住不要把initdefault 设置为0，因为这样会使Linux无法启动 ）

     1：单用户模式，就像Win9X下的安全模式。

     2：多用户，但是没有 NFS 。

     3：完全多用户模式，标准的运行级。

     4：一般不用，在一些特殊情况下可以用它来做一些事情。

     5：X11，即进到 X-Window 系统。

     6：重新启动 （记住不要把initdefault 设置为6，因为这样会使Linux不断地重新启动）。


     其中运行级3就是我们要进入的标准Console字符界面模式。

#### 四、Linux终端字符界面显示乱码解决方法


方法一：配置SSH工具



1. SecureCRT中文版配置  

[全局选项]→[默认会话]→[编辑默认设置]→[终端]→[外观]→[字体]→[新宋体 10pt CHINESE_GB2312]→[字符编码 UTF-8]  



2. Putty配置  

[window]→[Appearance]→[Font settings]→[Change]→[Fixedsys CHINESE_GB2312]  

[window]→[Appearance]→[Translation]→[Received data assumed to be in which character set]→[Use font encoding UTF-8]  

如果经常使用,把这些设置保存在session里面。  

打开putty，登录成功后，在shell中输入:export LC_ALL='zh_CN.utf8'



方法二：配置系统  



1. console终端乱码  

在/etc/profile文件的最后一行添加如下内容：  

export LC_ALL="zh_CN.GB18030"



2. xwindow终端乱码  

在/etc/sysconfig/i18n文件的最后一行添加如下内容：  

export LC_ALL="zh_CN.GB18030"



vi /etc/sysconfig/i18n  

将内容改为  

LANG="zh_CN.GB18030"

LANGUAGE="zh_CN.GB18030:zh_CN.GB2312:zh_CN"

SUPPORTED="zh_CN.GB18030:zh_CN:zh:en_US.UTF-8:en_US:en"

SYSFONT="lat0-sun16"

之后重启机器，这样中文在SSH,telnet终端就可以正常显示了。  



注意：若操作系统语言是英文，显示中文字符为乱码时



编辑/etc/sysconfig/i18n，修改为如下内容：  



LANG="en_US"

SUPPORTED="en_US.UTF-8:en_US:en"

SYSFONT="latarcyrheb-sun16"
