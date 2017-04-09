---
layout: "post"
title: "Scrapy爬虫入门教程四 Spider（爬虫）"
date: "2017-04-09 14:29"
tags:
- Scrapy
comments: true
---

# Scrapy爬虫入门教程四 Spider（爬虫）


**开发环境：**
`Python 3.6.0 版本` （当前最新）
`Scrapy 1.3.2 版本` （当前最新）

# Spider

爬虫是定义如何抓取某个网站（或一组网站）的类，包括如何执行抓取（即关注链接）以及如何从其网页中提取结构化数据（即抓取项目）。换句话说，Spider是您定义用于为特定网站（或在某些情况下，一组网站）抓取和解析网页的自定义行为的位置。

对于爬虫，循环经历这样的事情：

1.
您首先生成用于抓取第一个URL的初始请求，然后指定要使用从这些请求下载的响应调用的回调函数。

 第一个执行的请求通过调用 start_requests()（默认情况下）Request为在start_urls和中指定的URL生成的parse方法获取， 并且该方法作为请求的回调函数。

2.
在回调函数中，您将解析响应（网页），并返回带有提取的数据，Item对象， Request对象或这些对象的可迭代的对象。这些请求还将包含回调（可能是相同的），然后由Scrapy下载，然后由指定的回调处理它们的响应。

3.
在回调函数中，您通常使用选择器来解析页面内容 （但您也可以使用BeautifulSoup，lxml或您喜欢的任何机制），并使用解析的数据生成项目。

4. 最后，从爬虫返回的项目通常将持久存储到数据库（在某些项目管道中）或使用Feed导出写入文件。

即使这个循环（或多或少）适用于任何种类的爬虫，有不同种类的默认爬虫捆绑到Scrapy中用于不同的目的。我们将在这里谈论这些类型。

## `class scrapy.spiders.Spider`

这是最简单的爬虫，每个其他爬虫必须继承的爬虫（包括与Scrapy捆绑在一起的爬虫，以及你自己写的爬虫）。它不提供任何特殊功能。它只是提供了一个默认`start_requests()`实现，它从`start_urlsspider`属性发送请求，并`parse` 为每个结果响应调用`spider`的方法。

`name`
定义此爬虫名称的字符串。爬虫名称是爬虫如何由Scrapy定位（和实例化），因此它**必须是唯一的**。但是，没有什么能阻止你实例化同一个爬虫的多个实例。这是最重要的爬虫属性，它是必需的。

如果爬虫抓取单个域名，通常的做法是在域后面命名爬虫。因此，例如，抓取的爬虫mywebsite.com通常会被调用 mywebsite。

**注意**
在Python 2中，这必须是ASCII。

`allowed_domains`
允许此爬虫抓取的域的字符串的可选列表，指定一个列表可以抓取，其它就不会抓取了。

`start_urls`
当没有指定特定网址时，爬虫将开始抓取的网址列表。

`custom_settings`
运行此爬虫时将从项目宽配置覆盖的设置字典。它必须定义为类属性，因为设置在实例化之前更新。

有关可用内置设置的列表，请参阅： [内置设置参考](http://scrapy.readthedocs.io/en/latest/topics/settings.html#topics-settings-ref)。

`crawler`
此属性from_crawler()在初始化类后由类方法设置，并链接Crawler到此爬虫实例绑定到的对象。

Crawlers在项目中封装了很多组件，用于单个条目访问（例如扩展，中间件，信号管理器等）。有关详情，[请参阅抓取工具API](http://scrapy.readthedocs.io/en/latest/topics/api.html#topics-api-crawler)。

`settings`
运行此爬虫的配置。这是一个 Settings实例，有关此主题的详细介绍，[请参阅设置主题](http://scrapy.readthedocs.io/en/latest/topics/settings.html#topics-settings)。

`logger`
用Spider创建的Python记录器`name`。您可以使用它通过它发送日志消息，如[记录爬虫程序中所述](http://scrapy.readthedocs.io/en/latest/topics/logging.html#topics-logging-from-spiders)。

`from_crawler`（crawler，* args，** kwargs ）
是Scrapy用来创建爬虫的类方法。

您可能不需要直接覆盖这一点，因为默认实现充当方法的代理，`__init__()`使用给定的参数args和命名参数kwargs调用它。

尽管如此，此方法 在新实例中设置crawler和settings属性，以便以后可以在爬虫程序中访问它们。

- 参数：    
- crawler（Crawlerinstance） - 爬虫将绑定到的爬虫
- args（list） - 传递给**init**()方法的参数
- kwargs（dict） - 传递给**init**()方法的关键字参数

`start_requests（）`
此方法必须返回一个可迭代的第一个请求来抓取这个爬虫。

有了start_requests()，就不写了start_urls，写了也没有用。

默认实现是：start_urls，但是可以复写的方法start_requests。
例如，如果您需要通过使用POST请求登录来启动，您可以：

    class MySpider(scrapy.Spider):
        name = 'myspider'

        def start_requests(self):
            return [scrapy.FormRequest("http://www.example.com/login",
                                       formdata={'user': 'john', 'pass': 'secret'},
                                       callback=self.logged_in)]

        def logged_in(self, response):
            # here you would extract links to follow and return Requests for
            # each of them, with another callback
            pass

`make_requests_from_url(url)`
一种接收URL并返回Request 对象（或Request对象列表）进行抓取的方法。此方法用于在方法中构造初始请求 start_requests()，并且通常用于将URL转换为请求。

除非重写，此方法返回具有方法的Requests parse() 作为它们的回调函数，并启用dont_filter参数（Request有关更多信息，请参阅类）。

`parse(response)`
这是Scrapy用于处理下载的响应的默认回调，当它们的请求没有指定回调时。

该parse方法负责处理响应并返回所抓取的数据或更多的URL。其他请求回调具有与Spider类相同的要求。

此方法以及任何其他请求回调必须返回一个可迭代的Request和dicts或Item对象。

- 参数：    
- response（Response） - 解析的响应

`log(message[, level, component])`
包装器通过爬虫发送日志消息logger，保持向后兼容性。有关详细信息，请参阅 从Spider记录。

`closed(reason)`
当爬虫关闭时召唤。此方法为spider_closed信号的signals.connect()提供了一个快捷方式。

让我们看一个例子：

    import scrapy


    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = [
            'http://www.example.com/1.html',
            'http://www.example.com/2.html',
            'http://www.example.com/3.html',
        ]

        def parse(self, response):
            self.logger.info('A response from %s just arrived!', response.url)

从单个回调中返回多个请求和项：

    import scrapy

    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = [
            'http://www.example.com/1.html',
            'http://www.example.com/2.html',
            'http://www.example.com/3.html',
        ]

        def parse(self, response):
            for h3 in response.xpath('//h3').extract():
                yield {"title": h3}

            for url in response.xpath('//a/@href').extract():
                yield scrapy.Request(url, callback=self.parse)

你可以直接使用start_requests()，而不是start_urls; 项目可以更加方便获取数据：

    import scrapy
    from myproject.items import MyItem

    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']

        def start_requests(self):
            yield scrapy.Request('http://www.example.com/1.html', self.parse)
            yield scrapy.Request('http://www.example.com/2.html', self.parse)
            yield scrapy.Request('http://www.example.com/3.html', self.parse)

        def parse(self, response):
            for h3 in response.xpath('//h3').extract():
                yield MyItem(title=h3)

            for url in response.xpath('//a/@href').extract():
                yield scrapy.Request(url, callback=self.parse)

### Spider arguments

爬虫可以接收修改其行为的参数。爬虫参数的一些常见用法是定义起始URL或将爬网限制到网站的某些部分，但它们可用于配置爬虫的任何功能。

Spider crawl参数使用该-a选项通过命令 传递。例如：

`scrapy crawl myspider -a category=electronics`

爬虫可以在他们的**init**方法中访问参数：

    import scrapy

    class MySpider(scrapy.Spider):
        name = 'myspider'

        def __init__(self, category=None, *args, **kwargs):
            super(MySpider, self).__init__(*args, **kwargs)
            self.start_urls = ['http://www.example.com/categories/%s' % category]
            # ...

默认的**init**方法将获取任何爬虫参数，并将它们作为属性复制到爬虫。上面的例子也可以写成如下：

    import scrapy

    class MySpider(scrapy.Spider):
        name = 'myspider'

        def start_requests(self):
            yield scrapy.Request('http://www.example.com/categories/%s' % self.category)

请记住，spider参数只是字符串。爬虫不会自己做任何解析。如果要从命令行设置start_urls属性，则必须将它自己解析为列表，使用像 ast.literal_eval 或json.loads之类的属性 ，然后将其设置为属性。否则，你会导致迭代一个start_urls字符串（一个非常常见的python陷阱），导致每个字符被看作一个单独的url。

有效的用例是设置使用的http验证凭据HttpAuthMiddleware 或用户代理使用的用户代理UserAgentMiddleware：
`scrapy crawl myspider -a http_user=myuser -a http_pass=mypassword -a user_agent=mybot`

Spider参数也可以通过Scrapyd schedule.jsonAPI 传递。请参阅[Scrapyd文档](http://scrapyd.readthedocs.org/en/latest/)。

---

### 通用爬虫

Scrapy附带一些有用的通用爬虫，你可以使用它来子类化你的爬虫。他们的目的是为一些常见的抓取案例提供方便的功能，例如根据某些规则查看网站上的所有链接，从站点[地图抓取](http://www.sitemaps.org/)或解析XML / CSV Feed。

对于在以下爬虫中使用的示例，我们假设您有一个`TestItem`在`myproject.items`模块中声明的项目：

    import scrapy

    class TestItem(scrapy.Item):
        id = scrapy.Field()
        name = scrapy.Field()
        description = scrapy.Field()

#### 抓取爬虫

`类 scrapy.spiders.CrawlSpider`
这是最常用的爬行常规网站的爬虫，因为它通过定义一组规则为下列链接提供了一种方便的机制。它可能不是最适合您的特定网站或项目，但它是足够通用的几种情况，所以你可以从它开始，根据需要覆盖更多的自定义功能，或只是实现自己的爬虫。

除了从Spider继承的属性（你必须指定），这个类支持一个新的属性：

`rules`
它是一个（或多个）`Rule`对象的列表。每个都`Rule`定义了抓取网站的某种行为。规则对象如下所述。如果多个规则匹配相同的链接，则将根据它们在此属性中定义的顺序使用第一个。

这个爬虫还暴露了可覆盖的方法：

`parse_start_url(response)`
对于start_urls响应调用此方法。它允许解析初始响应，并且必须返回`Item`对象，`Request`对象或包含任何对象的迭代器。

#### 抓取规则

`class scrapy.spiders.Rule（link_extractor，callback = None，cb_kwargs = None，follow = None，process_links = None，process_request = None ）`
`link_extractor`是一个链接提取程序对象，它定义如何从每个爬网页面提取链接。

`callback`是一个可调用的或字符串（在这种情况下，将使用具有该名称的爬虫对象的方法），以便为使用指定的link_extractor提取的每个链接调用。这个回调接收一个响应作为其第一个参数，并且必须返回一个包含Item和 Request对象（或它们的任何子类）的列表。

**警告**
当编写爬网爬虫规则时，避免使用`parse`作为回调，因为`CrawlSpider`使用`parse`方法本身来实现其逻辑。所以如果你重写的`parse`方法，爬行爬虫将不再工作。

`cb_kwargs` 是包含要传递给回调函数的关键字参数的dict。

`follow`是一个布尔值，它指定是否应该从使用此规则提取的每个响应中跟踪链接。如果`callback`是`None follow`默认为`True`，否则默认为`False`。

`process_links`是一个可调用的或一个字符串（在这种情况下，将使用具有该名称的爬虫对象的方法），将使用指定从每个响应提取的每个链接列表调用该方法`link_extractor`。这主要用于过滤目的。

`process_request` 是一个可调用的或一个字符串（在这种情况下，将使用具有该名称的爬虫对象的方法），它将被此规则提取的每个请求调用，并且必须返回一个请求或无（过滤出请求） 。

#### 抓取爬虫示例

现在让我们来看一个CrawlSpider的例子：

    import scrapy
    from scrapy.spiders import CrawlSpider, Rule
    from scrapy.linkextractors import LinkExtractor

    class MySpider(CrawlSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com']

        rules = (
            # Extract links matching 'category.php' (but not matching 'subsection.php')
            # and follow links from them (since no callback means follow=True by default).
            Rule(LinkExtractor(allow=('category\.php', ), deny=('subsection\.php', ))),

            # Extract links matching 'item.php' and parse them with the spider's method parse_item
            Rule(LinkExtractor(allow=('item\.php', )), callback='parse_item'),
        )

        def parse_item(self, response):
            self.logger.info('Hi, this is an item page! %s', response.url)
            item = scrapy.Item()
            item['id'] = response.xpath('//td[@id="item_id"]/text()').re(r'ID: (\d+)')
            item['name'] = response.xpath('//td[@id="item_name"]/text()').extract()
            item['description'] = response.xpath('//td[@id="item_description"]/text()').extract()
            return item

这个爬虫会开始抓取example.com的主页，收集类别链接和项链接，用parse_item方法解析后者。对于每个项目响应，将使用XPath从HTML中提取一些数据，并将Item使用它填充。

#### XMLFeedSpider

`class scrapy.spiders.XMLFeedSpider`
XMLFeedSpider设计用于通过以特定节点名称迭代XML订阅源来解析XML订阅源。迭代器可以选自：iternodes，xml和html。iternodes为了性能原因，建议使用迭代器，因为xml和迭代器html一次生成整个DOM为了解析它。但是，html当使用坏标记解析XML时，使用作为迭代器可能很有用。

要设置迭代器和标记名称，必须定义以下类属性：

-
`iterator`
定义要使用的迭代器的字符串。它可以是：

- `'iternodes'` - 基于正则表达式的快速迭代器
- `'html'`- 使用的迭代器Selector。请记住，这使用DOM解析，并且必须加载所有DOM在内存中，这可能是一个大饲料的问题
- `'xml'`- 使用的迭代器Selector。请记住，这使用DOM解析，并且必须加载所有DOM在内存中，这可能是一个大饲料的问题
它默认为：`'iternodes'`。

`itertag`
一个具有要迭代的节点（或元素）的名称的字符串。示​​例：
`itertag = 'product'`

`namespaces`
定义该文档中将使用此爬虫处理的命名空间的元组列表。在 与将用于自动注册使用的命名空间 的方法。(prefix, uri)prefixuriregister_namespace()

然后，您可以在属性中指定具有命名空间的itertag 节点。

例：

    class YourSpider(XMLFeedSpider):

        namespaces = [('n', 'http://www.sitemaps.org/schemas/sitemap/0.9')]
        itertag = 'n:url'
        # ...

除了这些新的属性，这个爬虫也有以下可重写的方法：

`adapt_response(response)`
一种在爬虫开始解析响应之前，在响应从爬虫中间件到达时立即接收的方法。它可以用于在解析之前修改响应主体。此方法接收响应并返回响应（它可以是相同的或另一个）。

`parse_node(response, selector)`
对于与提供的标记名称（itertag）匹配的节点，将调用此方法。接收Selector每个节点的响应和 。覆盖此方法是必需的。否则，你的爬虫将不工作。此方法必须返回一个Item对象，一个 Request对象或包含任何对象的迭代器。

`process_results(response, results)`
对于由爬虫返回的每个结果（Items or Requests），将调用此方法，并且它将在将结果返回到框架核心之前执行所需的任何最后处理，例如设置项目ID。它接收结果列表和产生那些结果的响应。它必须返回结果列表（Items or Requests）。

#### XMLFeedSpider示例

这些爬虫很容易使用，让我们看一个例子：

    from scrapy.spiders import XMLFeedSpider
    from myproject.items import TestItem

    class MySpider(XMLFeedSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com/feed.xml']
        iterator = 'iternodes'  # This is actually unnecessary, since it's the default value
        itertag = 'item'

        def parse_node(self, response, node):
            self.logger.info('Hi, this is a <%s> node!: %s', self.itertag, ''.join(node.extract()))

            item = TestItem()
            item['id'] = node.xpath('@id').extract()
            item['name'] = node.xpath('name').extract()
            item['description'] = node.xpath('description').extract()
            return item

基本上我们做的是创建一个爬虫，从给定的下载一个start_urls，然后遍历每个item标签，打印出来，并存储一些随机数据Item。

### CSVFeedSpider

`class scrapy.spiders.CSVF`
这个爬虫非常类似于XMLFeedSpider，除了它迭代行，而不是节点。在每次迭代中调用的方法是parse_row()。

`delimiter`
CSV文件中每个字段的带分隔符的字符串默认为','（逗号）。

`quotechar`
CSV文件中每个字段的包含字符的字符串默认为'"'（引号）。

`headers`
文件CSV Feed中包含的行的列表，用于从中提取字段。

`parse_row(response, row)`
使用CSV文件的每个提供（或检测到）标头的键接收响应和dict（表示每行）。这个爬虫还给予机会重写adapt_response和process_results方法的前和后处理的目的。

#### CSVFeedSpider示例

让我们看一个类似于前一个例子，但使用 CSVFeedSpider：

    from scrapy.spiders import CSVFeedSpider
    from myproject.items import TestItem

    class MySpider(CSVFeedSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com/feed.csv']
        delimiter = ';'
        quotechar = "'"
        headers = ['id', 'name', 'description']

        def parse_row(self, response, row):
            self.logger.info('Hi, this is a row!: %r', row)

            item = TestItem()
            item['id'] = row['id']
            item['name'] = row['name']
            item['description'] = row['description']
            return item

### SitemapSpider

`class scrapy.spiders.SitemapSpider`
SitemapSpider允许您通过使用[Sitemaps](http://www.sitemaps.org/)发现网址来抓取[网站](http://www.sitemaps.org/)。

它支持嵌套Sitemap和从[robots.txt](http://www.robotstxt.org/)发现Sitemap网址 。

`sitemap_urls`
指向您要抓取的网址的网站的网址列表。

您还可以指向robots.txt，它会解析为从中提取Sitemap网址。

`sitemap_rules`
元组列表其中：`(regex, callback)`

- regex是与从Sitemap中提取的网址相匹配的正则表达式。 regex可以是一个str或一个编译的正则表达式对象。
- callback是用于处理与正则表达式匹配的url的回调。callback可以是字符串（指示蜘蛛方法的名称）或可调用的。

例如：
`sitemap_rules = [('/product/', 'parse_product')]`

规则按顺序应用，只有匹配的第一个将被使用。
如果省略此属性，则会在parse回调中处理在站点地图中找到的所有网址。

`sitemap_follow`
应遵循的网站地图的正则表达式列表。这只适用于使用指向其他Sitemap文件的[Sitemap索引文件](http://www.sitemaps.org/protocol.html#index)的网站。

默认情况下，将跟踪所有网站地图。

`sitemap_alternate_links`
指定是否url应遵循一个备用链接。这些是在同一个url块中传递的另一种语言的同一网站的链接。

例如：

    <url>
        <loc>http://example.com/</loc>
        <xhtml:link rel="alternate" hreflang="de" href="http://example.com/de"/>
    </url>

使用`sitemap_alternate_linksset`，这将检索两个URL。随着 `sitemap_alternate_links`禁用，只有[http://example.com/将进行检索。](http://example.com/%E5%B0%86%E8%BF%9B%E8%A1%8C%E6%A3%80%E7%B4%A2%E3%80%82)

默认为`sitemap_alternate_links`禁用。

#### SitemapSpider示例

最简单的示例：使用parse回调处理通过站点地图发现的所有网址 ：

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/sitemap.xml']

        def parse(self, response):
            pass # ... scrape item here ...

使用某个回调处理一些网址，并使用不同的回调处理其他网址：

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/sitemap.xml']
        sitemap_rules = [
            ('/product/', 'parse_product'),
            ('/category/', 'parse_category'),
        ]

        def parse_product(self, response):
            pass # ... scrape product ...

        def parse_category(self, response):
            pass # ... scrape category ...

关注[robots.txt](http://www.robotstxt.org/)文件中定义的sitemaps，并且只跟踪其网址包含`/sitemap_shop`以下内容的Sitemap ：

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/robots.txt']
        sitemap_rules = [
            ('/shop/', 'parse_shop'),
        ]
        sitemap_follow = ['/sitemap_shops']

        def parse_shop(self, response):
            pass # ... scrape shop here ...

将SitemapSpider与其他来源网址结合使用：

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/robots.txt']
        sitemap_rules = [
            ('/shop/', 'parse_shop'),
        ]

        other_urls = ['http://www.example.com/about']

        def start_requests(self):
            requests = list(super(MySpider, self).start_requests())
            requests += [scrapy.Request(x, self.parse_other) for x in self.other_urls]
            return requests

        def parse_shop(self, response):
            pass # ... scrape shop here ...

        def parse_other(self, response):
            pass # ... scrape other here ...
