.. _topics-stats:

==============================
数据收集(Stats Collection)
==============================

Scrapy提供了方便的收集数据的机制。数据以key/value方式存储，值大多是计数值。
该机制叫做数据收集器(Stats Collector)，可以通过
:ref:`topics-api-crawler` 的属性 :attr:`~scrapy.crawler.Crawler.stats`
来使用。在下面的章节
:ref:`topics-stats-usecases` 将给出例子来说明。

无论数据收集(stats collection)开启或者关闭，数据收集器永远都是可用的。
因此您可以import进自己的模块并使用其API(增加值或者设置新的状态键(stat keys))。
该做法是为了简化数据收集的方法: 您不应该使用超过一行代码来收集您的spider，Scrpay扩展或任何您使用状态收集器代码里头的状态。

状态收集器的另一个特性是(在启用状态下)很高效，(在关闭情况下)非常高效(几乎察觉不到)。

状态收集器对每个spider保持一个状态表。当spider启动时，该表自动打开，当spider关闭时，自动关闭。

.. _topics-stats-usecases:

常见状态收集器使用方法
===========================

通过 :attr:`~scrapy.crawler.Crawler.stats` 属性来使用状态收集器。
下面是在扩展中使用状态的例子::

    class ExtensionThatAccessStats(object):

        def __init__(self, stats):
            self.stats = stats

        @classmethod
        def from_crawler(cls, crawler):
            return cls(crawler.stats)

设置状态::

    stats.set_value('hostname', socket.gethostname())

增加状态值::

    stats.inc_value('pages_crawled')

当新的值比原来的值大时设置状态::

    stats.max_value('max_items_scraped', value)

当新的值比原来的值小时设置状态::

    stats.min_value('min_free_memory_percent', value)

获取状态::

    >>> stats.get_value('pages_crawled')
    8

获取所有状态::

    >>> stats.get_stats()
    {'pages_crawled': 1238, 'start_time': datetime.datetime(2009, 7, 14, 21, 47, 28, 977139)}

可用的状态收集器
==========================

除了基本的 :class:`StatsCollector` ，Scrapy也提供了基于 :class:`StatsCollector` 的状态收集器。
您可以通过 :setting:`STATS_CLASS` 设置来选择。默认使用的是
:class:`MemoryStatsCollector` 。

.. module:: scrapy.statscol
   :synopsis: Stats Collectors

MemoryStatsCollector
--------------------

.. class:: MemoryStatsCollector

    一个简单的状态收集器。其在spider运行完毕后将其状态保存在内存中。状态可以通过
    :attr:`spider_stats` 属性访问。该属性是一个以spider名字为键(key)的字典。

    这是Scrapy的默认选择。

    .. attribute:: spider_stats

       保存了每个spider最近一次爬取的状态的字典(dict)。该字典以spider名字为键，值也是字典。

DummyStatsCollector
-------------------

.. class:: DummyStatsCollector

    该状态收集器并不做任何事情但非常高效(因为什么都不做(写文档的人真调皮o(╯□╰)o))。
    您可以通过设置 :setting:`STATS_CLASS` 启用这个收集器，来关闭状态收集，提高效率。
    不过，状态收集的性能负担相较于Scrapy其他的处理(例如分析页面)来说是非常小的。
