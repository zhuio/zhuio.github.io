---
layout: "post"
title: "Scrapy爬虫入门教程十二 Link Extractors（链接提取器）"
date: "2017-04-09 14:34"
tags:
- Scrapy
comments: true
---

# Scrapy爬虫入门教程十二 Link Extractors（链接提取器）


**开发环境：**
`Python 3.6.0 版本` （当前最新）
`Scrapy 1.3.2 版本` （当前最新）

# 链接提取器

链接提取器是其唯一目的是从`scrapy.http.Response`最终将跟随的网页（对象）提取链接的对象。

有Scrapy，但你可以创建自己的自定义链接提取器，以满足您的需求通​​过实现一个简单的界面。`scrapy.linkextractors import LinkExtractor`

每个链接提取器唯一的公共方法是`extract_links`接收一个Response对象并返回一个`scrapy.link.Link`对象列表。链接提取器意在被实例化一次，并且它们的`extract_links`方法被调用几次，具有不同的响应以提取跟随的链接。

链接提取程序`CrawlSpider` 通过一组规则在类中使用（可以在Scrapy中使用），但是您也可以在爬虫中使用它，即使不从其中`CrawlSpider`提取子类 ，因为其目的非常简单：提取链接。

## 内置链接提取器参考

`scrapy.linkextractors`模块中提供了与Scrapy捆绑在一起的链接提取器类 。

默认的链接提取器是`LinkExtractor`，它是相同的 `LxmlLinkExtractor`：

    from scrapy.linkextractors import LinkExtractor

以前的Scrapy版本中曾经有过其他链接提取器类，但现在已经过时了。

### LxmlLinkExtractor

    class scrapy.linkextractors.lxmlhtml.LxmlLinkExtractor(allow=(), deny=(), allow_domains=(), deny_domains=(), deny_extensions=None, restrict_xpaths=(), restrict_css=(), tags=('a', 'area'), attrs=('href', ), canonicalize=True, unique=True, process_value=None)

LxmlLinkExtractor是推荐的链接提取器与方便的过滤选项。它使用lxml的强大的HTMLParser实现。

**参数：    **

- allow（正则表达式（或的列表）） - 一个单一的正则表达式（或正则表达式列表），（绝对）urls必须匹配才能提取。如果没有给出（或为空），它将匹配所有链接。
- deny（正则表达式或正则表达式列表） - 一个正则表达式（或正则表达式列表），（绝对）urls必须匹配才能排除（即不提取）。它优先于allow参数。如果没有给出（或为空），它不会排除任何链接。
- allow_domains（str或list） - 单个值或包含将被考虑用于提取链接的域的字符串列表
- deny_domains（str或list） - 单个值或包含不会被考虑用于提取链接的域的字符串列表
- deny_extensions（list） - 包含在提取链接时应该忽略的扩展的单个值或字符串列表。如果没有给出，它将默认为IGNORED_EXTENSIONS在[scrapy.linkextractors](https://github.com/scrapy/scrapy/blob/master/scrapy/linkextractors/__init__.py)包中定义的 列表 。
- restrict_xpaths（str或list） - 是一个XPath（或XPath的列表），它定义响应中应从中提取链接的区域。如果给出，只有那些XPath选择的文本将被扫描链接。参见下面的例子。
- restrict_css（str或list） - 一个CSS选择器（或选择器列表），用于定义响应中应提取链接的区域。有相同的行为restrict_xpaths。
标签（str或list） - 标签或在提取链接时要考虑的标签列表。默认为。('a', 'area')
- attrs（list） - 在查找要提取的链接时应该考虑的属性或属性列表（仅适用于参数中指定的那些标签tags ）。默认为('href',)
- canonicalize（boolean） - 规范化每个提取的url（使用w3lib.url.canonicalize_url）。默认为True。
- unique（boolean） - 是否应对提取的链接应用重复过滤。
- process_value（callable） -
接收从标签提取的每个值和扫描的属性并且可以修改值并返回新值的函数，或者返回None以完全忽略链接。如果没有给出，process_value默认为。lambda x: x

例如，要从此代码中提取链接：

    <a href="javascript:goToPage('../other/page.html'); return false">Link text</a>

您可以使用以下功能process_value：

    def process_value(value):
        m = re.search("javascript:goToPage\('(.*?)'", value)
        if m:
            return m.group(1)
