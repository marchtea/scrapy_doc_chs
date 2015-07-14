.. _topics-practices:

===========================
实践经验(Common Practices)
===========================

本章节记录了使用Scrapy的一些实践经验(common practices)。
这包含了很多使用不会包含在其他特定章节的的内容。

.. _run-from-script:

在脚本中运行Scrapy
========================

除了常用的 ``scrapy crawl`` 来启动Scrapy，您也可以使用 :ref:`API <topics-api>` 在脚本中启动Scrapy。

需要注意的是，Scrapy是在Twisted异步网络库上构建的，
因此其必须在Twisted reactor里运行。

First utility you can use to run your spiders is
:class:`scrapy.crawler.CrawlerProcess`. This class will start a Twisted reactor
for you, configuring the logging and setting shutdown handlers. This class is
the one used by all Scrapy commands.

Here's an example showing how to run a single spider with it.

::

    import scrapy
    from scrapy.crawler import CrawlerProcess

    class MySpider(scrapy.Spider):
        # Your spider definition
        ...

    process = CrawlerProcess({
        'USER_AGENT': 'Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)'
    })

    process.crawl(MySpider)
    process.start() # the script will block here until the crawling is finished

Make sure to check :class:`~scrapy.crawler.CrawlerProcess` documentation to get
acquainted with its usage details.

If you are inside a Scrapy project there are some additional helpers you can
use to import those components within the project. You can automatically import
your spiders passing their name to :class:`~scrapy.crawler.CrawlerProcess`, and
use ``get_project_settings`` to get a :class:`~scrapy.settings.Settings`
instance with your project settings.

下面给出了如何实现的例子，使用 `testspiders`_ 项目作为例子。

::

    from scrapy.crawler import CrawlerProcess
    from scrapy.utils.project import get_project_settings

    process = CrawlerProcess(get_project_settings())

    # 'followall' is the name of one of the spiders of the project.
    process.crawl('testspider', domain='scrapinghub.com')
    process.start() # the script will block here until the crawling is finished

There's another Scrapy utility that provides more control over the crawling
process: :class:`scrapy.crawler.CrawlerRunner`. This class is a thin wrapper
that encapsulates some simple helpers to run multiple crawlers, but it won't
start or interfere with existing reactors in any way.

Using this class the reactor should be explicitly run after scheduling your
spiders. It's recommended you use :class:`~scrapy.crawler.CrawlerRunner`
instead of :class:`~scrapy.crawler.CrawlerProcess` if your application is
already using Twisted and you want to run Scrapy in the same reactor.

Note that you will also have to shutdown the Twisted reactor yourself after the
spider is finished. This can be achieved by adding callbacks to the deferred
returned by the :meth:`CrawlerRunner.crawl
<scrapy.crawler.CrawlerRunner.crawl>` method.

Here's an example of its usage, along with a callback to manually stop the
reactor after `MySpider` has finished running.

::

    from twisted.internet import reactor
    import scrapy
    from scrapy.crawler import CrawlerRunner
    from scrapy.utils.log import configure_logging

    class MySpider(scrapy.Spider):
        # Your spider definition
        ...

    configure_logging({'LOG_FORMAT': '%(levelname)s: %(message)s'})
    runner = CrawlerRunner()

    d = runner.crawl(MySpider)
    d.addBoth(lambda _: reactor.stop())
    reactor.run() # the script will block here until the crawling is finished

.. seealso:: `Twisted Reactor Overview`_.

.. _run-multiple-spiders:

同一进程运行多个spider
============================================

默认情况下，当您执行 ``scrapy crawl`` 时，Scrapy每个进程运行一个spider。
当然，Scrapy通过
:ref:`内部(internal)API <topics-api>`
也支持单进程多个spider。

Here is an example that runs multiple spiders simultaneously:

::

    import scrapy
    from scrapy.crawler import CrawlerProcess

    class MySpider1(scrapy.Spider):
        # Your first spider definition
        ...

    class MySpider2(scrapy.Spider):
        # Your second spider definition
        ...

    process = CrawlerProcess()
    process.crawl(MySpider1)
    process.crawl(MySpider2)
    process.start() # the script will block here until all crawling jobs are finished

Same example using :class:`~scrapy.crawler.CrawlerRunner`:

::

    import scrapy
    from twisted.internet import reactor
    from scrapy.crawler import CrawlerRunner
    from scrapy.utils.log import configure_logging

    class MySpider1(scrapy.Spider):
        # Your first spider definition
        ...

    class MySpider2(scrapy.Spider):
        # Your second spider definition
        ...

    configure_logging()
    runner = CrawlerRunner()
    runner.crawl(MySpider1)
    runner.crawl(MySpider2)
    d = runner.join()
    d.addBoth(lambda _: reactor.stop())

    reactor.run() # the script will block here until all crawling jobs are finished

Same example but running the spiders sequentially by chaining the deferreds:

::

    from twisted.internet import reactor, defer
    from scrapy.crawler import CrawlerRunner
    from scrapy.utils.log import configure_logging

    class MySpider1(scrapy.Spider):
        # Your first spider definition
        ...

    class MySpider2(scrapy.Spider):
        # Your second spider definition
        ...

    configure_logging()
    runner = CrawlerRunner()

    @defer.inlineCallbacks
    def crawl():
        yield runner.crawl(MySpider1)
        yield runner.crawl(MySpider2)
        reactor.stop()

    crawl()
    reactor.run() # the script will block here until the last crawl call is finished

.. seealso:: :ref:`run-from-script`.

.. _distributed-crawls:

分布式爬虫(Distributed crawls)
=================================

Scrapy并没有提供内置的机制支持分布式(多服务器)爬取。不过还是有办法进行分布式爬取，
取决于您要怎么分布了。

如果您有很多spider，那分布负载最简单的办法就是启动多个Scrapyd，并分配到不同机器上。

如果想要在多个机器上运行一个单独的spider，那您可以将要爬取的url进行分块，并发送给spider。
例如:

首先，准备要爬取的url列表，并分配到不同的文件\url里::

    http://somedomain.com/urls-to-crawl/spider1/part1.list
    http://somedomain.com/urls-to-crawl/spider1/part2.list
    http://somedomain.com/urls-to-crawl/spider1/part3.list

接着在3个不同的Scrapd服务器中启动spider。spider会接收一个(spider)参数 ``part`` ，
该参数表示要爬取的分块::

    curl http://scrapy1.mycompany.com:6800/schedule.json -d project=myproject -d spider=spider1 -d part=1
    curl http://scrapy2.mycompany.com:6800/schedule.json -d project=myproject -d spider=spider1 -d part=2
    curl http://scrapy3.mycompany.com:6800/schedule.json -d project=myproject -d spider=spider1 -d part=3

.. _bans:

避免被禁止(ban)
=======================

有些网站实现了特定的机制，以一定规则来避免被爬虫爬取。
与这些规则打交道并不容易，需要技巧，有时候也需要些特别的基础。
如果有疑问请考虑联系 `商业支持`_ 。

下面是些处理这些站点的建议(tips):

* 使用user agent池，轮流选择之一来作为user agent。池中包含常见的浏览器的user agent(google一下一大堆)
* 禁止cookies(参考 :setting:`COOKIES_ENABLED`)，有些站点会使用cookies来发现爬虫的轨迹。
* 设置下载延迟(2或更高)。参考 :setting:`DOWNLOAD_DELAY` 设置。
* 如果可行，使用 `Google cache`_ 来爬取数据，而不是直接访问站点。
* 使用IP池。例如免费的 `Tor项目`_ 或付费服务(`ProxyMesh`_)。
* 使用高度分布式的下载器(downloader)来绕过禁止(ban)，您就只需要专注分析处理页面。这样的例子有:
  `Crawlera`_

如果您仍然无法避免被ban，考虑联系
`商业支持`_.

.. _Tor项目: https://www.torproject.org/
.. _商业支持: http://scrapy.org/support/
.. _ProxyMesh: http://proxymesh.com/
.. _Google cache: http://www.googleguide.com/cached_pages.html
.. _testspiders: https://github.com/scrapinghub/testspiders
.. _Twisted Reactor Overview: http://twistedmatrix.com/documents/current/core/howto/reactor-basics.html
.. _Crawlera: http://scrapinghub.com/crawlera

.. _dynamic-item-classes:
