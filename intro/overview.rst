.. _intro-overview:

==================
初窥Scrapy
==================

Scrapy是一个为了爬取网站数据，提取结构性数据而编写的应用框架。
可以应用在包括数据挖掘，信息处理或存储历史数据等一系列的程序中。

其最初是为了 `页面抓取`_ (更确切来说, `网络抓取`_ )所设计的，
也可以应用在获取API所返回的数据(例如 `Amazon Associates Web Services`_ ) 或者通用的网络爬虫。

本文档将通过介绍Scrapy背后的概念使您对其工作原理有所了解，
并确定Scrapy是否是您所需要的。

当您准备好开始您的项目后，您可以参考 :ref:`入门教程<intro-tutorial>` 。

选择一个网站
==============

当您需要从某个网站中获取信息，但该网站未提供API或能通过程序获取信息的机制时，
Scrapy可以助你一臂之力。

以 `Mininova`_ 网站为例，我们想要获取今日添加的所有种子的URL、
名字、描述以及文件大小信息。

今日添加的种子列表可以通过这个页面找到:

    http://www.mininova.org/today

.. _intro-overview-item:

定义您想抓取的数据
==================================

第一步是定义我们需要爬取的数据。在Scrapy中，
这是通过 :ref:`Scrapy Items <topics-items>` 来完成的。(在本例子中为种子文件)

我们定义的Item::

    import scrapy

    class TorrentItem(scrapy.Item):
        url = scrapy.Field()
        name = scrapy.Field()
        description = scrapy.Field()
        size = scrapy.Field()

编写提取数据的Spider
==================================

第二步是编写一个spider。其定义了初始URL(http://www.mininova.org/today)、
针对后续链接的规则以及从页面中提取数据的规则。

通过观察页面的内容可以发现，所有种子的URL都类似 ``http://www.mininova.org/tor/NUMBER`` 。
其中， ``NUMBER`` 是一个整数。
根据此规律，我们可以定义需要进行跟进的链接的正则表达式: ``/tor/\d+`` 。

我们使用 `XPath`_ 来从页面的HTML源码中选择需要提取的数据。
以其中一个种子文件的页面为例:

    http://www.mininova.org/tor/2676093

观察HTML页面源码并创建我们需要的数据(种子名字，描述和大小)的XPath表达式。

.. highlight:: html

通过观察，我们可以发现文件名是包含在 ``<h1>`` 标签中的::

   <h1>Darwin - The Evolution Of An Exhibition</h1>

.. highlight:: none

与此对应的XPath表达式::

    //h1/text()

.. highlight:: html

种子的描述是被包含在 ``id="description"`` 的 ``<div>`` 标签中::

   <h2>Description:</h2>

   <div id="description">
   Short documentary made for Plymouth City Museum and Art Gallery regarding the setup of an exhibit about Charles Darwin in conjunction with the 200th anniversary of his birth.

   ...

.. highlight:: none

对应获取描述的XPath表达式::

    //div[@id='description']

.. highlight:: html

文件大小的信息包含在 ``id=specifications`` 的 ``<div>`` 的第二个 ``<p>`` 标签中::

   <div id="specifications">

   <p>
   <strong>Category:</strong>
   <a href="/cat/4">Movies</a> &gt; <a href="/sub/35">Documentary</a>
   </p>

   <p>
   <strong>Total size:</strong>
   150.62&nbsp;megabyte</p>


.. highlight:: none

选择文件大小的XPath表达式::

   //div[@id='specifications']/p[2]/text()[2]

.. highlight:: python

关于XPath的详细内容请参考 `XPath参考`_ 。

最后，结合以上内容给出spider的代码::

    from scrapy.contrib.spiders import CrawlSpider, Rule
    from scrapy.contrib.linkextractors import LinkExtractor

    class MininovaSpider(CrawlSpider):

        name = 'mininova'
        allowed_domains = ['mininova.org']
        start_urls = ['http://www.mininova.org/today']
        rules = [Rule(LinkExtractor(allow=['/tor/\d+']), 'parse_torrent')]

        def parse_torrent(self, response):
            torrent = TorrentItem()
            torrent['url'] = response.url
            torrent['name'] = response.xpath("//h1/text()").extract()
            torrent['description'] = response.xpath("//div[@id='description']").extract()
            torrent['size'] = response.xpath("//div[@id='specifications']/p[2]/text()[2]").extract()
            return torrent

``TorrentItem`` 的定义在 :ref:`上面 <intro-overview-item>` 。

执行spider，获取数据
==================================

终于，我们可以运行spider来获取网站的数据，并以JSON格式存入到 
``scraped_data.json`` 文件中::

    scrapy crawl mininova -o scraped_data.json

命令中使用了 :ref:`feed导出 <topics-feed-exports>` 来导出JSON文件。您可以修改导出格式(XML或者CSV)或者存储后端(FTP或者 `Amazon S3`_)，这并不困难。

同时，您也可以编写 :ref:`item管道 <topics-item-pipeline>` 将item存储到数据库中。

查看提取到的数据
===================

执行结束后，当您查看 ``scraped_data.json`` , 您将看到提取到的item::

    [{"url": "http://www.mininova.org/tor/2676093", "name": ["Darwin - The Evolution Of An Exhibition"], "description": ["Short documentary made for Plymouth ..."], "size": ["150.62 megabyte"]},
    # ... other items ...
    ]

由于 :ref:`selectors <topics-selectors>` 返回list, 所以值都是以list存储的(除了 ``url`` 是直接赋值之外)。 
如果您想要保存单个数据或者对数据执行额外的处理,那将是 :ref:`Item Loaders <topics-loaders>` 发挥作用的地方。

.. _topics-whatelse:

还有什么？
==========

您已经了解了如何通过Scrapy提取存储网页中的信息，但这仅仅只是冰山一角。Scrapy提供了很多强大的特性来使得爬取更为简单高效, 例如:

* HTML, XML源数据 :ref:`选择及提取 <topics-selectors>` 的内置支持

* 提供了一系列在spider之间共享的可复用的过滤器(即 :ref:`Item Loaders <topics-loaders>`)，对智能处理爬取数据提供了内置支持。

* 通过 :ref:`feed导出 <topics-feed-exports>` 提供了多格式(JSON、CSV、XML)，多存储后端(FTP、S3、本地文件系统)的内置支持

* 提供了media pipeline，可以 :ref:`自动下载 <topics-images>` 爬取到的数据中的图片(或者其他资源)。

* 高扩展性。您可以通过使用 :ref:`signals <topics-signals>` ，设计好的API(中间件, :ref:`extensions <topics-extensions>`, :ref:`pipelines<topics-item-pipeline>`)来定制实现您的功能。

* 内置的中间件及扩展为下列功能提供了支持:

  * cookies and session 处理
  * HTTP 压缩
  * HTTP 认证 
  * HTTP 缓存
  * user-agent模拟
  * robots.txt
  * 爬取深度限制
  * 其他

* 针对非英语语系中不标准或者错误的编码声明, 提供了自动检测以及健壮的编码支持。

* 支持根据模板生成爬虫。在加速爬虫创建的同时，保持在大型项目中的代码更为一致。详细内容请参阅 :command:`genspider` 命令。

* 针对多爬虫下性能评估、失败检测，提供了可扩展的 :ref:`状态收集工具 <topics-stats>` 。

* 提供 :ref:`交互式shell终端 <topics-shell>` , 为您测试XPath表达式，编写和调试爬虫提供了极大的方便

* 提供 :ref:`System service <topics-scrapyd>`, 简化在生产环境的部署及运行

* 内置 :ref:`Web service <topics-webservice>`, 使您可以监视及控制您的机器

* 内置 :ref:`Telnet终端 <topics-telnetconsole>` ，通过在Scrapy进程中钩入Python终端，使您可以查看并且调试爬虫

* :ref:`Logging <topics-logging>` 为您在爬取过程中捕捉错误提供了方便

* 支持 `Sitemaps`_ 爬取

* 具有缓存的DNS解析器

接下来
============

下一步当然是 `下载Scrapy`_ 了， 您可以阅读 :ref:`入门教程 <intro-tutorial>` 并加入 `社区`_ 。感谢您的支持!

.. _下载Scrapy: http://scrapy.org/download/
.. _社区: http://scrapy.org/community/
.. _页面抓取: http://en.wikipedia.org/wiki/Screen_scraping
.. _网络抓取: http://en.wikipedia.org/wiki/Web_scraping
.. _Amazon Associates Web Services: https://affiliate-program.amazon.com/gp/advertising/api/detail/main.html
.. _Mininova: http://www.mininova.org
.. _XPath: http://www.w3.org/TR/xpath
.. _XPath参考: http://www.w3.org/TR/xpath
.. _Amazon S3: http://aws.amazon.com/s3/
.. _Sitemaps: http://www.sitemaps.org
