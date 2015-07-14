.. _topics-item-pipeline:

=============
Item Pipeline
=============

当Item在Spider中被收集之后，它将会被传递到Item Pipeline，一些组件会按照一定的顺序执行对Item的处理。

每个item pipeline组件(有时称之为“Item Pipeline”)是实现了简单方法的Python类。他们接收到Item并通过它执行一些行为，同时也决定此Item是否继续通过pipeline，或是被丢弃而不再进行处理。

以下是item pipeline的一些典型应用：

* 清理HTML数据
* 验证爬取的数据(检查item包含某些字段)
* 查重(并丢弃)
* 将爬取结果保存到数据库中


编写你自己的item pipeline
==============================

每个item pipiline组件是一个独立的Python类，同时必须实现以下方法:

.. method:: process_item(self, item, spider)

   每个item pipeline组件都需要调用该方法，这个方法必须返回一个具有数据的dict，或是 :class:`~scrapy.item.Item` (或任何继承类)对象，
   或是抛出 :exc:`~scrapy.exceptions.DropItem` 异常，被丢弃的item将不会被之后的pipeline组件所处理。


   :param item: 被爬取的item
   :type item: :class:`~scrapy.item.Item` 对象或者一个dict

   :param spider: 爬取该item的spider
   :type spider: :class:`~scrapy.spiders.Spider` 对象


此外,他们也可以实现以下方法:

.. method:: open_spider(self, spider)

   当spider被开启时，这个方法被调用。

   :param spider: 被开启的spider
   :type spider: :class:`~scrapy.spiders.Spider` 对象

.. method:: close_spider(self, spider)

   当spider被关闭时，这个方法被调用

   :param spider: 被关闭的spider
   :type spider: :class:`~scrapy.spiders.Spider` 对象

.. method:: from_crawler(cls, crawler)

   If present, this classmethod is called to create a pipeline instance
   from a :class:`~scrapy.crawler.Crawler`. It must return a new instance
   of the pipeline. Crawler object provides access to all Scrapy core
   components like settings and signals; it is a way for pipeline to
   access them and hook its functionality into Scrapy.

   :param crawler: crawler that uses this pipeline
   :type crawler: :class:`~scrapy.crawler.Crawler` object


Item pipeline 样例
=====================

验证价格，同时丢弃没有价格的item
--------------------------------------------------

让我们来看一下以下这个假设的pipeline，它为那些不含税(``price_excludes_vat`` 属性)的item调整了 ``price`` 属性，同时丢弃了那些没有价格的item::

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


将item写入JSON文件
--------------------------

以下pipeline将所有(从所有spider中)爬取到的item，存储到一个独立地 ``items.jl`` 文件，每行包含一个序列化为JSON格式的item::

   import json

   class JsonWriterPipeline(object):

       def __init__(self):
           self.file = open('items.jl', 'wb')

       def process_item(self, item, spider):
           line = json.dumps(dict(item)) + "\n"
           self.file.write(line)
           return item

.. note:: JsonWriterPipeline的目的只是为了介绍怎样编写item pipeline，如果你想要将所有爬取的item都保存到同一个JSON文件，
    你需要使用 :ref:`Feed exports <topics-feed-exports>` 。

Write items to MongoDB
----------------------

In this example we'll write items to MongoDB_ using pymongo_.
MongoDB address and database name are specified in Scrapy settings;
MongoDB collection is named after item class.

The main point of this example is to show how to use :meth:`from_crawler`
method and how to clean up the resources properly.

.. note::

    Previous example (JsonWriterPipeline) doesn't clean up resources properly.
    Fixing it is left as an exercise for the reader.

::

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

.. _MongoDB: http://www.mongodb.org/
.. _pymongo: http://api.mongodb.org/python/current/


去重
-----------------

一个用于去重的过滤器，丢弃那些已经被处理过的item。让我们假设我们的item有一个唯一的id，但是我们spider返回的多个item中包含有相同的id::


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

启用一个Item Pipeline组件
=====================================

为了启用一个Item Pipeline组件，你必须将它的类添加到 :setting:`ITEM_PIPELINES` 配置，就像下面这个例子::

   ITEM_PIPELINES = {
       'myproject.pipelines.PricePipeline': 300,
       'myproject.pipelines.JsonWriterPipeline': 800,
   }

分配给每个类的整型值，确定了他们运行的顺序，item按数字从低到高的顺序，通过pipeline，通常将这些数字定义在0-1000范围内。

