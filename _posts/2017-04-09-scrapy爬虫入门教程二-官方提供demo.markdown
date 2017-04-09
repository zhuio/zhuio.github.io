---
layout: "post"
title: "Scrapy爬虫入门教程二 官方提供Demo"
date: "2017-04-09 11:54"
tags:
- Scrapy
comments: true
---

# Scrapy爬虫入门教程二 官方提供Demo


**开发环境：**
`Python 3.6.0 版本` （当前最新）
`Scrapy 1.3.2 版本` （当前最新）

今天研究下官方给出的案例，大家可以多看看，多模仿模仿。

### 例子

最好的学习方法是使用示例，Scrapy也不例外。因此，有一个名为quotesbot的 Scrapy项目示例，请访问[https://github.com/scrapy/quotesbot](https://github.com/scrapy/quotesbot)，一个使用CSS选择器，另一个使用XPath表达式，此项目仅用于教育目的。

#### 提取的数据

提取的数据看起来像这个示例：

    {
        'author': 'Douglas Adams',
        'text': '“I may not have gone where I intended to go, but I think I ...”',
        'tags': ['life', 'navigation']
    }

---

#### 爬虫

此项目包含两个爬虫，您可以使用list 命令列出它们：

$ scrapy list
toscrape-css
toscrape-xpath

两个爬虫都从同一网站提取相同的数据，但toscrape-css 使用CSS选择器，而toscrape-xpath使用XPath表达式。

---

### 运行爬虫

您可以使用scrapy crawl命令运行爬虫，如：
`$ scrapy crawl toscrape-css`

如果要将已抓取的数据保存到文件，可以传递-o选项：
`$ scrapy crawl toscrape-css -o quotes.json`

---
