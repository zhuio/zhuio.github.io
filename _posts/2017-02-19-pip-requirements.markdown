---
layout: "post"
title: "pip requirements"
date: "2017-02-19 18:32"
tags:
- python
- pip
comments: true
---

# 如何自动生成和安装requirements.txt依赖

在查看别人的Python项目时，经常会看到一个`requirements.txt`文件，里面记录了当前程序的所有依赖包及其精确版本号。这个文件有点类似与Rails的`Gemfile`。其作用是用来在另一台PC上重新构建项目所需要的运行环境依赖。

requirements.txt可以通过`pip`命令自动生成和安装

### 生成requirements.txt文件

`pip freeze > requirements.txt`

### 安装requirements.txt依赖

`pip install -r requirements.txt`
