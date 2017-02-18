---
layout: "post"
title: "shadowinstaller"
date: "2017-02-11 00:04"
tags:
- shadowsocks
comments: true
---

# shadowsocks一键配置脚本
默认配置：
服务器端口：自己设定（如不设定，默认为6565）
客户端端口：1080
密码：自己设定（如不设定，默认为zhuio）
备注：脚本默认创建单用户配置文件，如需配置多用户，安装完毕后参照下面的教程 sample 手动修改配置文件后重启即可。

客户端下载：
[https://github.com/shadowsocks/shadowsocks-windows/releases](https://github.com/shadowsocks/shadowsocks-windows/releases)

使用方法：
使用root用户登录，运行以下命令：

    wget --no-check-certificate https://raw.githubusercontent.com/zhuio/shadowsocksinstaller/master/shadowsocks.sh
    chmod +x shadowsocks.sh
    ./shadowsocks.sh 2>&1 | tee shadowsocks.log

安装完成后，脚本提示如下：

    Congratulations, shadowsocks install completed!
    Your Server IP:your_server_ip
    Your Server Port:your_server_port
    Your Password:your_password
    Your Local IP:127.0.0.1
    Your Local Port:1080
    Your Encryption Method:aes-256-cfb
    Enjoy it!

卸载方法：
使用root用户登录，运行以下命令：

    ./shadowsocks.sh uninstall


配置文件路径：/etc/shadowsocks.json

    {
        "server":"0.0.0.0",
        "server_port":6565,
        "local_address":"127.0.0.1",
        "local_port":1080,
        "password":"yourpassword",
        "timeout":300,
        "method":"aes-256-cfb",
        "fast_open": false
    }

多用户多端口配置文件 ：
配置文件路径：/etc/shadowsocks.json

    {
        "server":"0.0.0.0",
        "local_address":"127.0.0.1",
        "local_port":1080,
        "port_password":{
             "8989":"password0",
             "9001":"password1",
             "9002":"password2",
             "9003":"password3",
             "9004":"password4"
        },
        "timeout":300,
        "method":"aes-256-cfb",
        "fast_open": false
    }

使用命令：
启动：/etc/init.d/shadowsocks start
停止：/etc/init.d/shadowsocks stop
重启：/etc/init.d/shadowsocks restart
状态：/etc/init.d/shadowsocks status
