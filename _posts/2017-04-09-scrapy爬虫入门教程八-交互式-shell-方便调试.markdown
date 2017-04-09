---
layout: "post"
title: "Scrapy爬虫入门教程八 交互式 shell 方便调试"
date: "2017-04-09 14:31"
tags:
- Scrapy
comments: true
---

# Scrapy爬虫入门教程八 交互式 shell 方便调试


**开发环境：**
`Python 3.6.0 版本` （当前最新）
`Scrapy 1.3.2 版本` （当前最新）

# Scrapy shell

Scrapy shell是一个交互式shell，您可以在此快速尝试和调试您的抓取代码，而无需运行爬虫程序。它用于测试数据提取代码，但实际上可以使用它来测试任何类型的代码，因为它也是一个常规的Python shell。

shell用于测试XPath或CSS表达式，并查看它们如何工作，以及他们从您要尝试抓取的网页中提取的数据。它允许您在编写爬虫时交互式测试表达式，而无需运行爬虫来测试每个更改。

一旦你熟悉了Scrapy shell，你会发现它是开发和调试你的爬虫的一个非常宝贵的工具。

## 配置shell

如果你安装了[IPython](http://ipython.org/)，Scrapy shell会使用它（而不是标准的Python控制台）。该IPython的控制台功能更强大，并提供智能自动完成和彩色输出，等等。

我们强烈建议您安装IPython，特别是如果你正在使用Unix系统（IPython擅长）。有关 详细信息，请参阅[IPython安装指南](http://ipython.org/install.html)。

Scrapy还支持[bpython](http://www.bpython-interpreter.org/)，并且将尝试在IPython 不可用的地方使用它。

通过scrapy的设置，您可以配置为使用中的任何一个 `ipython`，`bpython`或标准`python`外壳，安装无论哪个。这是通过设置`SCRAPY_PYTHON_SHELL`环境变量来完成的; 或通过在scrapy.cfg中定义它：

    [settings]
    shell = bpython

## 启动shell

要启动Scrapy shell，可以使用如下shell命令：

    scrapy shell <url>

其中，<url>是您要抓取的网址。

shell也适用于本地文件。如果你想玩一个网页的本地副本，这可以很方便。shell了解本地文件的以下语法：

    # UNIX-style
    scrapy shell ./path/to/file.html
    scrapy shell ../other/path/to/file.html
    scrapy shell /absolute/path/to/file.html

    # File URI
    scrapy shell file:///absolute/path/to/file.html

**注意**

> 当使用相对文件路径时，是显式的，并在它们前面./（或../相关时）。 将不会像一个人所期望的那样工作（这是设计，而不是一个错误）。scrapy shell index.html
> 因为shell喜欢文件URI上的HTTP URL，并且index.html在语法上类似example.com， shell会将其视为index.html域名并触发DNS查找错误：

    $ scrapy shell index.html
    [ ... scrapy shell starts ... ]
    [ ... traceback ... ]
    twisted.internet.error.DNSLookupError: DNS lookup failed:
    address 'index.html' not found: [Errno -5] No address associated with hostname.

shell将不会预先测试index.html 当前目录中是否存在调用的文件。再次，明确。

## 使用shell

Scrapy shell只是一个普通的Python控制台（或IPython控制台，如果你有它），为方便起见，它提供了一些额外的快捷方式功能。

### 可用快捷键

- `shelp()` - 打印有可用对象和快捷方式列表的帮助
*`fetch(url[, redirect=True])` - 从给定的URL获取新的响应，并相应地更新所有相关对象。你可以选择要求HTTP 3xx重定向，不要通过redirect=False
- `fetch(request)` - 从给定请求获取新响应，并相应地更新所有相关对象。
- `view(response)` - 在本地Web浏览器中打开给定的响应，以进行检查。这将向响应正文添加一个[<base>标记](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/base)，以便正确显示外部链接（如图片和样式表）。但请注意，这将在您的计算机中创建一个临时文件，不会自动删除。

### 可用Scrapy对象

Scrapy shell自动从下载的页面创建一些方便的对象，如Response对象和 Selector对象（对于HTML和XML内容）。

这些对象是：

- crawler- 当前Crawler对象。
- spider- 已知处理URL的Spider，或者Spider如果没有为当前URL找到的爬虫，则为 对象
- request- Request最后一个抓取页面的对象。您可以replace() 使用fetch 快捷方式或使用快捷方式获取新请求（而不离开shell）来修改此请求。
- response- 包含Response最后一个抓取页面的对象
- settings- 当前[Scrapy设置](http://scrapy.readthedocs.io/en/latest/topics/settings.html#topics-settings)

## shell会话的示例

下面是一个典型的shell会话示例，我们首先抓取 [http://scrapy.org页面，然后继续抓取https://reddit.com](http://scrapy.org%E9%A1%B5%E9%9D%A2%EF%BC%8C%E7%84%B6%E5%90%8E%E7%BB%A7%E7%BB%AD%E6%8A%93%E5%8F%96https://reddit.com) 页面。最后，我们将（Reddit）请求方法修改为POST并重新获取它获取错误。我们通过在Windows中键入Ctrl-D（在Unix系统中）或Ctrl-Z结束会话。

请记住，在这里提取的数据可能不一样，当你尝试它，因为那些网页不是静态的，可能已经改变了你测试这个。这个例子的唯一目的是让你熟悉Scrapy shell的工作原理。

首先，我们启动shell：
`scrapy shell 'http://scrapy.org' --nolog`

然后，shell获取URL（使用Scrapy下载器）并打印可用对象和有用的快捷方式列表（您会注意到这些行都以[s]前缀开头）：

    [s] Available Scrapy objects:
    [s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
    [s]   crawler    <scrapy.crawler.Crawler object at 0x7f07395dd690>
    [s]   item       {}
    [s]   request    <GET http://scrapy.org>
    [s]   response   <200 https://scrapy.org/>
    [s]   settings   <scrapy.settings.Settings object at 0x7f07395dd710>
    [s]   spider     <DefaultSpider 'default' at 0x7f0735891690>
    [s] Useful shortcuts:
    [s]   fetch(url[, redirect=True]) Fetch URL and update local objects (by default, redirects are followed)
    [s]   fetch(req)                  Fetch a scrapy.Request and update local objects
    [s]   shelp()           Shell help (print this help)
    [s]   view(response)    View response in a browser

    >>>

之后，我们可以开始使用对象：

    >>> response.xpath('//title/text()').extract_first()
    'Scrapy | A Fast and Powerful Scraping and Web Crawling Framework'

    >>> fetch("http://reddit.com")

    >>> response.xpath('//title/text()').extract()
    ['reddit: the front page of the internet']

    >>> request = request.replace(method="POST")

    >>> fetch(request)

    >>> response.status
    404

    >>> from pprint import pprint

    >>> pprint(response.headers)
    {'Accept-Ranges': ['bytes'],
     'Cache-Control': ['max-age=0, must-revalidate'],
     'Content-Type': ['text/html; charset=UTF-8'],
     'Date': ['Thu, 08 Dec 2016 16:21:19 GMT'],
     'Server': ['snooserv'],
     'Set-Cookie': ['loid=KqNLou0V9SKMX4qb4n; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',
                    'loidcreated=2016-12-08T16%3A21%3A19.445Z; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',
                    'loid=vi0ZVe4NkxNWdlH7r7; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',
                    'loidcreated=2016-12-08T16%3A21%3A19.459Z; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure'],
     'Vary': ['accept-encoding'],
     'Via': ['1.1 varnish'],
     'X-Cache': ['MISS'],
     'X-Cache-Hits': ['0'],
     'X-Content-Type-Options': ['nosniff'],
     'X-Frame-Options': ['SAMEORIGIN'],
     'X-Moose': ['majestic'],
     'X-Served-By': ['cache-cdg8730-CDG'],
     'X-Timer': ['S1481214079.394283,VS0,VE159'],
     'X-Ua-Compatible': ['IE=edge'],
     'X-Xss-Protection': ['1; mode=block']}
    >>>

## 从爬虫调用shell检查响应

有时候，你想检查在爬虫的某一点被处理的响应，如果只检查你期望的响应到达那里。

这可以通过使用该`scrapy.shell.inspect_response`功能来实现。

下面是一个如何从爬虫调用它的例子：

    import scrapy


    class MySpider(scrapy.Spider):
        name = "myspider"
        start_urls = [
            "http://example.com",
            "http://example.org",
            "http://example.net",
        ]

        def parse(self, response):
            # We want to inspect one specific response.
            if ".org" in response.url:
                from scrapy.shell import inspect_response
                inspect_response(response, self)

            # Rest of parsing code.

当你运行爬虫，你会得到类似的东西：

    2014-01-23 17:48:31-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.com> (referer: None)
    2014-01-23 17:48:31-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.org> (referer: None)
    [s] Available Scrapy objects:
    [s]   crawler    <scrapy.crawler.Crawler object at 0x1e16b50>
    ...

    >>> response.url
    'http://example.org'

然后，您可以检查提取代码是否正常工作：

    >>> response.xpath('//h1[@class="fn"]')
    []

不，不是。因此，您可以在Web浏览器中打开响应，看看它是否是您期望的响应：

    >>> view(response)
    True

最后，您按Ctrl-D（或Windows中的Ctrl-Z）退出外壳并继续抓取：

    >>> ^D
    2014-01-23 17:50:03-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.net> (referer: None)
    ...

请注意，您不能使用fetch此处的快捷方式，因为Scrapy引擎被shell阻止。然而，在你离开shell之后，爬虫会继续爬到它停止的地方，如上图所示。
