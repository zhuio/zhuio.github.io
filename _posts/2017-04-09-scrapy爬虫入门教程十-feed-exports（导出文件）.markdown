---
layout: "post"
title: "Scrapy爬虫入门教程十 Feed exports（导出文件）"
date: "2017-04-09 14:33"
tags:
- Scrapy
comments: true
---

# Scrapy爬虫入门教程十 Feed exports（导出文件）


**开发环境：**
`Python 3.6.0 版本` （当前最新）
`Scrapy 1.3.2 版本` （当前最新）

# 导出文件

新版本0.10。

实现爬虫时最常需要的特征之一是能够正确地存储所过滤的数据，并且经常意味着使用被过滤的数据（通常称为“export feed”）生成要由其他系统消耗的“导出文件” 。

Scrapy使用Feed导出功能即时提供此功能，这允许您使用多个序列化格式和存储后端来生成包含已抓取项目的Feed。

## 序列化格式

为了序列化抓取的数据，Feed导出使用项导出器。这些格式是开箱即用的：

- [JSON](http://scrapy.readthedocs.io/en/latest/topics/feed-exports.html#topics-feed-format-json)
- [JSON lines](http://scrapy.readthedocs.io/en/latest/topics/feed-exports.html#topics-feed-format-jsonlines)
- [CSV](http://scrapy.readthedocs.io/en/latest/topics/feed-exports.html#topics-feed-format-csv)
- [XML](http://scrapy.readthedocs.io/en/latest/topics/feed-exports.html#topics-feed-format-xml)

但您也可以通过FEED_EXPORTERS设置扩展支持的格式 。

### JSON

- FEED_FORMAT： json
- 使用出口： JsonItemExporter
- 如果您对大型Feed使用JSON，请参阅[此警告](http://scrapy.readthedocs.io/en/latest/topics/exporters.html#json-with-large-data)。

### JSON lines

- FEED_FORMAT： jsonlines
- 使用出口： JsonLinesItemExporter

### CSV

- FEED_FORMAT： csv
- 使用出口： CsvItemExporter
- 指定要导出的列及其顺序使用 FEED_EXPORT_FIELDS。其他Feed导出程序也可以使用此选项，但它对CSV很重要，因为与许多其他导出格式不同，CSV使用固定标头。

### XML

- FEED_FORMAT： xml
- 使用出口： XmlItemExporter

### Pickle

- FEED_FORMAT： pickle
- 使用出口： PickleItemExporter

### Marshal

- FEED_FORMAT： marshal
- 使用出口： MarshalItemExporter

---

## 存储

使用Feed导出时，您可以使用[URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier)（通过FEED_URI设置）定义在哪里存储Feed 。Feed导出支持由URI方案定义的多个存储后端类型。

支持开箱即用的存储后端包括：

- [本地文件系统](http://scrapy.readthedocs.io/en/latest/topics/feed-exports.html#topics-feed-storage-fs)
- [FTP](http://scrapy.readthedocs.io/en/latest/topics/feed-exports.html#topics-feed-storage-ftp)
- [S3](http://scrapy.readthedocs.io/en/latest/topics/feed-exports.html#topics-feed-storage-s3)（需要 [botocore](https://github.com/boto/botocore)或 [boto](https://github.com/boto/boto)）
- [标准输出](http://scrapy.readthedocs.io/en/latest/topics/feed-exports.html#topics-feed-storage-stdout)

如果所需的外部库不可用，则某些存储后端可能无法使用。例如，S3后端仅在安装了[botocore](https://github.com/boto/botocore) 或[boto](https://github.com/boto/boto)库时可用（Scrapy仅支持[boto](https://github.com/boto/boto)到Python 2）。

---

## 存储URI参数

存储URI还可以包含在创建订阅源时被替换的参数。这些参数是：

- %(time)s - 在创建订阅源时由时间戳替换
- %(name)s - 被蜘蛛名替换

任何其他命名参数将替换为同名的spider属性。例如， 在创建订阅源的那一刻，`%(site_id)s`将被`spider.site_id`属性替换。

这里有一些例子来说明：

- 存储在FTP中使用每个蜘蛛一个目录：
- `ftp://user:password@ftp.example.com/scraping/feeds/%(name)s/%(time)s.json`

- 存储在S3使用每个蜘蛛一个目录：
- `s3://mybucket/scraping/feeds/%(name)s/%(time)s.json`

## 存储后端

### 本地文件系统

订阅源存储在本地文件系统中。

URI方案： `file`
示例URI： `file:///tmp/export.csv`
所需的外部库：none
**请注意**，（仅）对于本地文件系统存储，如果指定绝对路径，则可以省略该方案`/tmp/export.csv`。这只适用于Unix系统。

### FTP

订阅源存储在FTP服务器中。

- URI方案： `ftp`
- 示例URI： `ftp://user:pass@ftp.example.com/`path/to/export.csv
- 所需的外部库：none

### S3

订阅源存储在[Amazon S3](https://aws.amazon.com/s3/)上。

- URI方案： `s3`
- 示例URI：
- `s3://mybucket/path/to/export.csv`
- `s3://aws_key:aws_secret@mybucket/path/to/export.csv`

- 所需的外部库：botocore或boto

AWS凭证可以作为URI中的用户/密码传递，也可以通过以下设置传递：

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

### 标准输出

Feed被写入Scrapy进程的标准输出。

- URI方案： `stdout`
- 示例URI： `stdout:`
- 所需的外部库：none

## 设置

这些是用于配置Feed导出的设置：

- FEED_URI （强制性）
- FEED_FORMAT
- FEED_STORAGES
- FEED_EXPORTERS
- FEED_STORE_EMPTY
- FEED_EXPORT_ENCODING
- FEED_EXPORT_FIELDS

### FEED_URI

默认： None

导出Feed的URI。请参阅支持的URI方案的存储后端。

启用Feed导出时需要此设置。

### FEED_FORMAT

要用于Feed的序列化格式。有关可能的值，请参阅 序列化格式。

### FEED_EXPORT_ENCODING

默认： None

要用于Feed的编码。

如果取消设置或设置为None（默认），它使用UTF-8除了JSON输出，\uXXXX由于历史原因使用安全的数字编码（序列）。

使用utf-8，如果你想UTF-8 JSON了。

### FEED_EXPORT_FIELDS

默认： None

要导出的字段的列表，可选。示例：。FEED_EXPORT_FIELDS = ["foo", "bar", "baz"]

使用FEED_EXPORT_FIELDS选项定义要导出的字段及其顺序。

当FEED_EXPORT_FIELDS为空或无（默认）时，Scrapy使用在Item蜘蛛正在产生的dicts 或子类中定义的字段。

如果导出器需要一组固定的字段（CSV导出格式为这种情况 ），并且FEED_EXPORT_FIELDS为空或无，则Scrapy会尝试从导出的​​数据中推断字段名称 - 当前它使用第一个项目中的字段名称。

### FEED_STORE_EMPTY

默认： False

是否导出空Feed（即，没有项目的Feed）。

FEED_STORAGES
默认： {}

包含您的项目支持的其他Feed存储后端的字典。键是URI方案，值是存储类的路径。

### FEED_STORAGES_BASE

默认：

    {
        '': 'scrapy.extensions.feedexport.FileFeedStorage',
        'file': 'scrapy.extensions.feedexport.FileFeedStorage',
        'stdout': 'scrapy.extensions.feedexport.StdoutFeedStorage',
        's3': 'scrapy.extensions.feedexport.S3FeedStorage',
        'ftp': 'scrapy.extensions.feedexport.FTPFeedStorage',
    }

包含Scrapy支持的内置Feed存储后端的字典。您可以通过分配其中None的URI方案 来禁用这些后端FEED_STORAGES。例如，要禁用内置FTP存储后端（无替换），请将其放置在settings.py：

    FEED_STORAGES = {
        'ftp': None,
    }

### FEED_EXPORTERS

默认： {}

包含您的项目支持的其他导出器的字典。键是序列化格式，值是Item exporter类的路径。

### FEED_EXPORTERS_BASE

默认：

    {
        'json': 'scrapy.exporters.JsonItemExporter',
        'jsonlines': 'scrapy.exporters.JsonLinesItemExporter',
        'jl': 'scrapy.exporters.JsonLinesItemExporter',
        'csv': 'scrapy.exporters.CsvItemExporter',
        'xml': 'scrapy.exporters.XmlItemExporter',
        'marshal': 'scrapy.exporters.MarshalItemExporter',
        'pickle': 'scrapy.exporters.PickleItemExporter',
    }

一个包含Scrapy支持的内置feed导出器的dict。您可以通过分配其中None的序列化格式来禁用任何这些导出器FEED_EXPORTERS。例如，要禁用内置的CSV导出器（无替换），请将其放置在settings.py：

    FEED_EXPORTERS = {
        'csv': None,
    }
