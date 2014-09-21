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

另外，在spider运行结束后，您必须自行关闭Twisted reactor。
这可以通过在 :meth:`CrawlerRunner.crawl
<scrapy.crawler.CrawlerRunner.crawl>` 所返回的对象中添加回调函数来实现。

下面给出了如何实现的例子，使用 `testspiders`_ 项目作为例子。

::

    from twisted.internet import reactor
    from scrapy.crawler import CrawlerRunner
    from scrapy.utils.project import get_project_settings

    runner = CrawlerRunner(get_project_settings())

    # 'followall' is the name of one of the spiders of the project. 
    d = runner.crawl('followall', domain='scrapinghub.com')
    d.addBoth(lambda _: reactor.stop())
    reactor.run() # the script will block here until the crawling is finished

Running spiders outside projects it's not much different. You have to create a
generic :class:`~scrapy.settings.Settings` object and populate it as needed
(See :ref:`topics-settings-ref` for the available settings), instead of using
the configuration returned by `get_project_settings`.

Spiders can still be referenced by their name if :setting:`SPIDER_MODULES` is
set with the modules where Scrapy should look for spiders.  Otherwise, passing
the spider class as first argument in the :meth:`CrawlerRunner.crawl
<scrapy.crawler.CrawlerRunner.crawl>` method is enough.

::

    from twisted.internet import reactor
    from scrapy.spider import Spider
    from scrapy.crawler import CrawlerRunner
    from scrapy.settings import Settings

    class MySpider(Spider):
        # Your spider definition
        ...

    settings = Settings({'USER_AGENT': 'Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)'})
    runner = CrawlerRunner(settings)

    d = runner.crawl(MySpider)
    d.addBoth(lambda _: reactor.stop())
    reactor.run() # the script will block here until the crawling is finished

.. seealso:: `Twisted Reactor Overview`_.

同一进程运行多个spider
============================================

默认情况下，当您执行 ``scrapy crawl`` 时，Scrapy每个进程运行一个spider。
当然，Scrapy通过
:ref:`内部(internal)API <topics-api>`
也支持单进程多个spider。

下面以 `testspiders`_ 作为例子来说明如何同时运行多个spider:

::

    from twisted.internet import reactor, defer
    from scrapy.crawler import CrawlerRunner
    from scrapy.utils.project import get_project_settings

    runner = CrawlerRunner(get_project_settings())
    dfs = set()
    for domain in ['scrapinghub.com', 'insophia.com']:
        d = runner.crawl('followall', domain=domain)
        dfs.add(d)

    defer.DeferredList(dfs).addBoth(lambda _: reactor.stop())
    reactor.run() # the script will block here until all crawling jobs are finished

相同的例子，不过通过链接(chaining) deferred来线性运行spider:

::

    from twisted.internet import reactor, defer
    from scrapy.crawler import CrawlerRunner
    from scrapy.utils.project import get_project_settings

    runner = CrawlerRunner(get_project_settings())

    @defer.inlineCallbacks
    def crawl():
        for domain in ['scrapinghub.com', 'insophia.com']:
            yield runner.crawl('followall', domain=domain)
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
.. _Crawlera: http://crawlera.com

.. _dynamic-item-classes:

动态创建Item类
================================

对于有些应用，item的结构由用户输入或者其他变化的情况所控制。您可以动态创建class。

::


	from scrapy.item import DictItem, Field

	def create_item_class(class_name, field_list):
	    field_dict = {}
	    for field_name in field_list:
	        field_dict[field_name] = Field()

	    return type(class_name, (DictItem,), field_dict)
