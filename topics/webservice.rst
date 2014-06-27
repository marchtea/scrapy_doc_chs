.. _topics-webservice:

===========
Web Service
===========

Scrapy提供用于监控及控制运行中的爬虫的web服务(service)。
服务通过 `JSON-RPC 2.0`_ 协议提供大部分的资源，不过也有些(只读)资源仅仅输出JSON数据。

Scrapy为管理Scrapy进程提供了一个可扩展的web服务。
您可以通过 :setting:`WEBSERVICE_ENABLED` 来启用服务。
服务将会监听 :setting:`WEBSERVICE_PORT` 的端口，并将记录写入到
:setting:`WEBSERVICE_LOGFILE` 指定的文件中。

web服务是默认启用的 :ref:`内置Scrapy扩展 <topics-extensions-ref>` ，
不过如果您运行的环境内存紧张的话，也可以关闭该扩展。

.. _topics-webservice-resources:

Web Service资源(resources)
===========================

web service提供多种资源，定义在
:setting:`WEBSERVICE_RESOURCES` 设置中。 每个资源提供了不同的功能。参考
:ref:`topics-webservice-resources-ref` 来查看默认可用的资源。

虽然您可以使用任何协议来实现您的资源，但有两种资源是和Scrapy绑定的:

* Simple JSON resources - 只读，输出JSON数据
* JSON-RPC resources - 通过使用 `JSON-RPC 2.0`_ 协议支持对一些Scrapy对象的直接访问

.. module:: scrapy.contrib.webservice
   :synopsis: Built-in web service resources

.. _topics-webservice-resources-ref:

可用JSON-RPC对象
----------------------------

Scrapy默认支持以下JSON-RPC资源:

.. _topics-webservice-crawler:

Crawler JSON-RPC资源
~~~~~~~~~~~~~~~~~~~~~~~~~

.. module:: scrapy.contrib.webservice.crawler
   :synopsis: Crawler JSON-RPC resource

.. class:: CrawlerResource

    提供对主Crawler对象的访问，来控制Scrapy进程。

    默认访问地址: http://localhost:6080/crawler

状态收集器(Stats Collector)JSON-RPC资源
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. module:: scrapy.contrib.webservice.stats
   :synopsis: Stats JSON-RPC resource

.. class:: StatsResource

    提供对crawler使用的状态收集器(Stats Collector)的访问。

    默认访问地址: http://localhost:6080/stats

爬虫管理器(Spider Manager)JSON-RPC资源
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

您可以通过
:ref:`topics-webservice-crawler` 来访问
爬虫管理器JSON-RPC资源。地址为:
http://localhost:6080/crawler/spiders

扩展管理器(Extension Manager)JSON-RPC资源
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

您可以通过
:ref:`topics-webservice-crawler` 来访问
扩展管理器JSON-RPC资源。地址为:

可用JSON资源
------------------------

Scrapy默认提供下列JSON资源:

引擎状态JSON资源
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. module:: scrapy.contrib.webservice.enginestatus
   :synopsis: Engine Status JSON resource

.. class:: EngineStatusResource

    提供了对引擎状态数据的访问

    默认访问地址: http://localhost:6080/enginestatus

Web服务设置
====================

您可以通过下列选项来设置web服务:

.. setting:: WEBSERVICE_ENABLED

WEBSERVICE_ENABLED
------------------

Default: ``True``

布尔值。确定web服务是否启用(以及说该扩展是否启用)。

.. setting:: WEBSERVICE_LOGFILE

WEBSERVICE_LOGFILE
------------------

Default: ``None``

记录对该web服务的http请求的文件。如果未设置，则记录将写到标准scrapy的log中。

.. setting:: WEBSERVICE_PORT

WEBSERVICE_PORT
---------------

Default: ``[6080, 7030]``

web服务的端口范围。如果设置为 ``None`` 或 ``0`` ，则使用动态分配的端口。

.. setting:: WEBSERVICE_HOST

WEBSERVICE_HOST
---------------

Default: ``'127.0.0.1'``

web服务监听的接口(interface)

WEBSERVICE_RESOURCES
--------------------

Default: ``{}``

您的项目所启用的web服务资源的列表(list)。请参考
:ref:`topics-webservice-resources` 。该列表将会添加/覆盖
:setting:`WEBSERVICE_RESOURCES_BASE` 中定义的Scrpay默认启用的资源的值。

WEBSERVICE_RESOURCES_BASE
-------------------------

Default::

    {
        'scrapy.contrib.webservice.crawler.CrawlerResource': 1,
        'scrapy.contrib.webservice.enginestatus.EngineStatusResource': 1,
        'scrapy.contrib.webservice.stats.StatsResource': 1,
    }

Scrapy默认提供的web服务资源的列表。
您不应该对您的项目修改这个设置，而是修改 :setting:`WEBSERVICE_RESOURCES` 。
如果您想要关闭某些资源，您可以在
:setting:`WEBSERVICE_RESOURCES` 设置其的值为 ``None`` 。

编写web服务资源(resource)
==============================

web服务资源的实现采用了Twisted Web API。
Twisted web及Twisted web资源的更多详情请参考 `Twisted Web guide`_ 。

编写web服务资源您需要继承 :class:`JsonResource` 或 :class:`JsonRpcResource`
并实现 :class:`renderGET` 方法。

.. class:: scrapy.webservice.JsonResource

    `twisted.web.resource.Resource`_ 的子类，实现了一个JSON web服务资源。参考

    .. attribute:: ws_name

        Scrapy web服务的名字，同时也是该资源监听的路径。举个例子，
        假设Scrapy web服务监听 http://localhost:6080/ ，``ws_name`` 是
        ``'resource1'`` ，则该资源的URL为:

            http://localhost:6080/resource1/

.. class:: scrapy.webservice.JsonRpcResource(crawler, target=None)

    :class:`JsonResource` 的子类，实现了JSON-RPC资源。
    JSON-RPC资源为Python(Scrapy)对象做了一层JSON-RPC API封装。
    被封装的资源必须通过
    :meth:`get_target` 方法返回。该方法默认返回构造器传入的目标(target)。

    .. method:: get_target()
        
        返回JSON-RPC所封装的对象。默认情况下，返回构造器传入的对象。

web服务资源例子
=================================

StatsResource (JSON-RPC resource)
---------------------------------

.. literalinclude:: src_used/stats.py

EngineStatusResource (JSON resource)
-------------------------------------

.. literalinclude:: src_used/enginestatus.py

Example of web service client
=============================

scrapy-ws.py script
-------------------

.. literalinclude:: src_used/scrapy-ws.py

.. _Twisted Web guide: http://jcalderone.livejournal.com/50562.html 
.. _JSON-RPC 2.0: http://www.jsonrpc.org/
.. _twisted.web.resource.Resource: http://twistedmatrix.com/documents/10.0.0/api/twisted.web.resource.Resource.html 

