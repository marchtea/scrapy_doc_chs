.. _topics-extensions:

==========
扩展(Extensions)
==========

扩展框架提供一个机制，使得你能将自定义功能绑定到Scrapy。

扩展只是正常的类，它们在Scrapy启动时被实例化、初始化。

扩展设置(Extension settings)
==================

扩展使用 :ref:`Scrapy settings <topics-settings>` 管理它们的设置，这跟其他Scrapy代码一样。

通常扩展需要给它们的设置加上前缀，以避免跟已有(或将来)的扩展冲突。
比如，一个扩展处理 `Google Sitemaps`_，
则可以使用类似 `GOOGLESITEMAP_ENABLED`、`GOOGLESITEMAP_DEPTH` 等设置。

.. _Google Sitemaps: http://en.wikipedia.org/wiki/Sitemaps

加载和激活扩展
===============================

扩展在扩展类被实例化时加载和激活。
因此，所有扩展的实例化代码必须在类的构造函数(``__init__``)中执行。

要使得扩展可用，需要把它添加到Scrapy的 :setting:`EXTENSIONS` 配置中。
在 :setting:`EXTENSIONS` 中，每个扩展都使用一个字符串表示，即扩展类的全Python路径。
比如::

    EXTENSIONS = {
        'scrapy.contrib.corestats.CoreStats': 500,
        'scrapy.webservice.WebService': 500,
        'scrapy.telnet.TelnetConsole': 500,
    }


如你所见，:setting:`EXTENSIONS` 配置是一个dict，key是扩展类的路径，value是顺序,
它定义扩展加载的顺序。扩展顺序不像中间件的顺序那么重要，而且扩展之间一般没有关联。
扩展加载的顺序并不重要，因为它们并不相互依赖。

如果你需要添加扩展而且它依赖别的扩展，你就可以使用该特性了。

[1] 这也是为什么Scrapy的配置项 :setting:`EXTENSIONS_BASE`
(它包括了所有内置且开启的扩展)定义所有扩展的顺序都相同 (``500``)。

可用的(Available)、开启的(enabled)和禁用的(disabled)的扩展
==========================================

并不是所有可用的扩展都会被开启。一些扩展经常依赖一些特别的配置。
比如，HTTP Cache扩展是可用的但默认是禁用的，除非 :setting:`HTTPCACHE_ENABLED` 配置项设置了。

禁用扩展(Disabling an extension)
======================

为了禁用一个默认开启的扩展(比如，包含在 :setting:`EXTENSIONS_BASE`中的扩展)，
需要将其顺序(order)设置为``None``。比如::

    EXTENSIONS = {
        'scrapy.contrib.corestats.CoreStats': None,
    }

实现你的扩展
==========================

实现你的扩展很简单。每个扩展是一个单一的Python class，它不需要实现任何特殊的方法。

Scrapy扩展(包括middlewares和pipelines)的主要入口是 ``from_crawler`` 类方法，
它接收一个 ``Crawler`` 类的实例，该实例是控制Scrapy crawler的主要对象。
如果扩展需要，你可以通过这个对象访问settings，signals，stats，控制爬虫的行为。

通常来说，扩展关联到 :ref:`signals <topics-signals>` 并执行它们触发的任务。

最后，如果 ``from_crawler`` 方法抛出 :exc:`~scrapy.exceptions.NotConfigured` 异常，
扩展会被禁用。否则，扩展会被开启。

扩展例子(Sample extension)
----------------

这里我们将实现一个简单的扩展来演示上面描述到的概念。
该扩展会在以下事件时记录一条日志：

* spider被打开
* spider被关闭
* 爬取了特定数量的条目(items)

该扩展通过 ``MYEXT_ENABLED`` 配置项开启，
items的数量通过 ``MYEXT_ITEMCOUNT`` 配置项设置。

以下是扩展的代码：

    from scrapy import signals
    from scrapy.exceptions import NotConfigured

    class SpiderOpenCloseLogging(object):

        def __init__(self, item_count):
            self.item_count = item_count

            self.items_scraped = 0

        @classmethod
        def from_crawler(cls, crawler):
            # first check if the extension should be enabled and raise

            # NotConfigured otherwise

            if not crawler.settings.getbool('MYEXT_ENABLED'):

                raise NotConfigured

            # get the number of items from settings

            item_count = crawler.settings.getint('MYEXT_ITEMCOUNT', 1000)

            # instantiate the extension object

            ext = cls(item_count)

            # connect the extension object to signals 

            crawler.signals.connect(ext.spider_opened, signal=signals.spider_opened)

            crawler.signals.connect(ext.spider_closed, signal=signals.spider_closed) 

            crawler.signals.connect(ext.item_scraped, signal=signals.item_scraped)

            # return the extension object 

            return ext

        def spider_opened(self, spider):
            spider.log("opened spider %s" % spider.name)

        def spider_closed(self, spider):
            spider.log("closed spider %s" % spider.name)

        def item_scraped(self, item, spider):
            self.items_scraped += 1

            if self.items_scraped == self.item_count:

                spider.log("scraped %d items, resetting counter" % self.items_scraped) 

                self.item_count = 0

.. _topics-extensions-ref:

内置扩展介绍
=============================

通用扩展
--------------------------

记录统计扩展(Log Stats extension)
~~~~~~~~~~~~~~~~~~~

.. module:: scrapy.contrib.logstats
   :synopsis: 记录基本统计(stats)

.. class:: LogStats

记录基本的统计信息，比如爬取的页面和条目(items)。

核心统计扩展(Core Stats extension)
~~~~~~~~~~~~~~~~~~~~

.. module:: scrapy.contrib.corestats
   :synopsis: Core stats collection

.. class:: CoreStats

如果统计收集器(stats collection)启用了，该扩展开启核心统计收集(参考 :ref:`topics-stats`)。

.. _topics-extensions-ref-webservice:

Web service 扩展
~~~~~~~~~~~~~~~~~~~~~

.. module:: scrapy.webservice
   :synopsis: Web service

.. class:: scrapy.webservice.WebService

参考 `topics-webservice`。

.. _topics-extensions-ref-telnetconsole:

Telnet console 扩展
~~~~~~~~~~~~~~~~~~~~~~~~

.. module:: scrapy.telnet
   :synopsis: Telnet console 

.. class:: scrapy.telnet.TelnetConsole

提供一个telnet控制台，用于进入当前执行的Scrapy进程的Python解析器，
这对代码调试非常有帮助。

telnet控制台通过 :setting:`TELNETCONSOLE_ENABLED` 配置项开启，
服务器会监听 :setting:`TELNETCONSOLE_PORT` 指定的端口。

.. _topics-extensions-ref-memusage:

内存使用扩展(Memory usage extension)
~~~~~~~~~~~~~~~~~~~~~~

.. module:: scrapy.contrib.memusage
   :synopsis: Memory usage extension

.. class:: scrapy.contrib.memusage.MemoryUsage

.. note:: This extension does not work in Windows.

监控Scrapy进程内存使用量，并且：

1. 如果使用内存量超过某个指定值，发送提醒邮件
2. 如果超过某个指定值，关闭spider

当内存用量达到 :setting:`MEMUSAGE_WARNING_MB` 指定的值，发送提醒邮件。
当内存用量达到 :setting:`MEMUSAGE_LIMIT_MB` 指定的值，发送提醒邮件，同时关闭spider，
Scrapy进程退出。

该扩展通过 :setting:`MEMUSAGE_ENABLED` 配置项开启，可以使用以下选项：

* :setting:`MEMUSAGE_LIMIT_MB`
* :setting:`MEMUSAGE_WARNING_MB`
* :setting:`MEMUSAGE_NOTIFY_MAIL`
* :setting:`MEMUSAGE_REPORT`

内存调试扩展(Memory debugger extension)
~~~~~~~~~~~~~~~~~~~~~~~~~

.. module:: scrapy.contrib.memdebug
   :synopsis: Memory debugger extension

.. class:: scrapy.contrib.memdebug.MemoryDebugger

该扩展用于调试内存使用量，它收集以下信息：

* 没有被Python垃圾回收器收集的对象
* 应该被销毁却仍然存活的对象。更多信息请参考 :ref:`topics-leaks-trackrefs`

开启该扩展，需打开 :setting:`MEMDEBUG_ENABLED` 配置项。
信息将会存储在统计信息(stats)中。


关闭spider扩展
~~~~~~~~~~~~~~~~~~~~~~

.. module:: scrapy.contrib.closespider
   :synopsis: Close spider extension

.. class:: scrapy.contrib.closespider.CloseSpider

当某些状况发生，spider会自动关闭。每种情况使用指定的关闭原因。

关闭spider的情况可以通过下面的设置项配置：

* :setting:`CLOSESPIDER_TIMEOUT`
* :setting:`CLOSESPIDER_ITEMCOUNT`
* :setting:`CLOSESPIDER_PAGECOUNT`
* :setting:`CLOSESPIDER_ERRORCOUNT`

.. setting:: CLOSESPIDER_TIMEOUT

CLOSESPIDER_TIMEOUT
"""""""""""""""""""

默认值: ``0``

一个整数值，单位为秒。如果一个spider在指定的秒数后仍在运行，
它将以 ``closespider_timeout`` 的原因被自动关闭。
如果值设置为0（或者没有设置），spiders不会因为超时而关闭。

.. setting:: CLOSESPIDER_ITEMCOUNT

CLOSESPIDER_ITEMCOUNT
"""""""""""""""""""""

缺省值: ``0``

一个整数值，指定条目的个数。如果spider爬取条目数超过了指定的数，
并且这些条目通过item pipeline传递，spider将会以 ``closespider_itemcount`` 的原因被自动关闭。

.. setting:: CLOSESPIDER_PAGECOUNT

CLOSESPIDER_PAGECOUNT
"""""""""""""""""""""

.. versionadded:: 0.11

缺省值: ``0``

一个整数值，指定最大的抓取响应(reponses)数。
如果spider抓取数超过指定的值，则会以 ``closespider_pagecount`` 的原因自动关闭。
如果设置为0（或者未设置），spiders不会因为抓取的响应数而关闭。

.. setting:: CLOSESPIDER_ERRORCOUNT

CLOSESPIDER_ERRORCOUNT
""""""""""""""""""""""

.. versionadded:: 0.11

缺省值: ``0``

一个整数值，指定spider可以接受的最大错误数。
如果spider生成多于该数目的错误，它将以 ``closespider_errorcount`` 的原因关闭。
如果设置为0（或者未设置），spiders不会因为发生错误过多而关闭。

StatsMailer extension
~~~~~~~~~~~~~~~~~~~~~

.. module:: scrapy.contrib.statsmailer
   :synopsis: StatsMailer extension

.. class:: scrapy.contrib.statsmailer.StatsMailer

这个简单的扩展可用来在一个域名爬取完毕时发送提醒邮件，
包含Scrapy收集的统计信息。
邮件会发送个通过 :setting:`STATSMAILER_RCPTS` 指定的所有接收人。

.. module:: scrapy.contrib.debug
   :synopsis: Extensions for debugging Scrapy

Debugging extensions
--------------------

Stack trace dump extension
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. class:: scrapy.contrib.debug.StackTraceDump

当收到 `SIGQUIT` 或 `SIGUSR2` 信号，spider进程的信息将会被存储下来。
存储的信息包括：

1. engin状态(使用``scrapy.utils.engin.get_engine_status()``)
2. 所有存活的引用(live references)(参考 :ref:`topics-leaks-trackrefs`)
3. 所有线程的堆栈信息

当堆栈信息和engine状态存储后，Scrapy进程继续正常运行。

该扩展只在POSIX兼容的平台上可运行（比如不能在Windows运行），
因为 `SIGQUIT` 和 `SIGUSR2` 信号在Windows上不可用。

至少有两种方式可以向Scrapy发送 `SIGQUIT`_ 信号：
ess)::
1. 在Scrapy进程运行时通过按Ctrl-\ (仅Linux可行?)
2. 运行该命令(假设``<pid>``是Scrapy运行的进程)::

    kill -QUIT <pid>

.. _SIGUSR2: http://en.wikipedia.org/wiki/SIGUSR1_and_SIGUSR2
.. _SIGQUIT: http://en.wikipedia.org/wiki/SIGQUIT

调试扩展(Debugger extension)
~~~~~~~~~~~~~~~~~~

.. class:: scrapy.contrib.debug.Debugger

当收到 `SIGUSR2` 信号，将会在Scrapy进程中调用 `Python debugger`_。
debugger退出后，Scrapy进程继续正常运行。

更多信息参考 `Debugging in Python`。

该扩展只在POSIX兼容平台上工作(比如不能再Windows上运行)。

.. _Python debugger: http://docs.python.org/library/pdb.html
.. _Debugging in Python: http://www.ferg.org/papers/debugging_in_python.html
