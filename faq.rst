.. _faq:

常见问题(FAQ)
==========================

Scrapy相BeautifulSoup或lxml比较,如何呢？
-------------------------------------------------

`BeautifulSoup`_ 及 `lxml`_ 是HTML和XML的分析库。Scrapy则是
编写爬虫，爬取网页并获取数据的应用框架(application framework)。

Scrapy提供了内置的机制来提取数据(叫做
:ref:`选择器(selectors) <topics-selectors>`)。 
但如果您觉得使用更为方便，也可以使用 `BeautifulSoup`_ (或 `lxml`_)。
总之，它们仅仅是分析库，可以在任何Python代码中被导入及使用。

换句话说，拿Scrapy与 `BeautifulSoup`_ (或 `lxml`_) 比较就好像是拿
`jinja2`_ 与 `Django`_ 相比。

.. _BeautifulSoup: http://www.crummy.com/software/BeautifulSoup/
.. _lxml: http://lxml.de/
.. _jinja2: http://jinja.pocoo.org/2/
.. _Django: http://www.djangoproject.com

.. _faq-python-versions:

Scrapy支持那些Python版本？
-----------------------------------------

Scrapy仅仅支持Python 2.7。
Python2.6的支持从Scrapy 0.20开始被废弃了。

Scrapy支持Python 3么？
---------------------------------

不。但是Python 3.3+的支持已经在计划中了。
现在，Scrapy支持Python 2.7。

.. seealso:: :ref:`faq-python-versions`.

Scrapy是否从Django中"剽窃"了X呢？
---------------------------------

也许吧，不过我们不喜欢这个词。我们认为 Django_ 是一个很好的开源项目，同时也是
一个很好的参考对象，所以我们把其作为Scrapy的启发对象。

我们坚信，如果有些事情已经做得很好了，那就没必要再重复制造轮子。这个想法，作为
开源项目及免费软件的基石之一，不仅仅针对软件，也包括文档，过程，政策等等。
所以，与其自行解决每个问题，我们选择从其他已经很好地解决问题的项目中复制想法(copy idea)
，并把注意力放在真正需要解决的问题上。

如果Scrapy能启发其他的项目，我们将为此而自豪。欢迎来抄(steal)我们！

.. _Django: http://www.djangoproject.com

Scrapy支持HTTP代理么？
-----------------------------------

是的。(从Scrapy 0.8开始)通过HTTP代理下载中间件对HTTP代理提供了支持。参考
:class:`~scrapy.contrib.downloadermiddleware.httpproxy.HttpProxyMiddleware`.

如何爬取属性在不同页面的item呢？
------------------------------------------------------------

参考 :ref:`topics-request-response-ref-request-callback-arguments`.


Scrapy退出，ImportError: Nomodule named win32api
----------------------------------------------------------

`这是个Twisted bug`_ ，您需要安装 `pywin32`_ 。

.. _pywin32: http://sourceforge.net/projects/pywin32/
.. _这是个Twisted bug: http://twistedmatrix.com/trac/ticket/3707

我要如何在spider里模拟用户登录呢?
---------------------------------------------

参考 :ref:`topics-request-response-ref-request-userlogin`.

Scrapy是以广度优先还是深度优先进行爬取的呢？
--------------------------------------------------------

默认情况下，Scrapy使用 `LIFO`_ 队列来存储等待的请求。简单的说，就是
`深度优先顺序`_ 。深度优先对大多数情况下是更方便的。如果您想以
`广度优先顺序`_ 进行爬取，你可以设置以下的设定::

    DEPTH_PRIORITY = 1
    SCHEDULER_DISK_QUEUE = 'scrapy.squeue.PickleFifoDiskQueue'
    SCHEDULER_MEMORY_QUEUE = 'scrapy.squeue.FifoMemoryQueue'

我的Scrapy爬虫有内存泄露，怎么办?
--------------------------------------------------

参考 :ref:`topics-leaks`.

另外，Python自己也有内存泄露，在
:ref:`topics-leaks-without-leaks` 有所描述。

如何让Scrapy减少内存消耗?
------------------------------------------

参考上一个问题

我能在spider中使用基本HTTP认证么？
--------------------------------------------------

可以。参考 :class:`~scrapy.contrib.downloadermiddleware.httpauth.HttpAuthMiddleware`.

为什么Scrapy下载了英文的页面，而不是我的本国语言？
------------------------------------------------------------------------

尝试通过覆盖 :setting:`DEFAULT_REQUEST_HEADERS` 设置来修改默认的 `Accept-Language`_ 请求头。

.. _Accept-Language: http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.4

我能在哪里找到Scrapy项目的例子？
----------------------------------------------

参考 :ref:`intro-examples`.

我能在不创建Scrapy项目的情况下运行一个爬虫(spider)么？
------------------------------------------------------

是的。您可以使用 :command:`runspider` 命令。例如，如果您有个
spider写在 ``my_spider.py`` 文件中，您可以运行::

    scrapy runspider my_spider.py

详情请参考 :command:`runspider` 命令。

我收到了 "Filtered offsite request" 消息。如何修复？
--------------------------------------------------------------

这些消息(以 ``DEBUG`` 所记录)并不意味着有问题，所以你可以不修复它们。

这些消息由Offsite Spider中间件(Middleware)所抛出。
该(默认启用的)中间件筛选出了不属于当前spider的站点请求。

更多详情请参见:
:class:`~scrapy.contrib.spidermiddleware.offsite.OffsiteMiddleware`.

发布Scrapy爬虫到生产环境的推荐方式？
---------------------------------------------------------------------

参见 :ref:`topics-scrapyd`.

我能对大数据(large exports)使用JSON么？
------------------------------------------

这取决于您的输出有多大。参考
:class:`~scrapy.contrib.exporter.JsonItemExporter` 文档中的
:ref:`这个警告 <json-with-large-data>` 

我能在信号处理器(signal handler)中返回(Twisted)引用么？
---------------------------------------------------------

有些信号支持从处理器中返回引用，有些不行。参考
:ref:`topics-signals-ref` 来了解详情。

reponse返回的状态值999代表了什么?
---------------------------------------------

999是雅虎用来控制请求量所定义的返回值。
试着减慢爬取速度，将spider的下载延迟改为 ``2`` 或更高::

    class MySpider(CrawlSpider):

        name = 'myspider'

        download_delay = 2

        # [ ... rest of the spider code ... ]

或在 :setting:`DOWNLOAD_DELAY` 中设置项目的全局下载延迟。

我能在spider中调用 ``pdb.set_trace()`` 来调试么？
-------------------------------------------------------------

可以，但你也可以使用Scrapy终端。这能让你快速分析(甚至修改)
spider处理返回的返回(response)。通常来说，比老旧的 ``pdb.set_trace()`` 有用多了。

更多详情请参考 :ref:`topics-shell-inspect-response`.

将所有爬取到的item转存(dump)到JSON/CSV/XML文件的最简单的方法?
-------------------------------------------------------------------

dump到JSON文件::

    scrapy crawl myspider -o items.json

dump到CSV文件::

    scrapy crawl myspider -o items.csv

dump到XML文件::

    scrapy crawl myspider -o items.xml

更多详情请参考 :ref:`topics-feed-exports`

在某些表单中巨大神秘的 ``__VIEWSTATE`` 参数是什么？
----------------------------------------------------------------------

``__VIEWSTATE`` 参数存在于ASP.NET/VB.NET建立的站点中。关于这个参数的作用请参考
`这篇文章`_ 。这里有一个爬取这种站点的
`样例爬虫`_ 。

.. _这篇文章: http://search.cpan.org/~ecarroll/HTML-TreeBuilderX-ASP_NET-0.09/lib/HTML/TreeBuilderX/ASP_NET.pm
.. _样例爬虫: http://github.com/AmbientLighter/rpn-fas/blob/master/fas/spiders/rnp.py

分析大XML/CSV数据源的最好方法是?
----------------------------------------------------

使用XPath选择器来分析大数据源可能会有问题。选择器需要在内存中对数据建立完整的
DOM树，这过程速度很慢且消耗大量内存。

为了避免一次性读取整个数据源，您可以使用
``scrapy.utils.iterators`` 中的 ``xmliter`` 及 ``csviter`` 方法。
实际上，这也是feed spider(参考 :ref:`topics-spiders`)中的处理方法。

Scrapy自动管理cookies么？
-----------------------------------------

是的，Scrapy接收并保持服务器返回来的cookies，在之后的请求会发送回去，就像正常的网页浏览器做的那样。

更多详情请参考 :ref:`topics-request-response` 及 :ref:`cookies-mw` 。

如何才能看到Scrapy发出及接收到的Scrapy呢？
--------------------------------------------------------------

启用 :setting:`COOKIES_DEBUG` 选项。

要怎么停止爬虫呢?
-------------------------------------------

在回调函数中raise :exc:`~scrapy.exceptions.CloseSpider` 异常。
更多详情请参见: :exc:`~scrapy.exceptions.CloseSpider` 。

如何避免我的Scrapy机器人(bot)被禁止(ban)呢？
----------------------------------------------------

参考 :ref:`bans`.

我应该使用spider参数(arguments)还是设置(settings)来配置spider呢？
-----------------------------------------------------------------

:ref:`spider参数 <spiderargs>` 及 :ref:`设置(settings) <topics-settings>` 都可以用来配置您的spider。
没有什么强制的规则来限定要使用哪个，但设置(settings)更适合那些一旦设置就不怎么会修改的参数，
而spider参数则意味着修改更为频繁，在每次spider运行都有修改，甚至是spider运行所必须的元素
(例如，设置spider的起始url)。

这里以例子来说明这个问题。假设您有一个spider需要登录某个网站来
爬取数据，并且仅仅想爬取特定网站的特定部分(每次都不一定相同)。
在这个情况下，认证的信息将写在设置中，而爬取的特定部分的url将是spider参数。

我爬取了一个XML文档但是XPath选择器不返回任何的item
--------------------------------------------------------------------------

也许您需要移除命名空间(namespace)。参见 :ref:`removing-namespaces`.


我得到错误: "不能导入name crawler“
--------------------------------------------------

这是由于Scrapy修改，去掉了单例模式(singletons)所引起的。
这个错误一般是由从 ``scrapy.project`` 导入 ``crawler`` 的模块引起的(扩展，中间件，pipeline或spider)。
例如::

    from scrapy.project import crawler

    class SomeExtension(object):
        def __init__(self):
            self.crawler = crawler
            # ...

这种访问crawler对象的方式已经被舍弃了，新的代码应该使用
``from_crawler`` 类方法来移植，例如::

    class SomeExtension(object):

        @classmethod
        def from_crawler(cls, crawler):
            o = cls()
            o.crawler = crawler
            return o

Scrapy终端工具(command line tool)针对旧的导入机制提供了一些支持(给出了废弃警告)，
但如果您以不同方式使用Scrapy(例如，作为类库)，该机制可能会失效。

.. _user agents: http://en.wikipedia.org/wiki/User_agent
.. _LIFO: http://en.wikipedia.org/wiki/LIFO
.. _深度优先顺序: http://en.wikipedia.org/wiki/Depth-first_search
.. _广度优先顺序: http://en.wikipedia.org/wiki/Breadth-first_search
