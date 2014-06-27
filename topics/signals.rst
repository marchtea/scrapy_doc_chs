.. _topics-signals:

================
信号(Signals)
================

Scrapy使用信号来通知事情发生。您可以在您的Scrapy项目中捕捉一些信号(使用
:ref:`extension <topics-extensions>`)来完成额外的工作或添加额外的功能，扩展Scrapy。

虽然信号提供了一些参数，不过处理函数不用接收所有的参数 -
信号分发机制(singal dispatching mechanism)仅仅提供处理器(handler)接受的参数。

您可以通过
:ref:`topics-api-signals` 来连接(或发送您自己的)信号。

延迟的信号处理器(Deferred signal handlers)
=================================================

有些信号支持从处理器返回 `Twisted deferreds`_ ，参考下边的
:ref:`topics-signals-ref` 来了解哪些支持。

.. _Twisted deferreds: http://twistedmatrix.com/documents/current/core/howto/defer.html

.. _topics-signals-ref:

内置信号参考手册(Built-in signals reference)
=============================================

.. module:: scrapy.signals
   :synopsis: Signals definitions

以下给出Scrapy内置信号的列表及其意义。

engine_started
--------------

.. signal:: engine_started
.. function:: engine_started()

    当Scrapy引擎启动爬取时发送该信号。

    该信号支持返回deferreds。

.. note:: 该信号可能会在信号 :signal:`spider_opened` 之后被发送，取决于spider的启动方式。
    所以不要 **依赖** 该信号会比 :signal:`spider-opened` 更早被发送。

engine_stopped
--------------

.. signal:: engine_stopped
.. function:: engine_stopped()

    当Scrapy引擎停止时发送该信号(例如，爬取结束)。

    该信号支持返回deferreds。

item_scraped
------------

.. signal:: item_scraped
.. function:: item_scraped(item, response, spider)

    当item被爬取，并通过所有
    :ref:`topics-item-pipeline` 后(没有被丢弃(dropped)，发送该信号。

    该信号支持返回deferreds。

    :param item: 爬取到的item
    :type item: :class:`~scrapy.item.Item` 对象

    :param spider: 爬取item的spider
    :type spider: :class:`~scrapy.spider.Spider` 对象

    :param response: 提取item的response
    :type response: :class:`~scrapy.http.Response` 对象

item_dropped
------------

.. signal:: item_dropped
.. function:: item_dropped(item, exception, spider)

    当item通过 :ref:`topics-item-pipeline` ，有些pipeline抛出
    :exc:`~scrapy.exceptions.DropItem` 异常，丢弃item时，该信号被发送。

    该信号支持返回deferreds。

    :param item: :ref:`topics-item-pipeline` 丢弃的item
    :type item: :class:`~scrapy.item.Item` 对象

    :param spider: 爬取item的spider
    :type spider: :class:`~scrapy.spider.Spider` 对象

    :param exception: 导致item被丢弃的异常(必须是
        :exc:`~scrapy.exceptions.DropItem` 的子类)
    :type exception: :exc:`~scrapy.exceptions.DropItem` 异常

spider_closed
-------------

.. signal:: spider_closed
.. function:: spider_closed(spider, reason)

    当某个spider被关闭时，该信号被发送。该信号可以用来释放每个spider在
    :signal:`spider_opened` 时占用的资源。

    该信号支持返回deferreds。

    :param spider: 关闭的spider
    :type spider: :class:`~scrapy.spider.Spider` 对象

    :param reason: 描述spider被关闭的原因的字符串。如果spider是由于完成爬取而被关闭，则其为
        ``'finished'`` 。否则，如果spider是被引擎的 ``close_spider`` 方法所关闭，则其为调用该方法时传入的
        ``reason`` 参数(默认为 ``'cancelled'``)。如果引擎被关闭(例如，
        输入Ctrl-C)，则其为 ``'shutdown'`` 。
    :type reason: str

spider_opened
-------------

.. signal:: spider_opened
.. function:: spider_opened(spider)

    当spider开始爬取时发送该信号。该信号一般用来分配spider的资源，不过其也能做任何事。

    该信号支持返回deferreds。

    :param spider: 开启的spider
    :type spider: :class:`~scrapy.spider.Spider` 对象

spider_idle
-----------

.. signal:: spider_idle
.. function:: spider_idle(spider)

    当spider进入空闲(idle)状态时该信号被发送。空闲意味着:

        * requests正在等待被下载
        * requests被调度
        * items正在item pipeline中被处理

    当该信号的所有处理器(handler)被调用后，如果spider仍然保持空闲状态，
    引擎将会关闭该spider。当spider被关闭后， :signal:`spider_closed` 信号将被发送。

    您可以，比如，在 :signal:`spider_idle` 处理器中调度某些请求来避免spider被关闭。

    该信号 **不支持** 返回deferreds。

    :param spider: 空闲的spider
    :type spider: :class:`~scrapy.spider.Spider` 对象

spider_error
------------

.. signal:: spider_error
.. function:: spider_error(failure, response, spider)

    当spider的回调函数产生错误时(例如，抛出异常)，该信号被发送。

    :param failure: 以Twisted `Failure`_ 对象抛出的异常
    :type failure: `Failure`_ 对象

    :param response: 当异常被抛出时被处理的response
    :type response: :class:`~scrapy.http.Response` 对象

    :param spider: 抛出异常的spider
    :type spider: :class:`~scrapy.spider.Spider` 对象

request_scheduled
-----------------

.. signal:: request_scheduled
.. function:: request_scheduled(request, spider)

    当引擎调度一个 :class:`~scrapy.http.Request` 对象用于下载时，该信号被发送。

    该信号 **不支持** 返回deferreds。

    :param request: 到达调度器的request
    :type request: :class:`~scrapy.http.Request` 对象

    :param spider: 产生该request的spider
    :type spider: :class:`~scrapy.spider.Spider` 对象

response_received
-----------------

.. signal:: response_received
.. function:: response_received(response, request, spider)

    当引擎从downloader获取到一个新的 :class:`~scrapy.http.Response` 时发送该信号。

    该信号 **不支持** 返回deferreds。

    :param response: 接收到的response
    :type response: :class:`~scrapy.http.Response` 对象

    :param request: 生成response的request
    :type request: :class:`~scrapy.http.Request` 对象

    :param spider: response所对应的spider
    :type spider: :class:`~scrapy.spider.Spider` 对象

response_downloaded
-------------------

.. signal:: response_downloaded
.. function:: response_downloaded(response, request, spider)

    当一个 ``HTTPResponse`` 被下载时，由downloader发送该信号。

    该信号 **不支持** 返回deferreds。

    :param response: 下载的response
    :type response: :class:`~scrapy.http.Response` 对象

    :param request: 生成response的request
    :type request: :class:`~scrapy.http.Request` 对象

    :param spider: response所对应的spider
    :type spider: :class:`~scrapy.spider.Spider` 对象

.. _Failure: http://twistedmatrix.com/documents/current/api/twisted.python.failure.Failure.html
