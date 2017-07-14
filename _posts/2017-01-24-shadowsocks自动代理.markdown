---
layout: "post"
title: "shadowsocks自动代理"
date: "2017-01-24 01:47"
tags:
- shadowsocks
comments: true
---

### 配置

安装Genepac

它可以自动生成的PAC文件

    sudo pip install genpac

新建shadowsocks文件夹存放pac文件

    mkdir ~/shadowsocks
    cd ~/shadowsocks
    
生成pac文件

    sudo genpac --proxy="SOCKS5 127.0.0.1:1080" -o autoproxy.pac --gfwlist-url="https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt"

设置系统网络代理为自动代理

    file:///home/suerte/shadowsocks/autoproxy.pac


url为shadowsocks里的pac文件
