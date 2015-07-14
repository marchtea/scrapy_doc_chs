.. _topics-index:

===========================================================
Scrapy |version| 文档(未完成,大部分仍与0.24相同,请谨慎参考)
===========================================================

本文档涵盖了所有Scrapy的内容。

获得帮助
============

遇到问题了？我们来帮您！

* 查看下 :doc:`FAQ <faq>` ，这里有些常见的问题的解决办法。
* 寻找详细的信息？试试 :ref:`genindex` 或者 :ref:`modindex` 。
* 您可以在 `scrapy-users的邮件列表`_ 中寻找内容，或者 `提问问题`_
* 在 `#scrapy IRC channel`_ 提问
* 在 `issue tracker`_ 中提交Scrapy的bug

.. _scrapy-users的邮件列表: https://groups.google.com/group/scrapy-users/
.. _提问问题: https://groups.google.com/group/scrapy-users/
.. _#scrapy IRC channel: irc://irc.freenode.net/scrapy
.. _issue tracker: https://github.com/scrapy/scrapy/issues


第一步
===========

.. toctree::
   :hidden:

   intro/overview
   intro/install
   intro/tutorial
   intro/examples

:doc:`intro/overview`
    了解Scrapy如何助你一臂之力。

:doc:`intro/install`
    安装Scrapy。

:doc:`intro/tutorial`
    编写您的第一个Scrapy项目。

:doc:`intro/examples`
    通过把玩已存在的Scrapy项目来学习更多内容。

.. _section-basics:

基本概念
==============

.. toctree::
   :hidden:

   topics/commands
   topics/spiders
   topics/selectors
   topics/items
   topics/loaders
   topics/shell
   topics/item-pipeline
   topics/feed-exports
   topics/request-response
   topics/link-extractors
   topics/settings
   topics/exceptions


:doc:`topics/commands`
    学习用于管理Scrapy项目的命令行工具

:doc:`topics/spiders`
    编写爬取网站的规则

:doc:`topics/selectors`
    使用XPath提取网页的数据

:doc:`topics/shell`
    在交互环境中测试提取数据的代码

:doc:`topics/items`
    定义爬取的数据

:doc:`topics/loaders`
    使用爬取到的数据填充item

:doc:`topics/item-pipeline`
    后处理(Post-process)，存储爬取的数据 

:doc:`topics/feed-exports`
    以不同格式输出爬取数据到不同的存储端

:doc:`topics/request-response`
    了解代表HTTP请求(request)和返回(response)的class.

:doc:`topics/link-extractors`
    方便用于提取后续跟进链接的类。

:doc:`topics/settings`
    了解如何配置Scrapy以及所有的 :ref:`available 配置 <topics-settings-ref>`

:doc:`topics/exceptions`
    查看所有已有的异常及相应的意义.



内置服务
=================

.. toctree::
   :hidden:

   topics/logging
   topics/stats
   topics/email
   topics/telnetconsole
   topics/webservice

:doc:`topics/logging`
    了解Scrapy提供的logging功能。
   
:doc:`topics/stats`
    收集爬虫运行数据

:doc:`topics/email`
    当特定事件发生时发送邮件通知

:doc:`topics/telnetconsole`
    使用内置的Python终端检查运行中的crawler(爬虫)

:doc:`topics/webservice`
    使用web service对您的爬虫进行监控和管理


解决特定问题
=========================

.. toctree::
   :hidden:

   faq
   topics/debug
   topics/contracts
   topics/practices
   topics/broad-crawls
   topics/firefox
   topics/firebug
   topics/leaks
   topics/media-pipeline
   topics/ubuntu
   topics/deployment
   topics/autothrottle
   topics/benchmarking
   topics/jobs

:doc:`faq`
    常见问题的解决办法。

:doc:`topics/debug`
    学习如何对scrapy spider的常见问题进行debug。

:doc:`topics/contracts`
    学习如何使用contract来测试您的spider。

:doc:`topics/practices`
    熟悉Scrapy的一些惯例做法。

:doc:`topics/broad-crawls`
    调整Scrapy来适应并发爬取大量网站(a lot of domains)。

:doc:`topics/firefox`
    了解如何使用Firefox及其他有用的插件来爬取数据。

:doc:`topics/firebug`
    了解如何使用Firebug来爬取数据。

:doc:`topics/leaks`
    了解如何查找并让您的爬虫避免内存泄露。

:doc:`topics/media-pipeline`
    下载爬取的item中的文件及图片。

:doc:`topics/ubuntu`
    在Ubuntu下下载最新的Scrapy。

:doc:`topics/deployment`
    在远程服务器上部署、运行Scrapy spiders。

:doc:`topics/autothrottle`
    根据负载(load)动态调节爬取速度。

:doc:`topics/benchmarking`
    在您的硬件平台上测试Scrapy的性能。

:doc:`topics/jobs`
    学习如何停止和恢复爬虫

.. _extending-scrapy:

扩展Scrapy
================

.. toctree::
   :hidden:

   topics/architecture
   topics/downloader-middleware
   topics/spider-middleware
   topics/extensions
   topics/api
   topics/signals
   topics/exporters


:doc:`topics/architecture`
    了解Scrapy架构。

:doc:`topics/downloader-middleware`
    自定义页面被请求及下载操作。

:doc:`topics/spider-middleware`
    自定义spider的输入与输出。

:doc:`topics/extensions`
    提供您自定义的功能来扩展Scrapy

:doc:`topics/api`
    在extension(扩展)和middleware(中间件)使用api来扩展Scrapy的功能

:doc:`topics/signals`
    查看如何使用及所有可用的信号

:doc:`topics/exporters`
    快速将您爬取到的item导出到文件中(XML, CSV等格式)


其他
============

.. toctree::
   :hidden:

   news
   contributing
   versioning

:doc:`news`
    了解最近的Scrapy版本的修改。

:doc:`contributing`
    了解如何为Scrapy项目做出贡献。

:doc:`versioning`
    了解Scrapy如何命名版本以及API的稳定性。
