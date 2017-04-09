---
layout: "post"
title: "Scrapy爬虫入门教程七 Item Loaders（项目加载器）"
date: "2017-04-09 14:31"
tags:
- Scrapy
comments: true
---

# Scrapy爬虫入门教程七 Item Loaders（项目加载器）


**开发环境：**
`Python 3.6.0 版本` （当前最新）
`Scrapy 1.3.2 版本` （当前最新）

# 项目加载器

项目加载器提供了一种方便的机制来填充抓取的[项目](http://scrapy.readthedocs.io/en/latest/topics/items.html#topics-items)。即使可以使用自己的类似字典的API填充项目，项目加载器提供了一个更方便的API，通过自动化一些常见的任务，如解析原始提取的数据，然后分配它从剪贴过程中填充他们。

换句话说，[Items](http://scrapy.readthedocs.io/en/latest/topics/items.html#topics-items)提供了抓取数据的容器，而Item Loader提供了填充该容器的机制。

项目加载器旨在提供一种灵活，高效和容易的机制，通过爬虫或源格式（HTML，XML等）扩展和覆盖不同的字段解析规则，而不会成为维护的噩梦。

## 使用装载机项目来填充的项目

要使用项目加载器，您必须首先实例化它。您可以使用类似dict的对象（例如Item或dict）实例化它，也可以不使用它，在这种情况下，项目将在Item Loader构造函数中使用属性中指定的Item类自动`ItemLoader.default_item_class` 实例化。

然后，您开始收集值到项装载程序，通常使用[选择器](http://scrapy.readthedocs.io/en/latest/topics/selectors.html#topics-selectors)。您可以向同一项目字段添加多个值; 项目加载器将知道如何使用适当的处理函数“加入”这些值。

这里是[Spider](http://scrapy.readthedocs.io/en/latest/topics/spiders.html#topics-spiders)中典型的Item Loader用法，使用[Items部分](http://scrapy.readthedocs.io/en/latest/topics/items.html#topics-items)中声明的[Product](http://scrapy.readthedocs.io/en/latest/topics/items.html#topics-items-declaring)项：

    from scrapy.loader import ItemLoader
    from myproject.items import Product

    def parse(self, response):
        l = ItemLoader(item=Product(), response=response)
        l.add_xpath('name', '//div[@class="product_name"]')
        l.add_xpath('name', '//div[@class="product_title"]')
        l.add_xpath('price', '//p[@id="price"]')
        l.add_css('stock', 'p#stock]')
        l.add_value('last_updated', 'today') # you can also use literal values
        return l.load_item()

通过快速查看该代码，我们可以看到该`name`字段正从页面中两个不同的XPath位置提取：

1. `//div[@class="product_name"]`
2. `//div[@class="product_title"]`
换句话说，通过使用`add_xpath()`方法从两个`XPath`位置提取数据来收集数据。这是稍后将分配给name字段的数据。

之后，类似的调用用于`price`和`stock`字段（后者使用带有`add_css()`方法的CSS选择器），最后使用不同的方法last_update直接使用文字值（today）填充字段add_value()。

最后，收集的所有数据时，该`ItemLoader.load_item()`方法被称为实际上返回填充先前提取并与收集到的数据的项目`add_xpath()`， `add_css()`和`add_value()`调用。

## 输入和输出处理器

项目加载器对于每个（项目）字段包含一个输入处理器和一个输出处理器。输入处理器只要它的接收处理所提取的数据（通过add_xpath()，add_css()或 add_value()方法）和输入处理器的结果被收集并保持ItemLoader内部。收集所有数据后，ItemLoader.load_item()调用该 方法来填充和获取填充 Item对象。这是当输出处理器使用先前收集的数据（并使用输入处理器处理）调用时。输出处理器的结果是分配给项目的最终值。

让我们看一个例子来说明如何为特定字段调用输入和输出处理器（同样适用于任何其他字段）：

    l = ItemLoader(Product(), some_selector)
    l.add_xpath('name', xpath1) # (1)
    l.add_xpath('name', xpath2) # (2)
    l.add_css('name', css) # (3)
    l.add_value('name', 'test') # (4)
    return l.load_item() # (5)

所以会发生什么：

1. 从数据xpath1提取出来，并通过所传递的输入处理器的的name字段。输入处理器的结果被收集并保存在项目加载器中（但尚未分配给项目）。
2. 从中xpath2提取数据，并通过（1）中使用的同一输入处理器。输入处理器的结果附加到（1）中收集的数据（如果有）。
3. 这种情况类似于先前的情况，除了数据从cssCSS选择器提取，并且通过在（1）和（2）中使用的相同的输入处理器。输入处理器的结果附加到在（1）和（2）中收集的数据（如果有的话）。
4. 这种情况也与之前的类似，除了要收集的值直接分配，而不是从XPath表达式或CSS选择器中提取。但是，该值仍然通过输入处理器。在这种情况下，由于该值不可迭代，因此在将其传递给输入处理器之前，它将转换为单个元素的可迭代，因为输入处理器总是接收迭代。
5. 在步骤（1），（2），（3）和（4）中收集的数据通过name字段的输出处理器。输出处理器的结果是分配给name 项目中字段的值。

值得注意的是，处理器只是可调用对象，它们使用要解析的数据调用，并返回解析的值。所以你可以使用任何功能作为输入或输出处理器。唯一的要求是它们必须接受一个（也只有一个）位置参数，这将是一个迭代器。

**注意**

> 输入和输出处理器都必须接收一个迭代器作为它们的第一个参数。这些函数的输出可以是任何东西。输入处理器的结果将附加到包含收集的值（对于该字段）的内部列表（在加载程序中）。输出处理器的结果是最终分配给项目的值。

另一件需要记住的事情是，输入处理器返回的值在内部（在列表中）收集，然后传递到输出处理器以填充字段。

最后，但并非最不重要的是，Scrapy自带一些常用的处理器内置的方便。

## 声明项目加载器

项目加载器通过使用类定义语法声明为Items。这里是一个例子：

    from scrapy.loader import ItemLoader
    from scrapy.loader.processors import TakeFirst, MapCompose, Join

    class ProductLoader(ItemLoader):

        default_output_processor = TakeFirst()

        name_in = MapCompose(unicode.title)
        name_out = Join()

        price_in = MapCompose(unicode.strip)

        # ...

可以看到，输入处理器使用_in后缀声明，而输出处理器使用_out后缀声明。您还可以使用ItemLoader.default_input_processor和 ItemLoader.default_output_processor属性声明默认输入/输出 处理器。

## 声明输入和输出处理器

如上一节所述，输入和输出处理器可以在Item Loader定义中声明，这种方式声明输入处理器是很常见的。但是，还有一个地方可以指定要使用的输入和输出处理器：在项目字段 元数据中。这里是一个例子：

    import scrapy
    from scrapy.loader.processors import Join, MapCompose, TakeFirst
    from w3lib.html import remove_tags

    def filter_price(value):
        if value.isdigit():
            return value

    class Product(scrapy.Item):
        name = scrapy.Field(
            input_processor=MapCompose(remove_tags),
            output_processor=Join(),
        )
        price = scrapy.Field(
            input_processor=MapCompose(remove_tags, filter_price),
            output_processor=TakeFirst(),
        )

    >>> from scrapy.loader import ItemLoader
    >>> il = ItemLoader(item=Product())
    >>> il.add_value('name', [u'Welcome to my', u'<strong>website</strong>'])
    >>> il.add_value('price', [u'€', u'<span>1000</span>'])
    >>> il.load_item()
    {'name': u'Welcome to my website', 'price': u'1000'}

输入和输出处理器的优先级顺序如下：

1. 项目加载程序字段特定属性：field_in和field_out（最高优先级）
2. 字段元数据（input_processor和output_processor键）
3. 项目加载器默认值：ItemLoader.default_input_processor()和 ItemLoader.default_output_processor()（最低优先级）

参见：[重用和扩展项目加载器](http://scrapy.readthedocs.io/en/latest/topics/loaders.html#topics-loaders-extending)。

## 项目加载器上下文

项目加载器上下文是在项目加载器中的所有输入和输出处理器之间共享的任意键/值的dict。它可以在声明，实例化或使用Item Loader时传递。它们用于修改输入/输出处理器的行为。

例如，假设您有一个parse_length接收文本值并从中提取长度的函数：

    def  parse_length （text ， loader_context ）：
        unit  =  loader_context 。get （'unit' ， 'm' ）
        ＃...长度解析代码在这里...
        return  parsed_length

通过接受一个loader_context参数，该函数显式地告诉Item Loader它能够接收一个Item Loader上下文，因此Item Loader在调用它时传递当前活动的上下文，因此处理器功能（parse_length在这种情况下）可以使用它们。

有几种方法可以修改Item Loader上下文值：

1.
通过修改当前活动的Item Loader上下文（context属性）：

     loader = ItemLoader(product)
     loader.context['unit'] = 'cm'

2.
On Item Loader实例化（Item Loader构造函数的关键字参数存储在Item Loader上下文中）：

     loader = ItemLoader(product, unit='cm')

3.
On Item Loader声明，对于那些支持使用Item Loader上下文实例化的输入/输出处理器。MapCompose是其中之一：

     class ProductLoader(ItemLoader):
         length_out = MapCompose(parse_length, unit='cm')

## ItemLoader对象

    class scrapy.loader.ItemLoader([item, selector, response, ]**kwargs)

返回一个新的Item Loader来填充给定的Item。如果没有给出项目，则使用中的类自动实例化 default_item_class。

当使用选择器或响应参数实例化时，ItemLoader类提供了使用选择器从网页提取数据的方便的机制。

**参数：**

- item（Item对象）-项目实例来填充用以后调用 add_xpath()，add_css()或add_value()。
- selector（Selectorobject） - 当使用add_xpath()（或。add_css()）或replace_xpath() （或replace_css()）方法时，从中提取数据的选择器 。
- response（Responseobject） - 用于使用构造选择器的响应 default_selector_class，除非给出选择器参数，在这种情况下，将忽略此参数。
项目，选择器，响应和剩余的关键字参数被分配给Loader上下文（可通过context属性访问）。

### ItemLoader 实例有以下方法：

`get_value（value，* processors，** kwargs ）`
处理给定value的给定processors和关键字参数。

可用的关键字参数：

**参数：    re（str 或compiled regex）**
一个正则表达式extract_regex()，用于使用方法从给定值提取数据，在处理器之前应用
例子：

    >>> from scrapy.loader.processors import TakeFirst
    >>> loader.get_value(u'name: foo', TakeFirst(), unicode.upper, re='name: (.+)')
    'FOO`

`add_value（field_name，value，* processors，** kwargs ）`
处理，然后添加给value定字段的给定。

该值首先通过get_value()赋予 processors和kwargs，然后通过 字段输入处理器及其结果追加到为该字段收集的数据。如果字段已包含收集的数据，则会添加新数据。

给定field_name可以是None，在这种情况下可以添加多个字段的值。并且已处理的值应为一个字段，其中field_name映射到值。

例子：

    loader.add_value('name', u'Color TV')
    loader.add_value('colours', [u'white', u'blue'])
    loader.add_value('length', u'100')
    loader.add_value('name', u'name: foo', TakeFirst(), re='name: (.+)')
    loader.add_value(None, {'name': u'foo', 'sex': u'male'})

`replace_value（field_name，value，* processors，** kwargs ）`
类似于add_value()但是用新值替换收集的数据，而不是添加它。

`get_xpath（xpath，* processors，** kwargs ）`
类似于ItemLoader.get_value()但接收XPath而不是值，用于从与此相关联的选择器提取unicode字符串的列表ItemLoader。

**参数：**

- xpath（str） - 从中​​提取数据的XPath
- re（str 或compiled regex） - 用于从所选XPath区域提取数据的正则表达式
例子：

    # HTML snippet: <p class="product-name">Color TV</p>
    loader.get_xpath('//p[@class="product-name"]')
    # HTML snippet: <p id="price">the price is $1200</p>
    loader.get_xpath('//p[@id="price"]', TakeFirst(), re='the price is (.*)')

`add_xpath（field_name，xpath，* processor，** kwargs ）`
类似于ItemLoader.add_value()但接收XPath而不是值，用于从与此相关联的选择器提取unicode字符串的列表ItemLoader。

见get_xpath()的kwargs。

**参数：**
xpath（str） - 从中​​提取数据的XPath

例子：

    # HTML snippet: <p class="product-name">Color TV</p>
    loader.add_xpath('name', '//p[@class="product-name"]')
    # HTML snippet: <p id="price">the price is $1200</p>
    loader.add_xpath('price', '//p[@id="price"]', re='the price is (.*)')

`replace_xpath（field_name，xpath，* processor，** kwargs ）`
类似于add_xpath()但替换收集的数据，而不是添加它。

`get_css（css，* processors，** kwargs ）`
类似于ItemLoader.get_value()但接收一个CSS选择器而不是一个值，用于从与此相关的选择器提取一个unicode字符串列表ItemLoader。

**参数：**

- css（str） - 从中​​提取数据的CSS选择器
- re（str 或compiled regex） - 用于从所选CSS区域提取数据的正则表达式
例子：

    # HTML snippet: <p class="product-name">Color TV</p>
    loader.get_css('p.product-name')
    # HTML snippet: <p id="price">the price is $1200</p>
    loader.get_css('p#price', TakeFirst(), re='the price is (.*)')

`add_css（field_name，css，* processors，** kwargs ）`
类似于ItemLoader.add_value()但接收一个CSS选择器而不是一个值，用于从与此相关的选择器提取一个unicode字符串列表ItemLoader。

见get_css()的kwargs。

**参数：**
css（str） - 从中​​提取数据的CSS选择器
例子：

    # HTML snippet: <p class="product-name">Color TV</p>
    loader.add_css('name', 'p.product-name')
    # HTML snippet: <p id="price">the price is $1200</p>
    loader.add_css('price', 'p#price', re='the price is (.*)')

`replace_css（field_name，css，* processors，** kwargs ）`
类似于add_css()但替换收集的数据，而不是添加它。

`load_item（）`
使用目前收集的数据填充项目，并返回。收集的数据首先通过输出处理器，以获得要分配给每个项目字段的最终值。

`nested_xpath（xpath ）`
使用xpath选择器创建嵌套加载器。所提供的选择器应用于与此相关的选择器ItemLoader。嵌套装载机股份Item 与母公司ItemLoader这么调用add_xpath()， add_value()，replace_value()等会像预期的那样。

`nested_css（css ）`
使用css选择器创建嵌套加载器。所提供的选择器应用于与此相关的选择器ItemLoader。嵌套装载机股份Item 与母公司ItemLoader这么调用add_xpath()， add_value()，replace_value()等会像预期的那样。

`get_collected_values（field_name ）`
返回给定字段的收集值。

`get_output_value（field_name ）`
返回给定字段使用输出处理器解析的收集值。此方法根本不填充或修改项目。

`get_input_processor（field_name ）`
返回给定字段的输入处理器。

`get_output_processor（field_name ）`
返回给定字段的输出处理器。

### ItemLoader 实例具有以下属性：

`item`
Item此项目加载器解析的对象。

`context`
此项目Loader 的当前活动上下文。

`default_item_class`
Item类（或工厂），用于在构造函数中未给出时实例化项。

`default_input_processor`
用于不指定一个字段的字段的默认输入处理器。

`default_output_processor`
用于不指定一个字段的字段的默认输出处理器。

`default_selector_class`
所使用的类构造selector的此 ItemLoader，如果只响应在构造函数给出。如果在构造函数中给出了选择器，则忽略此属性。此属性有时在子类中被覆盖。

`selector`
Selector从中提取数据的对象。它是在构造函数中给出的选择器，或者是从构造函数中使用的给定的响应创建的 default_selector_class。此属性意味着是只读的。

## 嵌套装载器

当解析来自文档的子部分的相关值时，创建嵌套加载器可能是有用的。假设您从页面的页脚中提取细节，看起来像：

例：

    <footer>
        <a class="social" href="http://facebook.com/whatever">Like Us</a>
        <a class="social" href="http://twitter.com/whatever">Follow Us</a>
        <a class="email" href="mailto:whatever@example.com">Email Us</a>
    </footer>

如果没有嵌套加载器，则需要为要提取的每个值指定完整的xpath（或css）。

例：

    loader = ItemLoader(item=Item())
    # load stuff not in the footer
    loader.add_xpath('social', '//footer/a[@class = "social"]/@href')
    loader.add_xpath('email', '//footer/a[@class = "email"]/@href')
    loader.load_item()

相反，您可以使用页脚选择器创建嵌套加载器，并相对于页脚添加值。功能是相同的，但您避免重复页脚选择器。

例：

    loader = ItemLoader(item=Item())
    # load stuff not in the footer
    footer_loader = loader.nested_xpath('//footer')
    footer_loader.add_xpath('social', 'a[@class = "social"]/@href')
    footer_loader.add_xpath('email', 'a[@class = "email"]/@href')
    # no need to call footer_loader.load_item()
    loader.load_item()

您可以任意嵌套加载器，并且可以使用xpath或css选择器。作为一般的指导原则，当他们使你的代码更简单，但不要超越嵌套或使用解析器可能变得难以阅读使用嵌套加载程序。

## 重用和扩展项目加载器

随着你的项目越来越大，越来越多的爬虫，维护成为一个根本的问题，特别是当你必须处理每个爬虫的许多不同的解析规则，有很多异常，但也想重用公共处理器。

项目加载器旨在减轻解析规则的维护负担，同时不会失去灵活性，同时提供了扩展和覆盖它们的方便的机制。因此，项目加载器支持传统的Python类继承，以处理特定爬虫（或爬虫组）的差异。

例如，假设某个特定站点以三个短划线（例如）包含其产品名称，并且您不希望最终在最终产品名称中删除那些破折号。---Plasma TV---

以下是如何通过重用和扩展默认产品项目Loader（ProductLoader）来删除这些破折号：

    from scrapy.loader.processors import MapCompose
    from myproject.ItemLoaders import ProductLoader

    def strip_dashes(x):
        return x.strip('-')

    class SiteSpecificLoader(ProductLoader):
        name_in = MapCompose(strip_dashes, ProductLoader.name_in)

另一种扩展项目加载器可能非常有用的情况是，当您有多种源格式，例如XML和HTML。在XML版本中，您可能想要删除CDATA事件。下面是一个如何做的例子：

    from scrapy.loader.processors import MapCompose
    from myproject.ItemLoaders import ProductLoader
    from myproject.utils.xml import remove_cdata

    class XmlProductLoader(ProductLoader):
        name_in = MapCompose(remove_cdata, ProductLoader.name_in)

这就是你通常扩展输入处理器的方式。

对于输出处理器，更常见的是在字段元数据中声明它们，因为它们通常仅依赖于字段而不是每个特定站点解析规则（如输入处理器）。另请参见： 声明输入和输出处理器。

还有许多其他可能的方法来扩展，继承和覆盖您的项目加载器，不同的项目加载器层次结构可能更适合不同的项目。Scrapy只提供了机制; 它不强加任何特定的组织你的Loader集合 - 这取决于你和你的项目的需要。

### 可用内置处理器

即使您可以使用任何可调用函数作为输入和输出处理器，Scrapy也提供了一些常用的处理器，如下所述。其中一些，像MapCompose（通常用作输入处理器）组成按顺序执行的几个函数的输出，以产生最终的解析值。

下面是所有内置处理器的列表：

`class scrapy.loader.processors.Identity`

最简单的处理器，什么都不做。它返回原始值不变。它不接收任何构造函数参数，也不接受Loader上下文。

例：

    >>> from scrapy.loader.processors import Identity
    >>> proc = Identity()
    >>> proc(['one', 'two', 'three'])
    ['one', 'two', 'three']

`class scrapy.loader.processors.TakeFirst`

从接收到的值中返回第一个非空值/非空值，因此它通常用作单值字段的输出处理器。它不接收任何构造函数参数，也不接受Loader上下文。

例：

    >>> from scrapy.loader.processors import TakeFirst
    >>> proc = TakeFirst()
    >>> proc(['', 'one', 'two', 'three'])
    'one'

`class scrapy.loader.processors.Join(separator=u' ')`

返回与构造函数中给定的分隔符联接的值，默认为。它不接受加载器上下文。u' '

当使用默认分隔符时，此处理器相当于以下功能： u' '.join

例子：

    >>> from scrapy.loader.processors import Join
    >>> proc = Join()
    >>> proc(['one', 'two', 'three'])
    u'one two three'
    >>> proc = Join('<br>')
    >>> proc(['one', 'two', 'three'])
    u'one<br>two<br>three'

`class scrapy.loader.processors.Compose(*functions, **default_loader_context)`

由给定函数的组合构成的处理器。这意味着该处理器的每个输入值都被传递给第一个函数，并且该函数的结果被传递给第二个函数，依此类推，直到最后一个函数返回该处理器的输出值。

默认情况下，停止进程None值。可以通过传递关键字参数来更改此行为stop_on_none=False。

例：

    >>> from scrapy.loader.processors import Compose
    >>> proc = Compose(lambda v: v[0], str.upper)
    >>> proc(['hello', 'world'])
    'HELLO'

每个功能可以可选地接收loader_context参数。对于那些处理器，这个处理器将通过该参数传递当前活动的Loader上下文。

在构造函数中传递的关键字参数用作传递给每个函数调用的默认Loader上下文值。但是，传递给函数的最后一个Loader上下文值将被当前可用该属性访问的当前活动Loader上下文ItemLoader.context() 覆盖。

`class scrapy.loader.processors.MapCompose(*functions, **default_loader_context)`

与处理器类似，由给定功能的组成构成的Compose处理器。与此处理器的区别在于内部结果在函数之间传递的方式，如下所示：

该处理器的输入值被迭代，并且第一函数被应用于每个元素。这些函数调用的结果（每个元素一个）被连接以构造新的迭代，然后用于应用​​第二个函数，等等，直到最后一个函数被应用于收集的值列表的每个值远。最后一个函数的输出值被连接在一起以产生该处理器的输出。

每个特定函数可以返回值或值列表，这些值通过应用于其他输入值的相同函数返回的值列表展平。函数也可以返回None，在这种情况下，该函数的输出将被忽略，以便在链上进行进一步处理。

此处理器提供了一种方便的方法来组合只使用单个值（而不是iterables）的函数。由于这个原因， MapCompose处理器通常用作输入处理器，因为数据通常使用选择器的 extract()方法提取，选择器返回unicode字符串的列表。

下面的例子应该说明它是如何工作的：

    >>> def filter_world(x):
    ...     return None if x == 'world' else x
    ...
    >>> from scrapy.loader.processors import MapCompose
    >>> proc = MapCompose(filter_world, unicode.upper)
    >>> proc([u'hello', u'world', u'this', u'is', u'scrapy'])
    [u'HELLO, u'THIS', u'IS', u'SCRAPY']

与Compose处理器一样，函数可以接收Loader上下文，并且构造函数关键字参数用作默认上下文值。有关Compose更多信息，请参阅 处理器。

`class scrapy.loader.processors.SelectJmes(json_path)`

使用提供给构造函数的json路径查询值，并返回输出。需要运行jmespath（[https://github.com/jmespath/jmespath.py](https://github.com/jmespath/jmespath.py)）。该处理器一次只需要一个输入。

例：

    >>> from scrapy.loader.processors import SelectJmes, Compose, MapCompose
    >>> proc = SelectJmes("foo") #for direct use on lists and dictionaries
    >>> proc({'foo': 'bar'})
    'bar'
    >>> proc({'foo': {'bar': 'baz'}})
    {'bar': 'baz'}

使用Json：

    >>> import json
    >>> proc_single_json_str = Compose(json.loads, SelectJmes("foo"))
    >>> proc_single_json_str('{"foo": "bar"}')
    u'bar'
    >>> proc_json_list = Compose(json.loads, MapCompose(SelectJmes('foo')))
    >>> proc_json_list('[{"foo":"bar"}, {"baz":"tar"}]')
    [u'bar']
