.. _topics-exceptions:

=================
异常(Exceptions)
=================

.. module:: scrapy.exceptions
   :synopsis: Scrapy exceptions

.. _topics-exceptions-ref:

内置异常参考手册(Built-in Exceptions reference)
===================================================

下面是Scrapy提供的异常及其用法。

DropItem
--------

.. exception:: DropItem

该异常由item pipeline抛出，用于停止处理item。详细内容请参考
:ref:`topics-item-pipeline` 。

CloseSpider
-----------

.. exception:: CloseSpider(reason='cancelled')

    该异常由spider的回调函数(callback)抛出，来暂停/停止spider。支持的参数:

    :param reason: 关闭的原因
    :type reason: str

样例::

    def parse_page(self, response):
        if 'Bandwidth exceeded' in response.body:
            raise CloseSpider('bandwidth_exceeded')

IgnoreRequest
-------------

.. exception:: IgnoreRequest

该异常由调度器(Scheduler)或其他下载中间件抛出，声明忽略该request。

NotConfigured
-------------

.. exception:: NotConfigured

该异常由某些组件抛出，声明其仍然保持关闭。这些组件包括:

 * Extensions
 * Item pipelines
 * Downloader middlwares
 * Spider middlewares

该异常必须由组件的构造器(constructor)抛出。

NotSupported
------------

.. exception:: NotSupported

该异常声明一个不支持的特性。

