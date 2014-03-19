.. _intro-install:

==================
安装指南
==================

前期准备
==============

下列的安装步骤假定您已经安装好下列程序:

* `Python`_ 2.7
* `lxml`_. 大多数Linux发行版自带了lxml。如果缺失，请查看http://lxml.de/installation.html
* `OpenSSL`_. 除了Windows(请查看 :ref:`intro-install-platform-notes`)之外的系统都已经提供。
* `pip`_ or `easy_install`_ Python安装管理器

安装Scrapy
=================

您可以通过使用easy_install或者(更被推荐用来分发和安装Python包的工具)pip来安装Scrapy。

.. note:: 请先查看 :ref:`intro-install-platform-notes`.

使用pip安装::

   pip install Scrapy

使用easy_install安装::

   easy_install Scrapy

.. _intro-install-platform-notes:

平台安装指南
====================================

Windows
-------

安装完Python后，在安装Scrapy前请先执行下列步骤:

* 修改 `控制面板`_ 中的 ``PATH`` 环境变量，将 ``C:\python27\Scripts`` 和 ``C:\python27`` 添加到系统路径中。

* 执行下列步骤来安装OpenSSL:

  1. 打开 `Win32 OpenSSL页面 <http://slproweb.com/products/Win32OpenSSL.html>`_

  2. 根据您的Windows版本和架构(32位或64位)，下载Visual C++ 2008 redistributables.

  3. 根据您的Windows版本和架构来下载OpenSSL(完整版，不是简化版)

  4. 按照上面添加 ``python27`` 环境变量的方式将 ``c:\openssl-win32\bin`` 添加到您的 ``PATH`` 中。

* 一些Scrapy依赖的二进制包(Twisted, lxml以及pyOpenSSL)要求系统中预装有编译器。如果没有安装Visual Studio,则会出现错误。您可以在下边的链接中找到Windows的安装包。在安装前请注意您的Python版本和Windows的架构(32或64位)

  * pywin32: http://sourceforge.net/projects/pywin32/files/
  * Twisted: http://twistedmatrix.com/trac/wiki/Downloads
  * zope.interface: 您可以从 `zope.interface pypi page <http://pypi.python.org/pypi/zope.interface>`_ 下载并运行 ``easy_install file.egg`` 安装
  * lxml: http://pypi.python.org/pypi/lxml/
  * pyOpenSSL: https://launchpad.net/pyopenssl

另外， 下面的页面包括很多已经编译好的Python二进制库，方便满足Scrapy的依赖。

    http://www.lfd.uci.edu/~gohlke/pythonlibs/

Ubuntu 9.10及以上版本 
~~~~~~~~~~~~~~~~~~~~

**不要** 使用Ubuntu提供的 ``python-scrapy`` ，相较于最新版的Scrapy，该包版本太旧，并且运行速度也较为缓慢。

您可以使用官方提供的 :ref:`Ubuntu Packages <topics-ubuntu>` 。该包解决了全部依赖问题，并且与最新的bug修复保持持续更新。


.. _Python: http://www.python.org
.. _pip: http://www.pip-installer.org/en/latest/installing.html
.. _easy_install: http://pypi.python.org/pypi/setuptools
.. _Control Panel: http://www.microsoft.com/resources/documentation/windows/xp/all/proddocs/en-us/sysdm_advancd_environmnt_addchange_variable.mspx
.. _lxml: http://lxml.de/
.. _OpenSSL: https://pypi.python.org/pypi/pyOpenSSL
