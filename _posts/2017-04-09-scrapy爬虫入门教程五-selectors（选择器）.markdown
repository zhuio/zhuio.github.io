---
layout: "post"
title: "Scrapy爬虫入门教程五 Selectors（选择器）"
date: "2017-04-09 14:30"
tags:
- Scrapy
comments: true
---

# Scrapy爬虫入门教程五 Selectors（选择器）

**开发环境：**
`Python 3.6.0 版本` （当前最新）
`Scrapy 1.3.2 版本` （当前最新）

# Selectors（选择器）

当您抓取网页时，您需要执行的最常见任务是从HTML源中提取数据。有几个库可以实现这一点：

[BeautifulSoup](http://www.crummy.com/software/BeautifulSoup/)是Python程序员中非常流行的网络抓取库，它基于HTML代码的结构构建一个Python对象，并且处理相当糟糕的标记，但它有一个缺点：它很慢。
[lxml](http://lxml.de/)是一个XML解析库（它还解析HTML）与基于[ElementTree](https://docs.python.org/2/library/xml.etree.elementtree.html)的pythonic API 。（lxml不是Python标准库的一部分。）
Scrapy自带了提取数据的机制。它们称为选择器，因为它们“选择”由[XPath](https://www.w3.org/TR/xpath)或[CSS](https://www.w3.org/TR/selectors)表达式指定的HTML文档的某些部分。

XPath是用于选择XML文档中的节点的语言，其也可以与HTML一起使用。CSS是一种用于将样式应用于HTML文档的语言。它定义了选择器以将这些样式与特定的HTML元素相关联。

Scrapy选择器构建在lxml库之上，这意味着它们的速度和解析精度非常相似。

这个页面解释了选择器是如何工作的，并描述了他们的API是非常小和简单，不像lxml API是更大，因为 lxml库可以用于许多其他任务，除了选择标记文档。

有关[选择器 API](http://scrapy.readthedocs.io/en/latest/topics/selectors.html#topics-selectors-ref)的完整参考，请参阅 [选择器引用](http://scrapy.readthedocs.io/en/latest/topics/selectors.html#topics-selectors-ref)

## 使用选择器

### 构造选择器

Scrapy选择器是Selector通过传递文本或TextResponse 对象构造的类的实例。它根据输入类型自动选择最佳的解析规则（XML与HTML）：

    >>> from scrapy.selector import Selector
    >>> from scrapy.http import HtmlResponse

从文本构造：

    >>> body = '<html><body><span>good</span></body></html>'
    >>> Selector(text=body).xpath('//span/text()').extract()
    [u'good']

构建响应：

    >>> response = HtmlResponse(url='http://example.com', body=body)
    >>> Selector(response=response).xpath('//span/text()').extract()
    [u'good']

为了方便起见，响应对象在.selector属性上显示一个选择器，在可能的情况下使用此快捷键是完全正确的：

    >>> response.selector.xpath('//span/text()').extract()
    [u'good']

### 使用选择器

为了解释如何使用选择器，我们将使用Scrapy shell（提供交互式测试）和位于Scrapy文档服务器中的示例页面：

[http://doc.scrapy.org/en/latest/_static/selectors-sample1.html](http://doc.scrapy.org/en/latest/_static/selectors-sample1.html)
这里是它的HTML代码：

    <html>
     <head>
      <base href='http://example.com/' />
      <title>Example website</title>
     </head>
     <body>
      <div id='images'>
       <a href='image1.html'>Name: My image 1 <br />![](image1_thumb.jpg)</a>
       <a href='image2.html'>Name: My image 2 <br />![](image2_thumb.jpg)</a>
       <a href='image3.html'>Name: My image 3 <br />![](image3_thumb.jpg)</a>
       <a href='image4.html'>Name: My image 4 <br />![](image4_thumb.jpg)</a>
       <a href='image5.html'>Name: My image 5 <br />![](image5_thumb.jpg)</a>
      </div>
     </body>
    </html>

首先，让我们打开shell：
`scrapy shell http://doc.scrapy.org/en/latest/_static/selectors-sample1.html`
然后，在加载shell之后，您将有可用的响应作为response shell变量，以及其附加的选择器response.selector属性。

由于我们处理HTML，选择器将自动使用HTML解析器。

因此，通过查看该页面的[HTML代码](http://scrapy.readthedocs.io/en/latest/topics/selectors.html#topics-selectors-htmlcode)，让我们构造一个XPath来选择标题标签中的文本：

    >>> response.selector.xpath('//title/text()')
    [<Selector (text) xpath=//title/text()>]

使用XPath和CSS查询响应非常普遍，响应包括两个方便的快捷键：response.xpath()和response.css()：

    >>> response.xpath('//title/text()')
    [<Selector (text) xpath=//title/text()>]
    >>> response.css('title::text')
    [<Selector (text) xpath=//title/text()>]

正如你所看到的，.xpath()而.css()方法返回一个 SelectorList实例，它是新的选择列表。此API可用于快速选择嵌套数据：

    >>> response.css('img').xpath('@src').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']

要实际提取文本数据，必须调用选择器.extract() 方法，如下所示：

    >>> response.xpath('//title/text()').extract()
    [u'Example website']

如果只想提取第一个匹配的元素，可以调用选择器 .extract_first()

    >>> response.xpath('//div[@id="images"]/a/text()').extract_first()
    u'Name: My image 1 '

None如果没有找到元素则返回：

    >>> response.xpath('//div[@id="not-exists"]/text()').extract_first() is None
    True

可以提供默认返回值作为参数，而不是使用None：

    >>> response.xpath('//div[@id="not-exists"]/text()').extract_first(default='not-found')
    'not-found'

请注意，CSS选择器可以使用CSS3伪元素选择文本或属性节点：

    >>> response.css('title::text').extract()
    [u'Example website']

现在我们要获取基本URL和一些图像链接：

    >>> response.xpath('//base/@href').extract()
    [u'http://example.com/']

    >>> response.css('base::attr(href)').extract()
    [u'http://example.com/']

    >>> response.xpath('//a[contains(@href, "image")]/@href').extract()
    [u'image1.html',
     u'image2.html',
     u'image3.html',
     u'image4.html',
     u'image5.html']

    >>> response.css('a[href*=image]::attr(href)').extract()
    [u'image1.html',
     u'image2.html',
     u'image3.html',
     u'image4.html',
     u'image5.html']

    >>> response.xpath('//a[contains(@href, "image")]/img/@src').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']

    >>> response.css('a[href*=image] img::attr(src)').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']

### 嵌套选择器

选择方法（.xpath()或.css()）返回相同类型的选择器的列表，因此您也可以调用这些选择器的选择方法。这里有一个例子：

    >>> links = response.xpath('//a[contains(@href, "image")]')
    >>> links.extract()
    [u'<a href="image1.html">Name: My image 1 <br>![](image1_thumb.jpg)</a>',
     u'<a href="image2.html">Name: My image 2 <br>![](image2_thumb.jpg)</a>',
     u'<a href="image3.html">Name: My image 3 <br>![](image3_thumb.jpg)</a>',
     u'<a href="image4.html">Name: My image 4 <br>![](image4_thumb.jpg)</a>',
     u'<a href="image5.html">Name: My image 5 <br>![](image5_thumb.jpg)</a>']

    >>> for index, link in enumerate(links):
    ...     args = (index, link.xpath('@href').extract(), link.xpath('img/@src').extract())
    ...     print 'Link number %d points to url %s and image %s' % args

    Link number 0 points to url [u'image1.html'] and image [u'image1_thumb.jpg']
    Link number 1 points to url [u'image2.html'] and image [u'image2_thumb.jpg']
    Link number 2 points to url [u'image3.html'] and image [u'image3_thumb.jpg']
    Link number 3 points to url [u'image4.html'] and image [u'image4_thumb.jpg']
    Link number 4 points to url [u'image5.html'] and image [u'image5_thumb.jpg']

#### 使用带有正则表达式的选择器

Selector也有一种.re()使用正则表达式提取数据的方法。但是，不同于使用 .xpath()或 .css()methods，.re()返回一个unicode字符串列表。所以你不能构造嵌套.re()调用。

以下是用于从上面的HTML代码中提取图片名称的示例：

    >>> response.xpath('//a[contains(@href, "image")]/text()').re(r'Name:\s*(.*)')
    [u'My image 1',
     u'My image 2',
     u'My image 3',
     u'My image 4',
     u'My image 5']

这里有一个额外的辅助往复.extract_first()进行.re()，命名.re_first()。使用它只提取第一个匹配的字符串：

    >>> response.xpath('//a[contains(@href, "image")]/text()').re_first(r'Name:\s*(.*)')
    u'My image 1'

#### 使用相对XPath

请记住，如果您嵌套选择器并使用以XPath开头的XPath /，该XPath将是绝对的文档，而不是相对于 Selector您调用它。

例如，假设要提取

元素中的所有<div> 元素。首先，你会得到所有的<div>元素：
`>>> divs = response.xpath('//div')`

首先，你可能会使用下面的方法，这是错误的，因为它实际上

从文档中提取所有元素，而不仅仅是那些内部<div>元素：

    >>> for p in divs.xpath('//p'):  # this is wrong - gets all <p> from the whole document
    ...     print p.extract()

这是正确的方式（注意点前面的.//pXPath 的点）：

    >>> for p in divs.xpath('.//p'):  # extracts all <p> inside
    ...     print p.extract()

另一个常见的情况是提取所有直接的\

孩子：

    >>> for p in divs.xpath('p'):
    ...     print p.extract()

有关相对XPath的更多详细信息，请参阅XPath规范中的[位置路径部分](https://www.w3.org/TR/xpath#location-paths)。

#### XPath表达式中的变量

XPath允许您使用`$somevariable`语法来引用XPath表达式中的变量。这在某种程度上类似于SQL世界中的参数化查询或预准备语句，您在查询中使用占位符替换一些参数，`?`然后用查询传递的值替换。

这里有一个例子来匹配元素基于其“id”属性值，没有硬编码它（如前所示）：

    >>> # `$val` used in the expression, a `val` argument needs to be passed
    >>> response.xpath('//div[@id=$val]/a/text()', val='images').extract_first()
    u'Name: My image 1 '

这里是另一个例子，找到一个`<div>`标签的“id” 属性包含五个`<a>`孩子（这里我们传递的值`5`作为一个整数）：

    >>> response.xpath('//div[count(a)=$cnt]/@id', cnt=5).extract_first()
    u'images'

所有变量引用在调用时必须有一个绑定值`.xpath()`（否则你会得到一个异常）。这是通过传递必要的命名参数。`ValueError: XPath error:`

[parsel](https://parsel.readthedocs.io/)是为Scrapy选择器提供动力的库，有关于[XPath变量](https://parsel.readthedocs.io/en/latest/usage.html#variables-in-xpath-expressions)的更多详细信息和示例。

#### 使用EXSLT扩展

在构建在[lxml](http://lxml.de/)之上时，Scrapy选择器还支持一些[EXSLT](http://exslt.org/)扩展，并带有这些预先注册的命名空间以在XPath表达式中使用：
字首命名空间<span class="Apple-tab-span" style="white-space:pre"></span>用法回覆[http://exslt.org/regular-expressions](http://exslt.org/regular-expressions)[正则表达式](http://exslt.org/regexp/index.html)组[http://exslt.org/sets](http://exslt.org/sets)[设置操作](http://exslt.org/set/index.html)
##### 正则表达式

test()例如，当XPath starts-with()或者contains()不足时，该函数可以证明是非常有用的 。

示例选择列表项中的链接，其中“类”属性以数字结尾：

    >>> from scrapy import Selector
    >>> doc = """
    ... <div>
    ...     <ul>
    ...         <li class="item-0"><a href="link1.html">first item</a></li>
    ...         <li class="item-1"><a href="link2.html">second item</a></li>
    ...         <li class="item-inactive"><a href="link3.html">third item</a></li>
    ...         <li class="item-1"><a href="link4.html">fourth item</a></li>
    ...         <li class="item-0"><a href="link5.html">fifth item</a></li>
    ...     </ul>
    ... </div>
    ... """
    >>> sel = Selector(text=doc, type="html")
    >>> sel.xpath('//li//@href').extract()
    [u'link1.html', u'link2.html', u'link3.html', u'link4.html', u'link5.html']
    >>> sel.xpath('//li[re:test(@class, "item-\d$")]//@href').extract()
    [u'link1.html', u'link2.html', u'link4.html', u'link5.html']
    >>>

**警告**

> C库libxslt本身不支持EXSLT正则表达式，所以lxml的实现使用钩子到Python的re模块。因此，在XPath表达式中使用regexp函数可能会增加小的性能损失。

##### 设置操作

这些可以方便地在提取文本元素之前排除文档树的部分。

使用项目范围组和相应的itemprops组提取微数据（从[http://schema.org/Product](http://schema.org/Product)中提取的示例内容）示例：

    >>> doc = """
    ... <div itemscope itemtype="http://schema.org/Product">
    ...   <span itemprop="name">Kenmore White 17" Microwave</span>
    ...   ![](kenmore-microwave-17in.jpg)
    ...   <div itemprop="aggregateRating"
    ...     itemscope itemtype="http://schema.org/AggregateRating">
    ...    Rated <span itemprop="ratingValue">3.5</span>/5
    ...    based on <span itemprop="reviewCount">11</span> customer reviews
    ...   </div>
    ...
    ...   <div itemprop="offers" itemscope itemtype="http://schema.org/Offer">
    ...     <span itemprop="price">$55.00</span>
    ...     <link itemprop="availability" href="http://schema.org/InStock" />In stock
    ...   </div>
    ...
    ...   Product description:
    ...   <span itemprop="description">0.7 cubic feet countertop microwave.
    ...   Has six preset cooking categories and convenience features like
    ...   Add-A-Minute and Child Lock.</span>
    ...
    ...   Customer reviews:
    ...
    ...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
    ...     <span itemprop="name">Not a happy camper</span> -
    ...     by <span itemprop="author">Ellie</span>,
    ...     <meta itemprop="datePublished" content="2011-04-01">April 1, 2011
    ...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
    ...       <meta itemprop="worstRating" content = "1">
    ...       <span itemprop="ratingValue">1</span>/
    ...       <span itemprop="bestRating">5</span>stars
    ...     </div>
    ...     <span itemprop="description">The lamp burned out and now I have to replace
    ...     it. </span>
    ...   </div>
    ...
    ...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
    ...     <span itemprop="name">Value purchase</span> -
    ...     by <span itemprop="author">Lucas</span>,
    ...     <meta itemprop="datePublished" content="2011-03-25">March 25, 2011
    ...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
    ...       <meta itemprop="worstRating" content = "1"/>
    ...       <span itemprop="ratingValue">4</span>/
    ...       <span itemprop="bestRating">5</span>stars
    ...     </div>
    ...     <span itemprop="description">Great microwave for the price. It is small and
    ...     fits in my apartment.</span>
    ...   </div>
    ...   ...
    ... </div>
    ... """
    >>> sel = Selector(text=doc, type="html")
    >>> for scope in sel.xpath('//div[@itemscope]'):
    ...     print "current scope:", scope.xpath('@itemtype').extract()
    ...     props = scope.xpath('''
    ...                 set:difference(./descendant::*/@itemprop,
    ...                                .//*[@itemscope]/*/@itemprop)''')
    ...     print "    properties:", props.extract()
    ...     print

    current scope: [u'http://schema.org/Product']
        properties: [u'name', u'aggregateRating', u'offers', u'description', u'review', u'review']

    current scope: [u'http://schema.org/AggregateRating']
        properties: [u'ratingValue', u'reviewCount']

    current scope: [u'http://schema.org/Offer']
        properties: [u'price', u'availability']

    current scope: [u'http://schema.org/Review']
        properties: [u'name', u'author', u'datePublished', u'reviewRating', u'description']

    current scope: [u'http://schema.org/Rating']
        properties: [u'worstRating', u'ratingValue', u'bestRating']

    current scope: [u'http://schema.org/Review']
        properties: [u'name', u'author', u'datePublished', u'reviewRating', u'description']

    current scope: [u'http://schema.org/Rating']
        properties: [u'worstRating', u'ratingValue', u'bestRating']

    >>>

这里我们先迭代`itemscope`元素，对于每一个元素，我们寻找所有`itemprops`元素，并排除那些在另一个元素内部的元素`itemscope`。

#### 一些XPath提示

这里有一些提示，你可能会发现有用的使用XPath与Scrapy选择器，基于这个帖子从ScrapingHub的博客。如果你不太熟悉XPath，你可能想先看看这个XPath教程。

##### 在条件中使用文本节点

当您需要使用文本内容作为[XPath字符串函数的](https://www.w3.org/TR/xpath/#section-String-Functions)参数时，请避免使用`.//text()`和使用`.`。

这是因为表达式`.//text()`产生一组文本元素 - 一个节点集。当一个节点集被转换为一个字符串，当它作为参数传递给一个字符串函数，如`contains()`or `starts-with()`时，会导致第一个元素的文本。

例：

    >>> from scrapy import Selector
    >>> sel = Selector(text='<a href="#">Click here to go to the <strong>Next Page</strong></a>')

将节点集转换为字符串：

    >>> sel.xpath('//a//text()').extract() # take a peek at the node-set
    [u'Click here to go to the ', u'Next Page']
    >>> sel.xpath("string(//a[1]//text())").extract() # convert it to string
    [u'Click here to go to the ']

一个节点转换为字符串，但是，拼文本本身及其所有的后代：

    >>> sel.xpath("//a[1]").extract() # select the first node
    [u'<a href="#">Click here to go to the <strong>Next Page</strong></a>']
    >>> sel.xpath("string(//a[1])").extract() # convert it to string
    [u'Click here to go to the Next Page']

所以，`.//text()`在这种情况下使用节点集不会选择任何东西：

    >>> sel.xpath("//a[contains(.//text(), 'Next Page')]").extract()
    []

但是使用的`.`意思是节点，工作原理：

    >>> sel.xpath("//a[contains(., 'Next Page')]").extract()
    [u'<a href="#">Click here to go to the <strong>Next Page</strong></a>']

**注意// node [1]和（// node）之间的区别[1]**
`//node[1]`选择在它们各自的父亲下首先出现的所有节点。

`(//node)[1]` 选择文档中的所有节点，然后仅获取其中的第一个。

例：

    >>> from scrapy import Selector
    >>> sel = Selector(text="""
    ....:     <ul class="list">
    ....:         <li>1</li>
    ....:         <li>2</li>
    ....:         <li>3</li>
    ....:     </ul>
    ....:     <ul class="list">
    ....:         <li>4</li>
    ....:         <li>5</li>
    ....:         <li>6</li>
    ....:     </ul>""")
    >>> xp = lambda x: sel.xpath(x).extract()

这将获得所有第一个`<li>` 元素，无论它是它的父：

    >>> xp("//li[1]")
    [u'<li>1</li>', u'<li>4</li>']

这`<li>` 是整个文档的第一个元素：

    >>> xp("(//li)[1]")
    [u'<li>1</li>']

这将获得 父`<li>` 下的所有第一个元素`<ul>`：

    >>> xp("//ul/li[1]")
    [u'<li>1</li>', u'<li>4</li>']

这将获得 整个文档中父级`<li>` 下的第一个元素`<ul>`：

    >>> xp("(//ul/li)[1]")
    [u'<li>1</li>']

##### 当按类查询时，请考虑使用CSS

因为一个元素可以包含多个CSS类，所以XPath选择元素的方法是相当冗长：

    *[contains(concat(' ', normalize-space(@class), ' '), ' someclass ')]

如果你使用`@class='someclass'`你可能最终缺少有其他类的元素，如果你只是使用补偿，你可能会得到更多的你想要的元素，如果他们有一个不同的类名共享字符串。`contains(@class, 'someclass')someclass`

事实证明，Scrapy选择器允许你链接选择器，所以大多数时候你可以使用CSS选择类，然后在需要时切换到XPath：

    >>> from scrapy import Selector
    >>> sel = Selector(text='<div class="hero shout"><time datetime="2014-07-23 19:00">Special date</time></div>')
    >>> sel.css('.shout').xpath('./time/@datetime').extract()
    [u'2014-07-23 19:00']

这比使用上面显示的详细XPath技巧更清晰。只要记住.在后面的XPath表达式中使用。

## 内置选择器参考

    class scrapy.selector.Selector(response=None, text=None, type=None)

一个实例`Selector`是一个包装器响应来选择其内容的某些部分。

response是一个`HtmlResponse`或一个`XmlResponse`将被用于选择和提取的数据对象。

`text`是一个`unicode`字符串或`utf-8`编码的文本，当一个 `response`不可用时。使用`text`和`response`一起是未定义的行为。

`type`定义选择器类型，它可以是`"html"`，`"xml"`或`None（默认）`。

如果`type`是`None`，选择器将根据`response`类型（见下文）自动选择最佳类型，或者默认`"html"`情况下与选项一起使用`text`。

如果`type`是`None`和`response`传递，选择器类型从响应类型推断如下：

- `"html"`对于HtmlResponse类型
- `"xml"`对于XmlResponse类型
- `"html"`为任何其他

否则，如果`type`设置，选择器类型将被强制，并且不会发生检测。

**xpath（查询）**
查找与`xpath`匹配的节点`query`，并将结果作为 `SelectorList`实例将所有元素展平。列表元素也实现`Selector`接口。

`query` 是一个包含要应用的XPATH查询的字符串。

**注意**

> 为了方便起见，这种方法可以称为 response.xpath()

**css（查询）**
应用给定的CSS选择器并返回一个SelectorList实例。

query 是一个包含要应用的CSS选择器的字符串。

在后台，CSS查询使用cssselect库和run .xpath()方法转换为XPath查询 。

**注意**

> 为了方便起见，该方法可以称为 response.css()

**extract（）**
序列化并返回匹配的节点作为unicode字符串列表。编码内容的百分比未引用。

**re（regex）**
应用给定的正则表达式并返回一个包含匹配项的unicode字符串的列表。

`regex`可以是编译的正则表达式或将被编译为正则表达式的字符串 `re.compile(regex)`

**注意**

> 注意，re()和re_first()解码HTML实体（除\<和\&）。

**register_namespace（prefix，uri）**
注册在此使用的给定命名空间Selector。如果不注册命名空间，则无法从非标准命名空间中选择或提取数据。参见下面的例子。

**remove_namespaces（）**
删除所有命名空间，允许使用无命名空间的xpaths遍历文档。参见下面的例子。

****nonzero**（）**
返回True如果有选择或任何实际的内容False 除外。换句话说，a的布尔值Selector由它选择的内容给出。

### SelectorList对象

`class scrapy.selector.SelectorList`

本SelectorList类是内置的一个子list 类，它提供了几个方法。

**xpath（查询）**
调用.xpath()此列表中每个元素的方法，并将其结果作为另一个返回SelectorList。

query 是同一个参数 Selector.xpath()

**css（查询）**
调用.css()此列表中每个元素的方法，并将其结果作为另一个返回SelectorList。

query 是同一个参数 Selector.css()

**extract（）**
调用.extract()此列表中每个元素的方法，并将其结果作为unicode字符串列表返回展平。

**re（）**
调用.re()此列表中每个元素的方法，并将其结果作为unicode字符串列表返回展平。

****nonzero**（）**
如果列表不为空，则返回True，否则返回False。

#### HTML响应的选择器示例

这里有几个Selector例子来说明几个概念。在所有情况下，我们假设已经Selector实例化了一个HtmlResponse对象，如下：

    sel = Selector(html_response)

1. `<h1>`从HTML响应主体中选择所有元素，返回Selector对象列表 （即SelectorList对象）：

    sel.xpath("//h1")

1. `<h1>`从HTML响应正文中提取所有元素的文本，返回unicode字符串

    sel.xpath("//h1").extract()         # this includes the h1 tag
    sel.xpath("//h1/text()").extract()  # this excludes the h1 tag

1. 迭代所有`<p>`标签并打印其类属性：

    for node in sel.xpath("//p"):
        print node.xpath("@class").extract()

#### XML响应的选择器示例

这里有几个例子来说明几个概念。在这两种情况下，我们假设已经Selector实例化了一个 XmlResponse对象，像这样：

    sel = Selector(xml_response)

1. <product>从XML响应主体中选择所有元素，返回Selector对象列表（即SelectorList对象）：

    sel.xpath("//product")

1. 从需要注册命名空间的[Google Base XML Feed](https://support.google.com/merchants/answer/160589?hl=en&amp;ref_topic=2473799)中提取所有价格：

    sel.register_namespace("g", "http://base.google.com/ns/1.0")
    sel.xpath("//g:price").extract()

#### 删除名称空间

当处理抓取项目时，通常很方便地完全删除命名空间，只需处理元素名称，编写更简单/方便的XPath。你可以使用的 Selector.remove_namespaces()方法。

让我们展示一个例子，用GitHub博客atom feed来说明这一点。

首先，我们打开shell和我们想要抓取的url：

    $ scrapy shell https://github.com/blog.atom

一旦在shell中，我们可以尝试选择所有<link>对象，并看到它不工作（因为Atom XML命名空间模糊了这些节点）：

    >>> response.xpath("//link")
    []

但是一旦我们调用该Selector.remove_namespaces()方法，所有节点都可以直接通过他们的名字访问：

    >>> response.selector.remove_namespaces()
    >>> response.xpath("//link")
    [<Selector xpath='//link' data=u'<link xmlns="http://www.w3.org/2005/Atom'>,
     <Selector xpath='//link' data=u'<link xmlns="http://www.w3.org/2005/Atom'>,
     ...

如果你想知道为什么默认情况下不调用命名空间删除过程，而不是手动调用它，这是因为两个原因，按照相关性的顺序：

1. 删除命名空间需要迭代和修改文档中的所有节点，这对于Scrapy爬取的所有文档来说是一个相当昂贵的操作
2. 可能有一些情况下，实际上需要使用命名空间，以防某些元素名称在命名空间之间冲突。这些情况非常罕见。
