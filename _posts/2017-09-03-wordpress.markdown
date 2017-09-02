---
layout: "post"
title: "wordpress"
date: "2017-09-03 05:10"
tags:
- wordpress
comments: true
---
## 1,首先当然是安装LAMP环境了。

输入：

    sudo apt-get install lamp-server^


## 2,安装过程中，需要设置数据库的root密码。

## 3,安装完后，在浏览器中打开localhost验证安装是否正确。

## 4,现在设置wordpress的目录了。

    cd /etc/apache2/sites-available

    sudo cp 000-default.conf mysite.conf

    sudo vim mysite.comf


## 5,修改DocumentRoot 为你的wordpress目录

修改后，要更新apache2

输入：

    sudo a2dissite 000-default.conf && sudo a2ensite mysite.conf


## 6,更新后需要重启apache2

    sudo /etc/init.d/apache2 restart

## 7,现在要创建数据库用户。

输入：

    mysql -u root -p

输入：

    mysql> CREATE DATABASE yypwordpress;

    mysql> CREATE USER wordpressuser;

    mysql> SET PASSWORD FOR wordpressuser= PASSWORD(“lepassword”);

    mysql> GRANT ALL PRIVILEGES ON yypwordpress.* TO wordpressuser IDENTIFIED BY ‘lepassword′;

注意每句话后面有分号。

每执行一句要确保又OK的提示。


## 8,现在可以设置wordpress了。
