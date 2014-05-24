.. _topics-commands:

=========================================
命令行工具(Command line tools)
=========================================

.. versionadded:: 0.10

Scrapy是通过 ``scrapy`` 命令行工具进行控制的。
这里我们称之为 "Scrapy tool" 以用来和子命令进行区分。
对于子命令，我们称为 "command" 或者 "Scrapy commands"。

Scrapy tool 针对不同的目的提供了多个命令，每个命令支持不同的参数和选项。

.. _topics-project-structure:

默认的Scrapy项目结构
====================================

在开始对命令行工具以及子命令的探索前，让我们首先了解一下Scrapy的项目的目录结构。

虽然可以被修改，但所有的Scrapy项目默认有类似于下边的文件结构::

   scrapy.cfg
   myproject/
       __init__.py
       items.py
       pipelines.py
       settings.py
       spiders/
           __init__.py
           spider1.py
           spider2.py
           ...

``scrapy.cfg`` 存放的目录被认为是 *项目的根目录* 。该文件中包含python模块名的字段定义了项目的设置。例如::

    [settings]
    default = myproject.settings

使用 ``scrapy`` 工具
=========================

您可以以无参数的方式启动Scrapy工具。该命令将会给出一些使用帮助以及可用的命令::

    Scrapy X.Y - no active project

    Usage:
      scrapy <command> [options] [args]

    Available commands:
      crawl         Run a spider
      fetch         Fetch a URL using the Scrapy downloader
    [...]

如果您在Scrapy项目中运行，当前激活的项目将会显示在输出的第一行。上面的输出就是响应的例子。如果您在一个项目中运行命令将会得到类似的输出::

    Scrapy X.Y - project: myproject

    Usage:
      scrapy <command> [options] [args]

    [...]

创建项目
-----------------

一般来说，使用 ``scrapy`` 工具的第一件事就是创建您的Scrapy项目::

    scrapy startproject myproject

该命令将会在 ``myproject`` 目录中创建一个Scrapy项目。

接下来，进入到项目目录中::

    cd myproject

这时候您就可以使用 ``scrapy`` 命令来管理和控制您的项目了。

控制项目
--------------------

您可以在您的项目中使用 ``scrapy`` 工具来对其进行控制和管理。

比如，创建一个新的spider::

    scrapy genspider mydomain mydomain.com

有些Scrapy命令(比如 :command:`crawl`)要求必须在Scrapy项目中运行。
您可以通过下边的 :ref:`commands reference <topics-commands-ref>`
来了解哪些命令需要在项目中运行，哪些不用。

另外要注意，有些命令在项目里运行时的效果有些许区别。
以fetch命令为例，如果被爬取的url与某个特定spider相关联，
则该命令将会使用spider的动作(spider-overridden behaviours)。
(比如spider指定的 ``user_agent``)。
该表现是有意而为之的。一般来说， ``fetch`` 命令就是用来测试检查spider是如何下载页面。

.. _topics-commands-ref:

可用的工具命令(tool commands)
========================================

该章节提供了可用的内置命令的列表。每个命令都提供了描述以及一些使用例子。您总是可以通过运行命令来获取关于每个命令的详细内容::

    scrapy <command> -h

您也可以查看所有可用的命令::

    scrapy -h

Scrapy提供了两种类型的命令。一种必须在Scrapy项目中运行(针对项目(Project-specific)的命令)，另外一种则不需要(全局命令)。全局命令在项目中运行时的表现可能会与在非项目中运行有些许差别(因为可能会使用项目的设定)。

全局命令:

* :command:`startproject`
* :command:`settings`
* :command:`runspider`
* :command:`shell`
* :command:`fetch`
* :command:`view`
* :command:`version`

项目(Project-only)命令:

* :command:`crawl`
* :command:`check`
* :command:`list`
* :command:`edit`
* :command:`parse`
* :command:`genspider`
* :command:`deploy`
* :command:`bench`

.. command:: startproject

startproject
------------

* 语法: ``scrapy startproject <project_name>``
* 是否需要项目: *no*

在 ``project_name`` 文件夹下创建一个名为 ``project_name`` 的Scrapy项目。

例子::

    $ scrapy startproject myproject

.. command:: genspider

genspider
---------

* 语法: ``scrapy genspider [-t template] <name> <domain>``
* 是否需要项目: *yes*

在当前项目中创建spider。

这仅仅是创建spider的一种快捷方法。该方法可以使用提前定义好的模板来生成spider。您也可以自己创建spider的源码文件。

例子::

    $ scrapy genspider -l
    Available templates:
      basic
      crawl
      csvfeed
      xmlfeed

    $ scrapy genspider -d basic
    import scrapy

    class $classname(scrapy.Spider):
        name = "$name"
        allowed_domains = ["$domain"]
        start_urls = (
            'http://www.$domain/',
            )

        def parse(self, response):
            pass

    $ scrapy genspider -t basic example example.com
    Created spider 'example' using template 'basic' in module:
      mybot.spiders.example

.. command:: crawl

crawl
-----

* 语法: ``scrapy crawl <spider>``
* 是否需要项目: *yes*

使用spider进行爬取。

例子::

    $ scrapy crawl myspider
    [ ... myspider starts crawling ... ]


.. command:: check

check
-----

* 语法: ``scrapy check [-l] <spider>``
* 是否需要项目: *yes*

运行contract检查。

例子::

    $ scrapy check -l
    first_spider
      * parse
      * parse_item
    second_spider
      * parse
      * parse_item

    $ scrapy check
    [FAILED] first_spider:parse_item
    >>> 'RetailPricex' field is missing

    [FAILED] first_spider:parse
    >>> Returned 92 requests, expected 0..4

.. command:: list

list
----

* 语法: ``scrapy list``
* 是否需要项目: *yes*

列出当前项目中所有可用的spider。每行输出一个spider。

使用例子::

    $ scrapy list
    spider1
    spider2

.. command:: edit

edit
----

* 语法: ``scrapy edit <spider>``
* 是否需要项目: *yes*

使用 :setting:`EDITOR` 中设定的编辑器编辑给定的spider

该命令仅仅是提供一个快捷方式。开发者可以自由选择其他工具或者IDE来编写调试spider。

例子::

    $ scrapy edit spider1

.. command:: fetch

fetch
-----

* 语法: ``scrapy fetch <url>``
* 是否需要项目: *no*

使用Scrapy下载器(downloader)下载给定的URL，并将获取到的内容送到标准输出。

该命令以spider下载页面的方式获取页面。例如，如果spider有 ``USER_AGENT`` 属性修改了 User Agent，该命令将会使用该属性。

因此，您可以使用该命令来查看spider如何获取某个特定页面。

该命令如果非项目中运行则会使用默认Scrapy downloader设定。

例子::

    $ scrapy fetch --nolog http://www.example.com/some/page.html
    [ ... html content here ... ]

    $ scrapy fetch --nolog --headers http://www.example.com/
    {'Accept-Ranges': ['bytes'],
     'Age': ['1263   '],
     'Connection': ['close     '],
     'Content-Length': ['596'],
     'Content-Type': ['text/html; charset=UTF-8'],
     'Date': ['Wed, 18 Aug 2010 23:59:46 GMT'],
     'Etag': ['"573c1-254-48c9c87349680"'],
     'Last-Modified': ['Fri, 30 Jul 2010 15:30:18 GMT'],
     'Server': ['Apache/2.2.3 (CentOS)']}

.. command:: view

view
----

* 语法: ``scrapy view <url>``
* 是否需要项目: *no*

在浏览器中打开给定的URL，并以Scrapy spider获取到的形式展现。
有些时候spider获取到的页面和普通用户看到的并不相同。
因此该命令可以用来检查spider所获取到的页面，并确认这是您所期望的。

例子::

    $ scrapy view http://www.example.com/some/page.html
    [ ... browser starts ... ]

.. command:: shell

shell
-----

* 语法: ``scrapy shell [url]``
* 是否需要项目: *no*

以给定的URL(如果给出)或者空(没有给出URL)启动Scrapy shell。
查看 :ref:`topics-shell` 获取更多信息。

例子::

    $ scrapy shell http://www.example.com/some/page.html
    [ ... scrapy shell starts ... ]

.. command:: parse

parse
-----

* 语法: ``scrapy parse <url> [options]``
* 是否需要项目: *yes*

获取给定的URL并使用相应的spider分析处理。如果您提供 ``--callback`` 选项，则使用spider的该方法处理，否则使用 ``parse`` 。

支持的选项:

* ``--spider=SPIDER``: 跳过自动检测spider并强制使用特定的spider

* ``--a NAME=VALUE``: 设置spider的参数(可能被重复)

* ``--callback`` or ``-c``: spider中用于解析返回(response)的回调函数

* ``--pipelines``: 在pipeline中处理item

* ``--rules`` or ``-r``: 使用 :class:`~scrapy.contrib.spiders.CrawlSpider` 规则来发现用来解析返回(response)的回调函数

* ``--noitems``: 不显示爬取到的item 

* ``--nolinks``: 不显示提取到的链接 

* ``--nocolour``: 避免使用pygments对输出着色

* ``--depth`` or ``-d``: 指定跟进链接请求的层次数(默认: 1)

* ``--verbose`` or ``-v``: 显示每个请求的详细信息

例子::

    $ scrapy parse http://www.example.com/ -c parse_item
    [ ... scrapy log lines crawling example.com spider ... ]

    >>> STATUS DEPTH LEVEL 1 <<<
    # Scraped Items  ------------------------------------------------------------
    [{'name': u'Example item',
     'category': u'Furniture',
     'length': u'12 cm'}]

    # Requests  -----------------------------------------------------------------
    []


.. command:: settings

settings
--------

* 语法: ``scrapy settings [options]``
* 是否需要项目: *no*

获取Scrapy的设定

在项目中运行时，该命令将会输出项目的设定值，否则输出Scrapy默认设定。

例子::

    $ scrapy settings --get BOT_NAME
    scrapybot
    $ scrapy settings --get DOWNLOAD_DELAY
    0

.. command:: runspider

runspider
---------

* 语法: ``scrapy runspider <spider_file.py>``
* 是否需要项目: *no*

在未创建项目的情况下，运行一个编写在Python文件中的spider。

例子::

    $ scrapy runspider myspider.py
    [ ... spider starts crawling ... ]

.. command:: version

version
-------

* 语法: ``scrapy version [-v]``
* 是否需要项目: *no*

输出Scrapy版本。配合 ``-v`` 运行时，该命令同时输出Python, Twisted以及平台的信息，方便bug提交。

.. command:: deploy

deploy
------

.. versionadded:: 0.11

* 语法: ``scrapy deploy [ <target:project> | -l <target> | -L ]``
* 是否需要项目: *yes*

将项目部署到Scrapyd服务。查看 `部署您的项目`_ 。

.. command:: bench

bench
-----

.. versionadded:: 0.17

* 语法: ``scrapy bench``
* 是否需要项目: *no*

运行benchmark测试。 :ref:`benchmarking` 。

自定义项目命令
=======================

您也可以通过 :setting:`COMMANDS_MODULE` 来添加您自己的项目命令。您可以以 `scrapy/commands`_ 中Scrapy commands为例来了解如何实现您的命令。

.. _scrapy/commands: https://github.com/scrapy/scrapy/blob/master/scrapy/commands
.. setting:: COMMANDS_MODULE

COMMANDS_MODULE
---------------

Default: ``''`` (empty string)

用于查找添加自定义Scrapy命令的模块。

例子::

    COMMANDS_MODULE = 'mybot.commands'

.. _部署您的项目: http://scrapyd.readthedocs.org/en/latest/deploy.html
