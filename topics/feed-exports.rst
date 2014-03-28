.. _topics-feed-exports:

============
Feed exports
============

.. versionadded:: 0.10

实现爬虫时最经常提到的需求就是能合适的保存爬取到的数据，或者说，生成一个带有爬取数据的"输出文件"(通常叫做"输出feed")，来供其他系统使用。

Scrapy自带了Feed输出，并且支持多种序列化格式(serialization format)及存储方式(storage backends)。

.. _topics-feed-format:

序列化方式(Serialization formats)
===================================

feed输出使用到了
:ref:`Item exporters <topics-exporters>` 。其自带支持的类型有:

 * :ref:`topics-feed-format-json`
 * :ref:`topics-feed-format-jsonlines`
 * :ref:`topics-feed-format-csv`
 * :ref:`topics-feed-format-xml`

您也可以通过
:setting:`FEED_EXPORTERS` 设置扩展支持的属性。
 
.. _topics-feed-format-json:

JSON
----

 * :setting:`FEED_FORMAT`: ``json``
 * 使用的exporter: :class:`~scrapy.contrib.exporter.JsonItemExporter`
 * 大数据量情况下使用JSON请参见 :ref:`这个警告 <json-with-large-data>` 

.. _topics-feed-format-jsonlines:

JSON lines
----------

 * :setting:`FEED_FORMAT`: ``jsonlines``
 * 使用的exporter: :class:`~scrapy.contrib.exporter.JsonLinesItemExporter`

.. _topics-feed-format-csv:

CSV
---

 * :setting:`FEED_FORMAT`: ``csv``
 * 使用的exporter: :class:`~scrapy.contrib.exporter.CsvItemExporter`

.. _topics-feed-format-xml:

XML
---

 * :setting:`FEED_FORMAT`: ``xml``
 * 使用的exporter: :class:`~scrapy.contrib.exporter.XmlItemExporter`

.. _topics-feed-format-pickle:

Pickle
------

 * :setting:`FEED_FORMAT`: ``pickle``
 * 使用的exporter: :class:`~scrapy.contrib.exporter.PickleItemExporter`

.. _topics-feed-format-marshal:

Marshal
-------

 * :setting:`FEED_FORMAT`: ``marshal``
 * 使用的exporter: :class:`~scrapy.contrib.exporter.MarshalItemExporter`


.. _topics-feed-storage:

存储(Storages)
==================

使用feed输出时您可以通过使用 URI_
(通过 :setting:`FEED_URI` 设置) 来定义存储端。
feed输出支持URI方式支持的多种存储后端类型。

自带支持的存储后端有:

 * :ref:`topics-feed-storage-fs`
 * :ref:`topics-feed-storage-ftp`
 * :ref:`topics-feed-storage-s3` (需要 boto_)
 * :ref:`topics-feed-storage-stdout`

有些存储后端会因所需的外部库未安装而不可用。例如，S3只有在 boto_ 库安装的情况下才可使用。


.. _topics-feed-uri-params:

存储URI参数
======================

存储URI也包含参数。当feed被创建时这些参数可以被覆盖:

 * ``%(time)s`` - 当feed被创建时被timestamp覆盖
 * ``%(name)s`` - 被spider的名字覆盖

其他命名的参数会被spider同名的属性所覆盖。例如，
当feed被创建时， ``%(site_id)s`` 将会被
``spider.site_id`` 属性所覆盖。

下面用一些例子来说明:

 * 存储在FTP，每个spider一个目录:

   * ``ftp://user:password@ftp.example.com/scraping/feeds/%(name)s/%(time)s.json``

 * 存储在S3，每一个spider一个目录:

   * ``s3://mybucket/scraping/feeds/%(name)s/%(time)s.json``


.. _topics-feed-storage-backends:

存储端(Storage backends)
===========================

.. _topics-feed-storage-fs:

本地文件系统
----------------

将feed存储在本地系统。

 * URI scheme: ``file``
 * URI样例: ``file:///tmp/export.csv``
 * 需要的外部依赖库: none

注意: (只有)存储在本地文件系统时，您可以指定一个绝对路径 ``/tmp/export.csv`` 并忽略协议(scheme)。不过这仅仅只能在Unix系统中工作。

.. _topics-feed-storage-ftp:

FTP
---

将feed存储在FTP服务器。

 * URI scheme: ``ftp``
 * URI样例: ``ftp://user:pass@ftp.example.com/path/to/export.csv``
 * 需要的外部依赖库: none

.. _topics-feed-storage-s3:

S3
--

将feed存储在 `Amazon S3`_ 。

 * URI scheme: ``s3``
 * URI样例:

   * ``s3://mybucket/path/to/export.csv``
   * ``s3://aws_key:aws_secret@mybucket/path/to/export.csv``

 * 需要的外部依赖库: `boto`_

您可以通过在URI中传递user/pass来完成AWS认证，或者也可以通过下列的设置来完成:

 * :setting:`AWS_ACCESS_KEY_ID`
 * :setting:`AWS_SECRET_ACCESS_KEY`

.. _topics-feed-storage-stdout:

标准输出
---------------

feed输出到Scrapy进程的标准输出。

 * URI scheme: ``stdout``
 * URI样例: ``stdout:``
 * 需要的外部依赖库: none


设定(Settings)
====================

这些是配置feed输出的设定:

 * :setting:`FEED_URI` (必须)
 * :setting:`FEED_FORMAT`
 * :setting:`FEED_STORAGES`
 * :setting:`FEED_EXPORTERS`
 * :setting:`FEED_STORE_EMPTY`

.. currentmodule:: scrapy.contrib.feedexport

.. setting:: FEED_URI

FEED_URI
--------

Default: ``None``

输出feed的URI。支持的URI协议请参见
:ref:`topics-feed-storage-backends` 。

为了启用feed输出，该设定是必须的。

.. setting:: FEED_FORMAT

FEED_FORMAT
-----------

输出feed的序列化格式。可用的值请参见
:ref:`topics-feed-format` 。

.. setting:: FEED_STORE_EMPTY

FEED_STORE_EMPTY
----------------

Default: ``False``

是否输出空feed(没有item的feed)。

.. setting:: FEED_STORAGES

FEED_STORAGES
-------------

Default:: ``{}``

包含项目支持的额外feed存储端的字典。
字典的键(key)是URI协议(scheme)，值是存储类(storage class)的路径。

.. setting:: FEED_STORAGES_BASE

FEED_STORAGES_BASE
------------------

Default:: 

    {
        '': 'scrapy.contrib.feedexport.FileFeedStorage',
        'file': 'scrapy.contrib.feedexport.FileFeedStorage',
        'stdout': 'scrapy.contrib.feedexport.StdoutFeedStorage',
        's3': 'scrapy.contrib.feedexport.S3FeedStorage',
        'ftp': 'scrapy.contrib.feedexport.FTPFeedStorage',
    }

包含Scrapy内置支持的feed存储端的字典。

.. setting:: FEED_EXPORTERS

FEED_EXPORTERS
--------------

Default:: ``{}``

包含项目支持的额外输出器(exporter)的字典。
该字典的键(key)是URI协议(scheme)，值是
:ref:`Item输出器(exporter) <topics-exporters>` 类的路径。

.. setting:: FEED_EXPORTERS_BASE

FEED_EXPORTERS_BASE
-------------------

Default:: 

    FEED_EXPORTERS_BASE = {
        'json': 'scrapy.contrib.exporter.JsonItemExporter',
        'jsonlines': 'scrapy.contrib.exporter.JsonLinesItemExporter',
        'csv': 'scrapy.contrib.exporter.CsvItemExporter',
        'xml': 'scrapy.contrib.exporter.XmlItemExporter',
        'marshal': 'scrapy.contrib.exporter.MarshalItemExporter',
    }

包含Scrapy内置支持的feed输出器(exporter)的字典。


.. _URI: http://en.wikipedia.org/wiki/Uniform_Resource_Identifier
.. _Amazon S3: http://aws.amazon.com/s3/
.. _boto: http://code.google.com/p/boto/
