.. _topics-downloader-middleware:

======================================
下载器中间件(Downloader Middleware)
======================================

下载器中间件是介于Scrapy的request/response处理的钩子框架。
是用于全局修改Scrapy request和response的一个轻量、底层的系统。

.. _topics-downloader-middleware-setting:

激活下载器中间件
==================================

要激活下载器中间件组件，将其加入到 :setting:`DOWNLOADER_MIDDLEWARES` 设置中。
该设置是一个字典(dict)，键为中间件类的路径，值为其中间件的顺序(order)。

这里是一个例子::

    DOWNLOADER_MIDDLEWARES = {
        'myproject.middlewares.CustomDownloaderMiddleware': 543,
    }

:setting:`DOWNLOADER_MIDDLEWARES` 设置会与Scrapy定义的
:setting:`DOWNLOADER_MIDDLEWARES_BASE` 设置合并(但不是覆盖)，
而后根据顺序(order)进行排序，最后得到启用中间件的有序列表:
第一个中间件是最靠近引擎的，最后一个中间件是最靠近下载器的。

关于如何分配中间件的顺序请查看
:setting:`DOWNLOADER_MIDDLEWARES_BASE` 设置，而后根据您想要放置中间件的位置选择一个值。
由于每个中间件执行不同的动作，您的中间件可能会依赖于之前(或者之后)执行的中间件，因此顺序是很重要的。

如果您想禁止内置的(在
:setting:`DOWNLOADER_MIDDLEWARES_BASE` 中设置并默认启用的)中间件，
您必须在项目的 :setting:`DOWNLOADER_MIDDLEWARES` 设置中定义该中间件，并将其值赋为 `None` 。
例如，如果您想要关闭user-agent中间件::

    DOWNLOADER_MIDDLEWARES = {
        'myproject.middlewares.CustomDownloaderMiddleware': 543,
        'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': None,
    }

最后，请注意，有些中间件需要通过特定的设置来启用。更多内容请查看相关中间件文档。

编写您自己的下载器中间件
======================================

编写下载器中间件十分简单。每个中间件组件是一个定义了以下一个或多个方法的Python类:

.. module:: scrapy.downloadermiddlewares

.. class:: DownloaderMiddleware

   .. method:: process_request(request, spider)

      当每个request通过下载中间件时，该方法被调用。

      :meth:`process_request` 必须返回其中之一: 返回 ``None`` 、返回一个
      :class:`~scrapy.http.Response` 对象、返回一个 :class:`~scrapy.http.Request`
      对象或raise :exc:`~scrapy.exceptions.IgnoreRequest` 。

      如果其返回 ``None`` ，Scrapy将继续处理该request，执行其他的中间件的相应方法，直到合适的下载器处理函数(download handler)被调用，
      该request被执行(其response被下载)。

      如果其返回 :class:`~scrapy.http.Response` 对象，Scrapy将不会调用 *任何*
      其他的 :meth:`process_request` 或 :meth:`process_exception` 方法，或相应地下载函数；
      其将返回该response。 已安装的中间件的 :meth:`process_response` 方法则会在每个response返回时被调用。

      如果其返回 :class:`~scrapy.http.Request` 对象，Scrapy则停止调用
      process_request方法并重新调度返回的request。当新返回的request被执行后，
      相应地中间件链将会根据下载的response被调用。

      如果其raise一个 :exc:`~scrapy.exceptions.IgnoreRequest` 异常，则安装的下载中间件的
      :meth:`process_exception` 方法会被调用。如果没有任何一个方法处理该异常，
      则request的errback(``Request.errback``)方法会被调用。如果没有代码处理抛出的异常，
      则该异常被忽略且不记录(不同于其他异常那样)。

      :param request: 处理的request
      :type request: :class:`~scrapy.http.Request` 对象

      :param spider: 该request对应的spider
      :type spider: :class:`~scrapy.spiders.Spider` 对象

   .. method:: process_response(request, response, spider)

      :meth:`process_request` 必须返回以下之一: 返回一个 :class:`~scrapy.http.Response` 对象、
      返回一个 :class:`~scrapy.http.Request` 对象或raise一个 :exc:`~scrapy.exceptions.IgnoreRequest` 异常。

      如果其返回一个 :class:`~scrapy.http.Response` (可以与传入的response相同，也可以是全新的对象)，
      该response会被在链中的其他中间件的 :meth:`process_response` 方法处理。

      如果其返回一个 :class:`~scrapy.http.Request` 对象，则中间件链停止，
      返回的request会被重新调度下载。处理类似于 :meth:`process_request` 返回request所做的那样。

      如果其抛出一个 :exc:`~scrapy.exceptions.IgnoreRequest` 异常，则调用request的errback(``Request.errback``)。
      如果没有代码处理抛出的异常，则该异常被忽略且不记录(不同于其他异常那样)。

      :param request: response所对应的request
      :type request:  :class:`~scrapy.http.Request` 对象

      :param response: 被处理的response
      :type response: :class:`~scrapy.http.Response` 对象 

      :param spider: response所对应的spider
      :type spider: :class:`~scrapy.spiders.Spider` 对象

   .. method:: process_exception(request, exception, spider)

      当下载处理器(download handler)或 :meth:`process_request`
      (下载中间件)抛出异常(包括 :exc:`~scrapy.exceptions.IgnoreRequest` 异常)时，
      Scrapy调用 :meth:`process_exception` 。

      :meth:`process_exception` 应该返回以下之一: 返回 ``None`` 、
      一个 :class:`~scrapy.http.Response` 对象、或者一个 :class:`~scrapy.http.Request` 对象。

      如果其返回 ``None`` ，Scrapy将会继续处理该异常，接着调用已安装的其他中间件的
      :meth:`process_exception` 方法，直到所有中间件都被调用完毕，则调用默认的异常处理。

      如果其返回一个 :class:`~scrapy.http.Response` 对象，则已安装的中间件链的
      :meth:`process_response` 方法被调用。Scrapy将不会调用任何其他中间件的
      :meth:`process_exception` 方法。 

      如果其返回一个 :class:`~scrapy.http.Request` 对象，
      则返回的request将会被重新调用下载。这将停止中间件的
      :meth:`process_exception` 方法执行，就如返回一个response的那样。

      :param request: 产生异常的request
      :type request: 是 :class:`~scrapy.http.Request` 对象

      :param exception: 抛出的异常
      :type exception:  ``Exception`` 对象

      :param spider: request对应的spider
      :type spider: :class:`~scrapy.spiders.Spider` 对象 

.. _topics-downloader-middleware-ref:

内置下载中间件参考手册
========================================

本页面介绍了Scrapy自带的所有下载中间件。关于如何使用及编写您自己的中间件，请参考
:ref:`downloader middleware usage guide <topics-downloader-middleware>`.

关于默认启用的中间件列表(及其顺序)请参考
:setting:`DOWNLOADER_MIDDLEWARES_BASE` 设置。

.. _cookies-mw:

CookiesMiddleware
-----------------

.. module:: scrapy.downloadermiddlewares.cookies
   :synopsis: Cookies Downloader Middleware

.. class:: CookiesMiddleware

   该中间件使得爬取需要cookie(例如使用session)的网站成为了可能。
   其追踪了web server发送的cookie，并在之后的request中发送回去，
   就如浏览器所做的那样。

以下设置可以用来配置cookie中间件:

* :setting:`COOKIES_ENABLED`
* :setting:`COOKIES_DEBUG`

.. reqmeta:: cookiejar

单spider多cookie session
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 0.15

Scrapy通过使用 :reqmeta:`cookiejar` Request meta key来支持单spider追踪多cookie session。
默认情况下其使用一个cookie jar(session)，不过您可以传递一个标示符来使用多个。

例如::

    for i, url in enumerate(urls):
        yield scrapy.Request("http://www.example.com", meta={'cookiejar': i},
            callback=self.parse_page)

需要注意的是 :reqmeta:`cookiejar` meta key不是"黏性的(sticky)"。
您需要在之后的request请求中接着传递。例如::

    def parse_page(self, response):
        # do some processing
        return scrapy.Request("http://www.example.com/otherpage",
            meta={'cookiejar': response.meta['cookiejar']},
            callback=self.parse_other_page)

.. setting:: COOKIES_ENABLED

COOKIES_ENABLED
~~~~~~~~~~~~~~~

默认: ``True``

是否启用cookies middleware。如果关闭，cookies将不会发送给web server。

.. setting:: COOKIES_DEBUG

COOKIES_DEBUG
~~~~~~~~~~~~~

默认: ``False``

如果启用，Scrapy将记录所有在request(``Cookie``
请求头)发送的cookies及response接收到的cookies(``Set-Cookie`` 接收头)。

下边是启用 :setting:`COOKIES_DEBUG` 的记录的样例::

    2011-04-06 14:35:10-0300 [scrapy] INFO: Spider opened
    2011-04-06 14:35:10-0300 [scrapy] DEBUG: Sending cookies to: <GET http://www.diningcity.com/netherlands/index.html>
            Cookie: clientlanguage_nl=en_EN
    2011-04-06 14:35:14-0300 [scrapy] DEBUG: Received cookies from: <200 http://www.diningcity.com/netherlands/index.html>
            Set-Cookie: JSESSIONID=B~FA4DC0C496C8762AE4F1A620EAB34F38; Path=/
            Set-Cookie: ip_isocode=US
            Set-Cookie: clientlanguage_nl=en_EN; Expires=Thu, 07-Apr-2011 21:21:34 GMT; Path=/
    2011-04-06 14:49:50-0300 [scrapy] DEBUG: Crawled (200) <GET http://www.diningcity.com/netherlands/index.html> (referer: None)
    [...]


DefaultHeadersMiddleware
------------------------

.. module:: scrapy.downloadermiddlewares.defaultheaders
   :synopsis: Default Headers Downloader Middleware

.. class:: DefaultHeadersMiddleware

    该中间件设置
    :setting:`DEFAULT_REQUEST_HEADERS` 指定的默认request header。

DownloadTimeoutMiddleware
-------------------------

.. module:: scrapy.downloadermiddlewares.downloadtimeout
   :synopsis: Download timeout middleware

.. class:: DownloadTimeoutMiddleware

    该中间件设置
    :setting:`DOWNLOAD_TIMEOUT` 指定的request下载超时时间.

HttpAuthMiddleware
------------------

.. module:: scrapy.downloadermiddlewares.httpauth
   :synopsis: HTTP Auth downloader middleware

.. class:: HttpAuthMiddleware

    该中间件完成某些使用 `Basic access authentication`_ (或者叫HTTP认证)的spider生成的请求的认证过程。

    在spider中启用HTTP认证，请设置spider的 ``http_user`` 及 ``http_pass`` 属性。

    样例::

        from scrapy.spiders import CrawlSpider

        class SomeIntranetSiteSpider(CrawlSpider):

            http_user = 'someuser'
            http_pass = 'somepass'
            name = 'intranet.example.com'

            # .. rest of the spider code omitted ...

.. _Basic access authentication: http://en.wikipedia.org/wiki/Basic_access_authentication


HttpCacheMiddleware
-------------------

.. module:: scrapy.downloadermiddlewares.httpcache
   :synopsis: HTTP Cache downloader middleware

.. class:: HttpCacheMiddleware

    该中间件为所有HTTP request及response提供了底层(low-level)缓存支持。
    其由cache存储后端及cache策略组成。

    Scrapy提供了两种HTTP缓存存储后端:

        * :ref:`httpcache-storage-fs`
        * :ref:`httpcache-storage-dbm`

    您可以使用 :setting:`HTTPCACHE_STORAGE` 设定来修改HTTP缓存存储后端。
    您也可以实现您自己的存储后端。

    Scrapy提供了两种了缓存策略:

        * :ref:`httpcache-policy-rfc2616`
        * :ref:`httpcache-policy-dummy`


    您可以使用 :setting:`HTTPCACHE_POLICY` 设定来修改HTTP缓存存储后端。
    您也可以实现您自己的存储策略。


.. _httpcache-policy-dummy:

Dummy策略(默认值)
~~~~~~~~~~~~~~~~~~~~~~

该策略不考虑任何HTTP Cache-Control指令。每个request及其对应的response都被缓存。
当相同的request发生时，其不发送任何数据，直接返回response。

Dummpy策略对于测试spider十分有用。其能使spider运行更快(不需要每次等待下载完成)，
同时在没有网络连接时也能测试。其目的是为了能够回放spider的运行过程， *使之与之前的运行过程一模一样* 。

使用这个策略请设置:

* :setting:`HTTPCACHE_POLICY` 为 ``scrapy.extensions.httpcache.DummyPolicy``


.. _httpcache-policy-rfc2616:

RFC2616策略
~~~~~~~~~~~~~~

该策略提供了符合RFC2616的HTTP缓存，例如符合HTTP Cache-Control，
针对生产环境并且应用在持续性运行环境所设置。该策略能避免下载未修改的数据(来节省带宽，提高爬取速度)。

实现了:

* 当 `no-store` cache-control指令设置时不存储response/request。
* 当 `no-cache` cache-control指定设置时不从cache中提取response，即使response为最新。
* 根据 `max-age` cache-control指令中计算保存时间(freshness lifetime)。
* 根据 `Expires` 指令来计算保存时间(freshness lifetime)。
* 根据response包头的 `Last-Modified` 指令来计算保存时间(freshness lifetime)(Firefox使用的启发式算法)。
* 根据response包头的 `Age` 计算当前年龄(current age)
* 根据 `Date` 计算当前年龄(current age)
* 根据response包头的 `Last-Modified` 验证老旧的response。
* 根据response包头的 `ETag` 验证老旧的response。
* 为接收到的response设置缺失的 `Date` 字段。
* 支持request中cache-control指定的 `max-stale`

  通过该字段，使得spider完整支持了RFC2616缓存策略，但避免了多次请求下情况下的重验证问题(revalidation on a request-by-request basis).
  后者仍然需要HTTP标准进行确定.

  例子: 

  在Request的包头中添加 `Cache-Control: max-stale=600` 表明接受未超过600秒的超时时间的response.

  更多请参考: RFC2616, 14.9.3

目前仍然缺失:

* `Pragma: no-cache` 支持 http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9.1
* `Vary` 字段支持 http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.6
* 当update或delete之后失效相应的response http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.10
* ... 以及其他可能缺失的特性 ..

使用这个策略，设置:

* :setting:`HTTPCACHE_POLICY` 为 ``scrapy.extensions.httpcache.RFC2616Policy``


.. _httpcache-storage-fs:

Filesystem storage backend (默认值)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

文件系统存储后端可以用于HTTP缓存中间件。

使用该存储端，设置:

* :setting:`HTTPCACHE_STORAGE` 为 ``scrapy.extensions.httpcache.FilesystemCacheStorage``

每个request/response组存储在不同的目录中，包含下列文件:

 * ``request_body`` - the plain request body
 * ``request_headers`` - the request headers (原始HTTP格式)
 * ``response_body`` - the plain response body
 * ``response_headers`` - the request headers (原始HTTP格式)
 * ``meta`` - 以Python ``repr()`` 格式(grep-friendly格式)存储的该缓存资源的一些元数据。
 * ``pickled_meta`` - 与 ``meta`` 相同的元数据，不过使用pickle来获得更高效的反序列化性能。

目录的名称与request的指纹(参考
``scrapy.utils.request.fingerprint``)有关，而二级目录是为了避免在同一文件夹下有太多文件
(这在很多文件系统中是十分低效的)。目录的例子::

   /path/to/cache/dir/example.com/72/72811f648e718090f041317756c03adb0ada46c7

.. _httpcache-storage-dbm:

DBM storage backend
~~~~~~~~~~~~~~~~~~~

.. versionadded:: 0.13

同时也有 DBM_ 存储后端可以用于HTTP缓存中间件。

默认情况下，其采用 anydbm_ 模块，不过您也可以通过
:setting:`HTTPCACHE_DBM_MODULE` 设置进行修改。

使用该存储端，设置:

* :setting:`HTTPCACHE_STORAGE` 为 ``scrapy.extensions.httpcache.DbmCacheStorage``

.. _httpcache-storage-leveldb:

LevelDB storage backend
~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 0.23

A LevelDB_ storage backend is also available for the HTTP cache middleware.

This backend is not recommended for development because only one process can
access LevelDB databases at the same time, so you can't run a crawl and open
the scrapy shell in parallel for the same spider.

In order to use this storage backend:

* set :setting:`HTTPCACHE_STORAGE` to ``scrapy.extensions.httpcache.LeveldbCacheStorage``
* install `LevelDB python bindings`_ like ``pip install leveldb``

.. _LevelDB: http://code.google.com/p/leveldb/
.. _leveldb python bindings: https://pypi.python.org/pypi/leveldb


HTTPCache中间件设置
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

:class:`HttpCacheMiddleware` 可以通过以下设置进行配置:

.. setting:: HTTPCACHE_ENABLED

HTTPCACHE_ENABLED
^^^^^^^^^^^^^^^^^

.. versionadded:: 0.11

默认: ``False``

HTTP缓存是否开启。

.. versionchanged:: 0.11
   在0.11版本前，是使用 :setting:`HTTPCACHE_DIR` 来开启缓存。

.. reqmeta:: dont_cache

   您可以通过设置 :reqmeta:`dont_cache` 元数据为True来避免缓存.

.. setting:: HTTPCACHE_EXPIRATION_SECS

HTTPCACHE_EXPIRATION_SECS
^^^^^^^^^^^^^^^^^^^^^^^^^

默认: ``0``

缓存的request的超时时间，单位秒。

超过这个时间的缓存request将会被重新下载。如果为0，则缓存的request将永远不会超时。

.. versionchanged:: 0.11
   在0.11版本前，0的意义是缓存的request永远超时。

.. setting:: HTTPCACHE_DIR

HTTPCACHE_DIR
^^^^^^^^^^^^^

默认: ``'httpcache'``

存储(底层的)HTTP缓存的目录。如果为空，则HTTP缓存将会被关闭。
如果为相对目录，则相对于项目数据目录(project data dir)。更多内容请参考 :ref:`topics-project-structure` 。

.. setting:: HTTPCACHE_IGNORE_HTTP_CODES

HTTPCACHE_IGNORE_HTTP_CODES
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 0.10

默认: ``[]``

不缓存设置中的HTTP返回值(code)的request。

.. setting:: HTTPCACHE_IGNORE_MISSING

HTTPCACHE_IGNORE_MISSING
^^^^^^^^^^^^^^^^^^^^^^^^

默认: ``False``

如果启用，在缓存中没找到的request将会被忽略，不下载。

.. setting:: HTTPCACHE_IGNORE_SCHEMES

HTTPCACHE_IGNORE_SCHEMES
^^^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 0.10

默认: ``['file']``

不缓存这些URI标准(scheme)的response。

.. setting:: HTTPCACHE_STORAGE

HTTPCACHE_STORAGE
^^^^^^^^^^^^^^^^^

默认: ``'scrapy.extensions.httpcache.FilesystemCacheStorage'``

实现缓存存储后端的类。

.. setting:: HTTPCACHE_DBM_MODULE

HTTPCACHE_DBM_MODULE
^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 0.13

默认: ``'anydbm'``

在 :ref:`DBM存储后端 <httpcache-storage-dbm>` 的数据库模块。
该设定针对DBM后端。

.. setting:: HTTPCACHE_POLICY

HTTPCACHE_POLICY
^^^^^^^^^^^^^^^^

.. versionadded:: 0.18

默认: ``'scrapy.extensions.httpcache.DummyPolicy'``

实现缓存策略的类。

.. setting:: HTTPCACHE_GZIP

HTTPCACHE_GZIP
^^^^^^^^^^^^^^

.. versionadded:: 0.25

默认: ``False``

如果启用，scrapy将会使用gzip压缩所有缓存的数据.
该设定只针对文件系统后端(Filesystem backend)有效。


HttpCompressionMiddleware
-------------------------

.. module:: scrapy.downloadermiddlewares.httpcompression
   :synopsis: Http Compression Middleware

.. class:: HttpCompressionMiddleware

   该中间件提供了对压缩(gzip, deflate)数据的支持。

HttpCompressionMiddleware Settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: COMPRESSION_ENABLED

COMPRESSION_ENABLED
^^^^^^^^^^^^^^^^^^^

默认: ``True``

Compression Middleware(压缩中间件)是否开启。


ChunkedTransferMiddleware
-------------------------

.. module:: scrapy.downloadermiddlewares.chunked
   :synopsis: Chunked Transfer Middleware

.. class:: ChunkedTransferMiddleware

   该中间件添加了对 `chunked transfer encoding`_ 的支持。

HttpProxyMiddleware
-------------------

.. module:: scrapy.downloadermiddlewares.httpproxy
   :synopsis: Http Proxy Middleware

.. versionadded:: 0.8

.. reqmeta:: proxy

.. class:: HttpProxyMiddleware

   该中间件提供了对request设置HTTP代理的支持。您可以通过在
   :class:`~scrapy.http.Request` 对象中设置 ``proxy`` 元数据来开启代理。

   类似于Python标准库模块 `urllib`_ 及 `urllib2`_ ，其使用了下列环境变量:

   * ``http_proxy``
   * ``https_proxy``
   * ``no_proxy``

   您也可以针对每个请求设置 ``proxy`` 元数据, 其形式类似于 ``http://some_proxy_server:port``.

.. _urllib: https://docs.python.org/library/urllib.html
.. _urllib2: https://docs.python.org/library/urllib2.html

RedirectMiddleware
------------------

.. module:: scrapy.downloadermiddlewares.redirect
   :synopsis: Redirection Middleware

.. class:: RedirectMiddleware

   该中间件根据response的状态处理重定向的request。

.. reqmeta:: redirect_urls

通过该中间件的(被重定向的)request的url可以通过 
:attr:`Request.meta <scrapy.http.Request.meta>` 的 ``redirect_urls`` 键找到。

:class:`RedirectMiddleware` 可以通过下列设置进行配置(更多内容请参考设置文档):

* :setting:`REDIRECT_ENABLED`
* :setting:`REDIRECT_MAX_TIMES`

.. reqmeta:: dont_redirect

如果 :attr:`Request.meta <scrapy.http.Request.meta>` 包含
``dont_redirect`` 键，则该request将会被此中间件忽略。


RedirectMiddleware settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: REDIRECT_ENABLED

REDIRECT_ENABLED
^^^^^^^^^^^^^^^^

.. versionadded:: 0.13

默认: ``True``

是否启用Redirect中间件。

.. setting:: REDIRECT_MAX_TIMES

REDIRECT_MAX_TIMES
^^^^^^^^^^^^^^^^^^

默认: ``20``

单个request被重定向的最大次数。

MetaRefreshMiddleware
---------------------

.. class:: MetaRefreshMiddleware

   该中间件根据meta-refresh html标签处理request重定向。

:class:`MetaRefreshMiddleware` 可以通过以下设定进行配置
(更多内容请参考设置文档)。

* :setting:`METAREFRESH_ENABLED`
* :setting:`METAREFRESH_MAXDELAY`

该中间件遵循 :class:`RedirectMiddleware` 描述的
:setting:`REDIRECT_MAX_TIMES` 设定，:reqmeta:`dont_redirect` 
及 :reqmeta:`redirect_urls` meta key。


MetaRefreshMiddleware settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: METAREFRESH_ENABLED

METAREFRESH_ENABLED
^^^^^^^^^^^^^^^^^^^

.. versionadded:: 0.17

默认: ``True``

Meta Refresh中间件是否启用。

.. setting:: REDIRECT_MAX_METAREFRESH_DELAY

REDIRECT_MAX_METAREFRESH_DELAY
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

默认: ``100``

跟进重定向的最大 meta-refresh 延迟(单位:秒)。

RetryMiddleware
---------------

.. module:: scrapy.downloadermiddlewares.retry
   :synopsis: Retry Middleware

.. class:: RetryMiddleware

   该中间件将重试可能由于临时的问题，例如连接超时或者HTTP 500错误导致失败的页面。

爬取进程会收集失败的页面并在最后，spider爬取完所有正常(不失败)的页面后重新调度。
一旦没有更多需要重试的失败页面，该中间件将会发送一个信号(retry_complete)，
其他插件可以监听该信号。

:class:`RetryMiddleware` 可以通过下列设定进行配置
(更多内容请参考设置文档):

* :setting:`RETRY_ENABLED`
* :setting:`RETRY_TIMES`
* :setting:`RETRY_HTTP_CODES`

关于HTTP错误的考虑:

如果根据HTTP协议，您可能想要在设定 :setting:`RETRY_HTTP_CODES` 中移除400错误。
该错误被默认包括是由于这个代码经常被用来指示服务器过载(overload)了。而在这种情况下，我们想进行重试。

.. reqmeta:: dont_retry

如果 :attr:`Request.meta <scrapy.http.Request.meta>` 包含 ``dont_retry`` 键，
该request将会被本中间件忽略。

RetryMiddleware Settings
~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: RETRY_ENABLED

RETRY_ENABLED
^^^^^^^^^^^^^

.. versionadded:: 0.13

默认: ``True``

Retry Middleware是否启用。

.. setting:: RETRY_TIMES

RETRY_TIMES
^^^^^^^^^^^

默认: ``2``

包括第一次下载，最多的重试次数

.. setting:: RETRY_HTTP_CODES

RETRY_HTTP_CODES
^^^^^^^^^^^^^^^^

默认: ``[500, 502, 503, 504, 400, 408]``

重试的response 返回值(code)。其他错误(DNS查找问题、连接失败及其他)则一定会进行重试。

.. _topics-dlmw-robots:

RobotsTxtMiddleware
-------------------

.. module:: scrapy.downloadermiddlewares.robotstxt
   :synopsis: robots.txt middleware

.. class:: RobotsTxtMiddleware

    该中间件过滤所有robots.txt eclusion standard中禁止的request。

    确认该中间件及 :setting:`ROBOTSTXT_OBEY` 设置被启用以确保Scrapy尊重robots.txt。

    .. warning:: 记住, 如果您在一个网站中使用了多个并发请求，
       Scrapy仍然可能下载一些被禁止的页面。这是由于这些页面是在robots.txt被下载前被请求的。
       这是当前robots.txt中间件已知的限制，并将在未来进行修复。

DownloaderStats
---------------

.. module:: scrapy.downloadermiddlewares.stats
   :synopsis: Downloader Stats Middleware

.. class:: DownloaderStats

   保存所有通过的request、response及exception的中间件。

   您必须启用 :setting:`DOWNLOADER_STATS` 来启用该中间件。

UserAgentMiddleware
-------------------

.. module:: scrapy.downloadermiddlewares.useragent
   :synopsis: User Agent Middleware

.. class:: UserAgentMiddleware

   用于覆盖spider的默认user agent的中间件。

   要使得spider能覆盖默认的user agent，其 `user_agent` 属性必须被设置。

.. _ajaxcrawl-middleware:

AjaxCrawlMiddleware
-------------------

.. module:: scrapy.downloadermiddlewares.ajaxcrawl

.. class:: AjaxCrawlMiddleware

   根据meta-fragment html标签查找 'AJAX可爬取' 页面的中间件。查看
   https://developers.google.com/webmasters/ajax-crawling/docs/getting-started
   来获得更多内容。

   .. note::

       即使没有启用该中间件，Scrapy仍能查找类似于
       ``'http://example.com/!#foo=bar'`` 这样的'AJAX可爬取'页面。
       AjaxCrawlMiddleware是针对不具有 ``'!#'`` 的URL，通常发生在'index'或者'main'页面中。

AjaxCrawlMiddleware设置
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: AJAXCRAWL_ENABLED

AJAXCRAWL_ENABLED
^^^^^^^^^^^^^^^^^

.. versionadded:: 0.21

默认: ``False``

AjaxCrawlMiddleware是否启用。您可能需要针对 :ref:`通用爬虫 <topics-broad-crawls>` 启用该中间件。


.. _DBM: http://en.wikipedia.org/wiki/Dbm
.. _anydbm: https://docs.python.org/library/anydbm.html
.. _chunked transfer encoding: http://en.wikipedia.org/wiki/Chunked_transfer_encoding
