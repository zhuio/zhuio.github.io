---
layout: "post"
title: "shadowsocks锐速"
date: "2017-02-18 21:35"
tags:
- shadowsocks
comments: true
---


## 更换内核

    rpm -ivh http://soft.91yun.org/ISO/Linux/CentOS/kernel/kernel-3.10.0-229.1.2.el7.x86_64.rpm --force

## 锐速加速

查看[linux支持内核列表](https://www.91yun.org/wp-content/plugins/91yun-serverspeeder/systemlist.html)**

使用`uname -a`命令来查询内核版本，例如返回的是`Linux ss 3.8.0-35-generic`，`3.8.0-35-generic`就是内核版本

    这里顺便说一下linode VPS仅支持debian7系统，在选择内核时请选择3.14.5-x86_64-linode42版本，这个版本才能安装锐速。

然后检查你的VPS是什么虚拟化技术，锐速不支持OpenVZ的（而大部分很便宜的VPS都是OpenVZ）

我们只需要安装**vitr-what**就能知道VPS的虚拟化技术是什么了。

**Cent OS 系统：**

    yum install virt-what -y


**Debian/Ubuntu 系统：**

    apt-get install virt-what -y


安装后执行下面这个命令，

    virt-what


运行后会显示你的VPS虚拟化技术，如果不是OpenVZ，那么可以继续下面的安装步骤了。

## 安装开心版锐速

确定自己的内核版本在支持列表里，并且虚拟化技术非OpenVZ，就可以使用以下命令一键安装了。

为了备份和存档，这里我收集了几个锐速开心版的一键安装脚本，**大家优先使用第一个脚本！**

    wget -N --no-check-certificate https://raw.githubusercontent.com/91yun/serverspeeder/master/serverspeeder-all.sh && bash serverspeeder-all.sh
    # 注意！！建议使用第一个代码来安装，下面的几个都是备用的，功能都一样，不需要重复执行，只需要执行第一行代码就行了，每一行代码都是独立的。
    ————
    curl -s http://f.ylnote.com/f/install -o install;chmod 775 install;./install
    wget -O - http://file.idc.wiki/get.php?serverSpeeder | bash && bash serverSpeeder_setup.sh
    ————
    # 上面三个不区分系统。下面这四个区分系统
    ————
    # Debian7 64位
    wget o0o.re/rs&&sh rs
    # Centos6 64位
    wget o0o.re/rscentos&&sh rscentos
    # centos7 64位
    wget o0o.re/rscentos7&&sh rscentos7
    # Ubuntu14 64位
    wget o0o.re/rsu&&sh rsu


脚本会自动检测你的VPS是否可以按照开心版锐速，如果可以就会提示你参数或者直接安装完成，参数设置直接回车默认即可。

最后两项输入`y`开机自动启动锐速，`y`立刻启动锐速。（这个参数设置一般情况下都是不会出现的。）

![](https://doub.io/wp-content/plugins/wp-images-lazy-loading/images/grey.gif)![](https://img.mlkxs.com/ruisu-jc1.1.jpg?imageView2/1/w/2000/q/100|watermark/1/image/aHR0cDovLzd4ajh0NC5jb20xLnowLmdsYi5jbG91ZGRuLmNvbS9zaHVpeWluLnBuZw==/dissolve/80/gravity/SouthEast/dx/10/dy/10)

## 卸载开心版锐速

    chattr -i /serverspeeder/etc/apx* && /serverspeeder/bin/serverSpeeder.sh uninstall -f


### 锐速开心版功能：

1. 如果内核完全匹配就会自动下载安装。
2. 如果没有完全匹配的内核，会在界面提示可选内核，可以手动选个最接近的尝试
3. 自动下载授权文件
4. 自动修改配置文件
5. 已chattr +i /serverspeeder/etc/apx*禁止修改配置文件，可以不用加hosts了
6. 目前只支持CentOS，ubuntu和debian。如果有其他系统支持，可以到http://www.91yun.org/serverspeeder91yun手动下载其他系统的安装包

## 开启高级算法

    Tip：开心版锐速默认都启动高级算法了，所以可以忽略下面的步骤。

一些KVM/XEN的VPS上支持高级算法，所以我们可以修改锐速的3个参数来开启，`vi /serverspeeder/etc/config`

    rsc="1" #RSC网卡驱动模式  
    gso="1"
    maxmode="1" #最大传输模式



### 使用命令

    #重启锐速
    /serverspeeder/bin/serverSpeeder.sh restart
    #启动锐速
    /serverspeeder/bin/serverSpeeder.sh start
    #停止锐速
    /serverspeeder/bin/serverSpeeder.sh stop
    #查看锐速运行情况
    /serverspeeder/bin/serverSpeeder.sh status


### 删除锐速

    #脚本删除，如果不行就用下面的强制删除
    ./serverSpeederInstaller.sh uninstall
    强制删除
    rm -rf /serverspeeder


---


第一个开心版锐速脚本来自：[https://www.91yun.org/archives/683](https://www.91yun.org/archives/683)
