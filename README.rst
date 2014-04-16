======================================
Scrapy文档快速开始向导
======================================

文档地址
---------------------------
文档目前托管在readthedocs。请查看

http://scrapy-chs.readthedocs.org/zh_CN/latest/

Scrapy中文文档翻译计划
-----------------------------
本文档由marchtea初始进行翻译。网上也有相应的翻译，
但是大多仅仅翻译了其中的入门教程，并没有完整翻译。

于是这个坑就打开了。

目前已经翻译的有:

  * intro下边四篇文章
  * index.rst
  * faq.rst
  * topics/api.rst
  * topics/commands.rst
  * topics/items.rst
  * topics/spiders.rst
  * topics/selectors.rst
  * topics/shell.rst
  * topics/images.rst
  * topics/link-extractors.rst
  * topics/logging.rst
  * topics/feed-exporters.rst
  * topics/stats.rst
  * topics/email.rst
  * topics/telnetconsole.rst
  * topics/webservice.rst
  * topics/debug.rst
  * topics/contracts.rst
  * topics/practices.rst
  * topics/broad-crawls.rst
  * topics/firefox.rst
  * topics/firebug.rst
  * topics/leaks.rst
  * topics/autothrottle.rst
  * topics/scrapyd.rst
  * topics/ubuntu.rst
  * topics/benchmark.rst
  * topics/djangoitem.rst
  * topics/architecture.rst
  * topics/download-middleware.rst
  * topics/spider-middleware.rst
  * topics/signals.rst


加入我们吧
^^^^^^^^^^^^^^^^^^^^^^^^^^^
Scrapy的文档写得很详细，与之带来的就是文档量很大。
仅仅靠我需要很多的时间。希望您能加入这个计划，让翻译更快速。

除此之外，您也可以通过阅读相应的文章，提出翻译错误或者不满意的地方。

*  欢迎open issue
*  欢迎加入翻译组

如何翻译
^^^^^^^^^^^^^^^^^^^^^^

在开始翻译前，请先open issue，说明自己要翻译的章节，大致完成的时间。
之后fork我的repo。

翻译完成前请先在自己的环境中运行

.. code-block:: bash

    make html

大致浏览一下结果。然后提交pull request。


We need U
^^^^^^^^^^^^^^^^^^^^


本文档提供了编译Scrapy文档的快速入门


环境配置
---------------------

以下Python库在编译文档时将会被使用:

 * Sphinx
 * docutils
 * jinja

如果您已经安装好setuptools,执行下列命令将会安装全部所需的库
(Sphinx同时依赖了docutils和jinja)::

    easy_install Sphinx


编译文档
-------------------------

在当前路径下运行下面的命令将编译文档(输出为HTML)::

    make html

(HTML格式的)文档将被生成在``build/html``路径下.


查看文档
----------------------

您可以执行下列命令来查看文档::

    make htmlview


该命令将会启动您默认的浏览器，并且打开您之前生成的HTML文档的首页


重新开始
----------

执行下列命令来清除生成的文档::

    make clean

注意，该命令不会对文档的源文件有任何影响


