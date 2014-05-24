.. _topics-debug:

==========================
调试(Debugging)Spiders
==========================

本篇介绍了调试spider的常用技术。
考虑下面的spider::

    import scrapy
    from myproject.items import MyItem

    class MySpider(scrapy.Spider):
        name = 'myspider'
        start_urls = (
            'http://example.com/page1',
            'http://example.com/page2',
            )

        def parse(self, response):
            # collect `item_urls`
            for item_url in item_urls:
                yield scrapy.Request(item_url, self.parse_item)

        def parse_item(self, response):
            item = MyItem()
            # populate `item` fields
            # and extract item_details_url
            yield scrapy.Request(item_details_url, self.parse_details, meta={'item': item})

        def parse_details(self, response):
            item = response.meta['item']
            # populate more `item` fields
            return item

简单地说，该spider分析了两个包含item的页面(start_urls)。Item有详情页面，
所以我们使用 :class:`~scrapy.http.Request` 的 ``meta`` 功能来传递已经部分获取的item。


Parse命令
=============

检查spier输出的最基本方法是使用
:command:`parse` 命令。这能让你在函数层(method level)上检查spider
各个部分的效果。其十分灵活并且易用，不过不能在代码中调试。

查看特定url爬取到的item::

    $ scrapy parse --spider=myspider -c parse_item -d 2 <item_url>
    [ ... scrapy log lines crawling example.com spider ... ]

    >>> STATUS DEPTH LEVEL 2 <<<
    # Scraped Items  ------------------------------------------------------------
    [{'url': <item_url>}]

    # Requests  -----------------------------------------------------------------
    []

使用 ``--verbose`` 或 ``-v`` 选项，查看各个层次的状态::

    $ scrapy parse --spider=myspider -c parse_item -d 2 -v <item_url>
    [ ... scrapy log lines crawling example.com spider ... ]

    >>> DEPTH LEVEL: 1 <<<
    # Scraped Items  ------------------------------------------------------------
    []

    # Requests  -----------------------------------------------------------------
    [<GET item_details_url>]


    >>> DEPTH LEVEL: 2 <<<
    # Scraped Items  ------------------------------------------------------------
    [{'url': <item_url>}]

    # Requests  -----------------------------------------------------------------
    []

检查从单个start_url爬取到的item也是很简单的::

    $ scrapy parse --spider=myspider -d 3 'http://example.com/page1'


Scrapy终端(Shell)
=======================

尽管 :command:`parse` 命令对检查spider的效果十分有用，但除了显示收到的response及输出外，
其对检查回调函数内部的过程并没有提供什么便利。
如何调试 ``parse_detail`` 没有收到item的情况呢？

幸运的是，救世主 :command:`shell` 出现了(参考
:ref:`topics-shell-inspect-response`)::

    from scrapy.shell import inspect_response

    def parse_details(self, response):
        item = response.meta.get('item', None)
        if item:
            # populate more `item` fields
            return item
        else:
            inspect_response(response, self)

参考 :ref:`topics-shell-inspect-response` 。

在浏览器中打开
===============

有时候您想查看某个response在浏览器中显示的效果，这是可以使用
``open_in_browser`` 功能。下面是使用的例子::

    from scrapy.utils.response import open_in_browser

    def parse_details(self, response):
        if "item name" not in response.body:
            open_in_browser(response)

``open_in_browser`` 将会使用Scrapy获取到的response来打开浏览器，并且调整
`base tag`_ 使得图片及样式(style)能正常显示。

Logging
=======

记录(logging)是另一个获取到spider运行信息的方法。虽然不是那么方便，
但好处是log的内容在以后的运行中也可以看到::

    from scrapy import log

    def parse_details(self, response):
        item = response.meta.get('item', None)
        if item:
            # populate more `item` fields
            return item
        else:
            self.log('No item received for %s' % response.url,
                level=log.WARNING)

更多内容请参见 :ref:`topics-logging` 部分。

.. _base tag: http://www.w3schools.com/tags/tag_base.asp
