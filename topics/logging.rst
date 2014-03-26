.. _topics-logging:

=======
Logging
=======

Scrapy提供了log功能。您可以通过
:mod:`scrapy.log` 模块使用。当前底层实现使用了 `Twisted logging`_ ，不过可能在之后会有所变化。

.. _Twisted logging: http://twistedmatrix.com/projects/core/documentation/howto/logging.html

log服务必须通过显示调用 :func:`scrapy.log.start` 来开启。

.. _topics-logging-levels:

Log levels
==========

Scrapy提供5层logging级别:

1. :data:`~scrapy.log.CRITICAL` - 严重错误(critical)
2. :data:`~scrapy.log.ERROR` - 一般错误(regular errors)
3. :data:`~scrapy.log.WARNING` - 警告信息(warning messages)
4. :data:`~scrapy.log.INFO` - 一般信息(informational messages)
5. :data:`~scrapy.log.DEBUG` - 调试信息(debugging messages)

如何设置log级别
========================

您可以通过终端选项(command line option) `--loglevel/-L` 或 :setting:`LOG_LEVEL` 来设置log级别。

如何记录信息(log messages)
================================

下面给出如何使用 ``WARNING`` 级别来记录信息的例子::

    from scrapy import log
    log.msg("This is a warning", level=log.WARNING)

在Spider中添加log(Logging from Spiders)
========================================

在spider中添加log的推荐方式是使用Spider的
:meth:`~scrapy.spider.Spider.log` 方法。该方法会自动在调用
:func:`scrapy.log.msg` 时赋值 ``spider`` 参数。其他的参数则直接传递给
:func:`~scrapy.log.msg` 方法。

scrapy.log模块
=================

.. module:: scrapy.log
   :synopsis: Logging facility

.. function:: start(logfile=None, loglevel=None, logstdout=None)

    启动log功能。该方法必须在记录(log)任何信息前被调用。否则调用前的信息将会丢失。

    :param logfile: 用于保存log输出的文件路径。如果被忽略，
        :setting:`LOG_FILE` 设置会被使用。 
        如果两个参数都是 ``None`` ，log将会被输出到标准错误流(standard error)。
    :type logfile: str

    :param loglevel: 记录的最低的log级别. 可用的值有:
        :data:`CRITICAL`, :data:`ERROR`, :data:`WARNING`, :data:`INFO` and
        :data:`DEBUG`.

    :param logstdout: 如果为 ``True`` ，
        所有您的应用的标准输出(包括错误)将会被记录(logged instead)。
        例如，如果您调用 "print 'hello'" ，则'hello' 会在Scrapy的log中被显示。
        如果被忽略，则 :setting:`LOG_STDOUT` 设置会被使用。

    :type logstdout: boolean

.. function:: msg(message, level=INFO, spider=None)

    记录信息(Log a message)

    :param message: log的信息
    :type message: str

    :param level: 该信息的log级别. 参考
        :ref:`topics-logging-levels`.

    :param spider: 记录该信息的spider. 当记录的信息和特定的spider有关联时，该参数必须被使用。
    :type spider: :class:`~scrapy.spider.Spider` 对象

.. data:: CRITICAL

    严重错误的Log级别

.. data:: ERROR

    错误的Log级别
    Log level for errors

.. data:: WARNING

    警告的Log级别
    Log level for warnings

.. data:: INFO

    记录信息的Log级别(生产部署时推荐的Log级别)

.. data:: DEBUG

    调试信息的Log级别(开发时推荐的Log级别)

Logging设置
================

以下设置可以被用来配置logging:

* :setting:`LOG_ENABLED`
* :setting:`LOG_ENCODING`
* :setting:`LOG_FILE`
* :setting:`LOG_LEVEL`
* :setting:`LOG_STDOUT`

