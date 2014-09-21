.. _intro-install:

==================
安装指南
==================

安装Scrapy
==============

.. note:: 请先阅读 :ref:`intro-install-platform-notes`.

下列的安装步骤假定您已经安装好下列程序:

* `Python`_ 2.7

* Python Package: `pip`_ and `setuptools`_. 现在 `pip`_ 依赖 `setuptools`_ ，如果未安装，则会自动安装 `setuptools`_ 。

* `lxml`_. 大多数Linux发行版自带了lxml。如果缺失，请查看http://lxml.de/installation.html

* `OpenSSL`_. 除了Windows(请查看 :ref:`intro-install-platform-notes`)之外的系统都已经提供。

您可以使用pip来安装Scrapy(推荐使用pip来安装Python package).


使用pip安装::

   pip install Scrapy


.. _intro-install-platform-notes:

平台安装指南
====================================

Windows
-------


* 从 http://python.org/download/ 上安装Python 2.7.

    您需要修改 ``PATH`` 环境变量，将Python的可执行程序及额外的脚本添加到系统路径中。将以下路径添加到 ``PATH`` 中::

      C:\Python2.7\;C:\Python2.7\Scripts\;

  请打开命令行，并且运行以下命令来修改 ``PATH``::

      c:\python27\python.exe c:\python27\tools\scripts\win_add2path.py

  关闭并重新打开命令行窗口，使之生效。运行接下来的命令来确认其输出所期望的Python版本::

      python --version

* 从 https://pip.pypa.io/en/latest/installing.html 安装 `pip`_
  
  打开命令行窗口，确认 ``pip`` 被正确安装::

      pip --version

* 到目前为止Python 2.7 及 ``pip`` 已经可以正确运行了。接下来安装Scrapy::

      pip install Scrapy


Ubuntu 9.10及以上版本 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**不要** 使用Ubuntu提供的 ``python-scrapy`` ，相较于最新版的Scrapy，该包版本太旧，并且运行速度也较为缓慢。

您可以使用官方提供的 :ref:`Ubuntu Packages <topics-ubuntu>` 。该包解决了全部依赖问题，并且与最新的bug修复保持持续更新。

Archlinux
~~~~~~~~~

您可以依照通用的方式或者从 `AUR Scrapy package` 来安装Scrapy::

    yaourt -S scrapy


.. _Python: http://www.python.org
.. _pip: http://www.pip-installer.org/en/latest/installing.html
.. _easy_install: http://pypi.python.org/pypi/setuptools
.. _控制面板: http://www.microsoft.com/resources/documentation/windows/xp/all/proddocs/en-us/sysdm_advancd_environmnt_addchange_variable.mspx
.. _lxml: http://lxml.de/
.. _OpenSSL: https://pypi.python.org/pypi/pyOpenSSL
.. _setuptools: https://pypi.python.org/pypi/setuptools
.. _AUR Scrapy package: https://aur.archlinux.org/packages/scrapy/
