---
layout: "post"
title: "Scrapy爬虫入门教程六 Items（项目）"
date: "2017-04-09 14:30"
tags:
- Scrapy
comments: true
---

# Scrapy爬虫入门教程六 Items（项目）


**开发环境：**
`Python 3.6.0 版本` （当前最新）
`Scrapy 1.3.2 版本` （当前最新）

# Items

主要目标是从非结构化来源（通常是网页）提取结构化数据。Scrapy爬虫可以将提取的数据作为Python语句返回。虽然方便和熟悉，Python dicts缺乏结构：很容易在字段名称中输入错误或返回不一致的数据，特别是在与许多爬虫的大项目。

要定义公共输出数据格式，Scrapy提供Item类。 Item对象是用于收集所抓取的数据的简单容器。它们提供了一个[类似字典](https://docs.python.org/2/library/stdtypes.html#dict)的 API，具有用于声明其可用字段的方便的语法。

各种Scrapy组件使用项目提供的额外信息：导出器查看声明的字段以计算要导出的列，序列化可以使用项字段元数据trackref 定制，跟踪项实例以帮助查找内存泄漏（请参阅使用[trackref](http://scrapy.readthedocs.io/en/latest/topics/leaks.html#topics-leaks-trackrefs)调试内存泄漏）等。

## 声明项目

使用简单的类定义语法和Field 对象来声明项目。这里是一个例子：

    import scrapy

    class Product(scrapy.Item):
        name = scrapy.Field()
        price = scrapy.Field()
        stock = scrapy.Field()
        last_updated = scrapy.Field(serializer=str)

**注意**

> 熟悉Django的人会注意到Scrapy Items被声明为类似于Django Models，只是Scrapy Items比较简单，因为没有不同字段类型的概念。

## 项目字段

`Field`对象用于为每个字段指定元数据。例如，`last_updated`上面示例中所示的字段的序列化函数。

您可以为每个字段指定任何种类的元数据。对Field对象接受的值没有限制。出于同样的原因，没有所有可用元数据键的参考列表。Field对象中定义的每个键可以由不同的组件使用，并且只有那些组件知道它。您也可以定义和使用Field项目中的任何其他 键，为您自己的需要。Field对象的主要目标 是提供一种在一个地方定义所有字段元数据的方法。通常，那些行为取决于每个字段的组件使用某些字段键来配置该行为。您必须参考他们的文档，以查看每个组件使用哪些元数据键。

重要的是要注意，Field用于声明项目的对象不会被分配为类属性。相反，可以通过`Item.fields`属性访问它们。

### 使用项目

下面是使用上面声明的Product项目对项目执行的常见任务的一些示例 。你会注意到API非常类似于dict API。

#### 创建项目

    >>> product = Product(name='Desktop PC', price=1000)
    >>> print product
    Product(name='Desktop PC', price=1000)

#### 获取字段值

    >>> product['name']
    Desktop PC
    >>> product.get('name')
    Desktop PC

    >>> product['price']
    1000

    >>> product['last_updated']
    Traceback (most recent call last):
        ...
    KeyError: 'last_updated'

    >>> product.get('last_updated', 'not set')
    not set

    >>> product['lala'] # getting unknown field
    Traceback (most recent call last):
        ...
    KeyError: 'lala'

    >>> product.get('lala', 'unknown field')
    'unknown field'

    >>> 'name' in product  # is name field populated?
    True

    >>> 'last_updated' in product  # is last_updated populated?
    False

    >>> 'last_updated' in product.fields  # is last_updated a declared field?
    True

    >>> 'lala' in product.fields  # is lala a declared field?
    False

#### 设置字段值

    >>> product [ 'last_updated' ]  =  'today'
    >>> product [ 'last_updated' ]
    today

    >>> product [ 'lala' ]  =  'test'  ＃设置未知字段
    Traceback（最近调用最后一次）：
        ...
    KeyError：'产品不支持字段：lala'

#### 访问所有填充值

要访问所有填充值，只需使用典型的dict API：

    >>> product.keys()
    ['price', 'name']

    >>> product.items()
    [('price', 1000), ('name', 'Desktop PC')]

#### 其他常见任务

复制项目：

    >>> product2 = Product(product)
    >>> print product2
    Product(name='Desktop PC', price=1000)

    >>> product3 = product2.copy()
    >>> print product3
    Product(name='Desktop PC', price=1000)

#### 从项目创建词典：

    >>> dict(product) # create a dict from all populated values
    {'price': 1000, 'name': 'Desktop PC'}

#### 从短片创建项目：

    >>> Product({'name': 'Laptop PC', 'price': 1500})
    Product(price=1500, name='Laptop PC')

    >>> Product({'name': 'Laptop PC', 'lala': 1500}) # warning: unknown field in dict
    Traceback (most recent call last):
        ...
    KeyError: 'Product does not support field: lala'

### 扩展项目

您可以通过声明原始项的子类来扩展项（以添加更多字段或更改某些字段的某些元数据）。

例如：

    class DiscountedProduct(Product):
        discount_percent = scrapy.Field(serializer=str)
        discount_expiration_date = scrapy.Field()

您还可以通过使用先前的字段元数据并附加更多值或更改现有值来扩展字段元数据，如下所示：

    class SpecificProduct(Product):
        name = scrapy.Field(Product.fields['name'], serializer=my_serializer)

添加（或替换）字段的serializer元数据键name，保留所有以前存在的元数据值。

#### 项目对象

`class scrapy.item.Item([arg])`
返回一个可以从给定参数初始化的新项目。

项目复制标准[dict API](https://docs.python.org/2/library/stdtypes.html#dict)，包括其构造函数。Items提供的唯一附加属性是：

`fields`
包含字典中的所有声明的字段为这个项目，不仅是那些填充。键是字段名称，值是`Field`在[项目声明中](http://scrapy.readthedocs.io/en/latest/topics/items.html#topics-items-declaring)使用的 对象。

#### 字段对象

`class scrapy.item.Field([arg])`
`Field`类只是一个别名内置字典类和不提供任何额外的功能或属性。换句话说， `Field`对象是普通的`Python`代码。单独的类用于支持 基于类属性的项声明语法。
