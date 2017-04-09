---
layout: "post"
title: "Scrapy爬虫入门教程九 Item Pipeline（项目管道）"
date: "2017-04-09 14:32"
tags:
- Scrapy
comments: true
---

# Scrapy爬虫入门教程九 Item Pipeline（项目管道）


**开发环境：**
`Python 3.6.0 版本` （当前最新）
`Scrapy 1.3.2 版本` （当前最新）

# Item Pipeline（项目管道）

在项目被蜘蛛抓取后，它被发送到项目管道，它通过顺序执行的几个组件来处理它。

每个项目管道组件（有时称为“Item Pipeline”）是一个实现简单方法的Python类。他们接收一个项目并对其执行操作，还决定该项目是否应该继续通过流水线或被丢弃并且不再被处理。

项目管道的典型用途是：

- 清理HTML数据
- 验证抓取的数据（检查项目是否包含特定字段）
- 检查重复（并删除）
- 将刮取的项目存储在数据库中

## 编写自己的项目管道

每个项目管道组件是一个Python类，必须实现以下方法：
`process_item(self, item, spider)`

对于每个项目管道组件调用此方法。process_item() 必须：返回一个带数据的dict，返回一个Item （或任何后代类）对象，返回一个Twisted Deferred或者raise DropItemexception。丢弃的项目不再由其他管道组件处理。

**参数：**

- item（Itemobject或dict） - 剪切的项目
- Spider（Spider对象） - 抓取物品的蜘蛛

另外，它们还可以实现以下方法：

---

`open_spider(self, spider)`
当蜘蛛打开时调用此方法。

**参数：**

- 蜘蛛（Spider对象） - 打开的蜘蛛

---

`close_spider(self, spider)`
当蜘蛛关闭时调用此方法。

**参数：**

- 蜘蛛（Spider对象） - 被关闭的蜘蛛

---

`from_crawler(cls, crawler)`
如果存在，则调用此类方法以从a创建流水线实例Crawler。它必须返回管道的新实例。Crawler对象提供对所有Scrapy核心组件（如设置和信号）的访问; 它是管道访问它们并将其功能挂钩到Scrapy中的一种方式。

**参数：**

- crawler（Crawlerobject） - 使用此管道的crawler

---

## 项目管道示例

### 价格验证和丢弃项目没有价格

让我们来看看以下假设的管道，它调整 price那些不包括增值税（price_excludes_vat属性）的项目的属性，并删除那些不包含价格的项目：

    from scrapy.exceptions import DropItem

    class PricePipeline(object):

        vat_factor = 1.15

        def process_item(self, item, spider):
            if item['price']:
                if item['price_excludes_vat']:
                    item['price'] = item['price'] * self.vat_factor
                return item
            else:
                raise DropItem("Missing price in %s" % item)

将项目写入JSON文件
以下管道将所有抓取的项目（来自所有蜘蛛）存储到单个items.jl文件中，每行包含一个项目，以JSON格式序列化：

    import json

    class JsonWriterPipeline(object):

        def open_spider(self, spider):
            self.file = open('items.jl', 'wb')

        def close_spider(self, spider):
            self.file.close()

        def process_item(self, item, spider):
            line = json.dumps(dict(item)) + "\n"
            self.file.write(line)
            return item

**注意**

> JsonWriterPipeline的目的只是介绍如何编写项目管道。如果您真的想要将所有抓取的项目存储到JSON文件中，则应使用Feed导出。

---

### 将项目写入MongoDB

在这个例子中，我们使用[pymongo](https://api.mongodb.org/python/current/)将项目写入[MongoDB](https://www.mongodb.org/)。MongoDB地址和数据库名称在Scrapy设置中指定; MongoDB集合以item类命名。

这个例子的要点是显示如何使用`from_crawler()`方法和如何正确清理资源：

    import pymongo

    class MongoPipeline(object):

        collection_name = 'scrapy_items'

        def __init__(self, mongo_uri, mongo_db):
            self.mongo_uri = mongo_uri
            self.mongo_db = mongo_db

        @classmethod
        def from_crawler(cls, crawler):
            return cls(
                mongo_uri=crawler.settings.get('MONGO_URI'),
                mongo_db=crawler.settings.get('MONGO_DATABASE', 'items')
            )

        def open_spider(self, spider):
            self.client = pymongo.MongoClient(self.mongo_uri)
            self.db = self.client[self.mongo_db]

        def close_spider(self, spider):
            self.client.close()

        def process_item(self, item, spider):
            self.db[self.collection_name].insert(dict(item))
            return item

---

### 拍摄项目的屏幕截图

此示例演示如何从方法返回Deferredprocess_item()。它使用Splash来呈现项目网址的屏幕截图。Pipeline请求本地运行的Splash实例。在请求被下载并且Deferred回调触发后，它将项目保存到一个文件并将文件名添加到项目。

    import scrapy
    import hashlib
    from urllib.parse import quote


    class ScreenshotPipeline(object):
        """Pipeline that uses Splash to render screenshot of
        every Scrapy item."""

        SPLASH_URL = "http://localhost:8050/render.png?url={}"

        def process_item(self, item, spider):
            encoded_item_url = quote(item["url"])
            screenshot_url = self.SPLASH_URL.format(encoded_item_url)
            request = scrapy.Request(screenshot_url)
            dfd = spider.crawler.engine.download(request, spider)
            dfd.addBoth(self.return_item, item)
            return dfd

        def return_item(self, response, item):
            if response.status != 200:
                # Error happened, return item.
                return item

            # Save screenshot to file, filename will be hash of url.
            url = item["url"]
            url_hash = hashlib.md5(url.encode("utf8")).hexdigest()
            filename = "{}.png".format(url_hash)
            with open(filename, "wb") as f:
                f.write(response.body)

            # Store filename in item.
            item["screenshot_filename"] = filename
            return item

---

### 复制过滤器

用于查找重复项目并删除已处理的项目的过滤器。假设我们的项目具有唯一的ID，但是我们的蜘蛛会返回具有相同id的多个项目：

    from scrapy.exceptions import DropItem

    class DuplicatesPipeline(object):

        def __init__(self):
            self.ids_seen = set()

        def process_item(self, item, spider):
            if item['id'] in self.ids_seen:
                raise DropItem("Duplicate item found: %s" % item)
            else:
                self.ids_seen.add(item['id'])
                return item

---

## 激活项目管道组件

要激活项目管道组件，必须将其类添加到 ITEM_PIPELINES设置，类似于以下示例：

    ITEM_PIPELINES = {
        'myproject.pipelines.PricePipeline': 300,
        'myproject.pipelines.JsonWriterPipeline': 800,
    }

您在此设置中分配给类的整数值确定它们运行的​​顺序：项目从较低值到较高值类。通常将这些数字定义在0-1000范围内。
