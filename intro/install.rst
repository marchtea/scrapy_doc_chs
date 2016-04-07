.. _intro-install:

==================
安装指南
==================

安装Scrapy
==============

.. note:: 请先阅读 :ref:`intro-install-platform-notes`.

下列的安装步骤假定您已经安装好下列程序:

* `Python`_ 2.7

* Python Package: `pip`_ and `setuptools`_. 现在 `pip`_ 依赖 `setuptools`_ ，如果未安装，则会自动安装 `setuptools`_ 。Python 2.7.9 and later include
  `pip`_ by default, so you may have it already.

* `lxml`_. 大多数Linux发行版自带了lxml。如果缺失，请查看http://lxml.de/installation.html

* `OpenSSL`_. 除了Windows(请查看 :ref:`intro-install-platform-notes`)之外的系统都已经提供。

您可以使用pip来安装Scrapy(推荐使用pip来安装Python package).

使用pip安装::

   pip install Scrapy


.. _intro-install-platform-notes:

平台安装指南
====================================

Anaconda
--------

.. note::

  对于Windows或者使用 `pip` 安装有问题的用户, 推荐使用本方案来安装Scrapy.

如果您已经安装了 `Anaconda`_ 或者 `Miniconda`_ , 
`Scrapinghub`_ 维护了Linux, Windows, OS X系统的官方版本的conda包.

使用 ``conda`` 安装Scrapy, 请执行::

  conda install -c scrapinghub scrapy 


Windows
-------


* 从 https://www.python.org/download/ 上安装Python 2.7.

    您需要修改 ``PATH`` 环境变量，将Python的可执行程序及额外的脚本添加到系统路径中。将以下路径添加到 ``PATH`` 中::

      C:\Python2.7\;C:\Python2.7\Scripts\;

  请打开命令行，并且运行以下命令来修改 ``PATH``::

      c:\python27\python.exe c:\python27\tools\scripts\win_add2path.py

  关闭并重新打开命令行窗口，使之生效。运行接下来的命令来确认其输出所期望的Python版本::

      python --version

* 从 http://sourceforge.net/projects/pywin32/ 安装 `pywin32` 
  
  请确认下载符合您系统的版本(win32或者amd64)

* *(只有Python<2.7.9才需要)* 从
  https://pip.pypa.io/en/latest/installing.html
  安装 `pip`_ 
  
  打开命令行窗口，确认 ``pip`` 被正确安装::

      pip --version

* 到目前为止Python 2.7 及 ``pip`` 已经可以正确运行了。接下来安装Scrapy::

      pip install Scrapy


Ubuntu 9.10及以上版本 
---------------------

**不要** 使用Ubuntu提供的 ``python-scrapy`` ，相较于最新版的Scrapy，该包版本太旧，并且运行速度也较为缓慢。

您可以使用官方提供的 :ref:`Ubuntu Packages <topics-ubuntu>` 。该包解决了全部依赖问题，
并且与最新的bug修复保持持续更新。

如果您更倾向于本地构建python的依赖,而不是使用系统库(system package), 您需要先安装非python的依赖::

    sudo apt-get install python-dev python-pip libxml2-dev libxslt1-dev zlib1g-dev libffi-dev libssl-dev

您可以使用 ``pip`` 安装Scrapy::

    pip install Scrapy

.. note::

    以上非python的依赖,可以用于在Debian Wheezy (7.0) 
    及以上的系统中安装Scrpay

Archlinux
---------

您可以依照通用的方式或者从 `AUR Scrapy package` 来安装Scrapy::

    yaourt -S scrapy

Mac OS X
--------

构建Scrapy的依赖需要C编译器及开发的头文件(development headers). 在OS X中,
这通常由Apple的Xcode development tools提供. 安装Xcode command line tools, 
您需要打开一个终端,并且执行::

    xcode-select --install

这里有一个 `已知的问题 <https://github.com/pypa/pip/issues/2468>`_ 阻止
``pip`` 更新system package. 这发生在成功地安装了Scrapy极其依赖之后,以下提供了
一些可供参考的解决办法:

* *(Recommended)* **不要** 使用系统提供的python, 而且安装一个最新的,并且不会
  与系统冲突的版本. 下面展现了如何使用 `homebrew`_ 包管理工具来实现:

  * 依照 http://brew.sh/ 的指示,安装 `homebrew`_

  * 更新您的 ``PATH`` 变量, 使得 homebrew的包在system packages之前加载
    (修改 ``.bashrc`` 为 ``.zshrc`` 如果您使用 `zsh`_ 作为默认的shell)::

      echo "export PATH=/usr/local/bin:/usr/local/sbin:$PATH" >> ~/.bashrc

  * 重新加载 ``.bashrc`` 来保证修改已经生效::

      source ~/.bashrc

  * 安装python:: 

      brew install python

  * 最新版本的python已经捆绑了 ``pip`` ,所以您不需要单独安装. 
    如果不是,则需要更新python::

      brew update; brew upgrade python

* *(可选)* 在单独的python环境中安装Scrpay.

  该方法能解决OS X的问题, 不过第一种方式更为优雅.

  `virtualenv`_ 是一个在python中创建虚拟环境的工具,我们推荐您阅读
  http://docs.python-guide.org/en/latest/dev/virtualenvs/ 来了解.

在完成了以上动作后,您将可以安装Scrapy::

  pip install Scrapy

.. _Python: https://www.python.org
.. _pip: https://www.pip-installer.org/en/latest/installing.html
.. _easy_install: https://pypi.python.org/pypi/setuptools
.. _控制面板: http://www.microsoft.com/resources/documentation/windows/xp/all/proddocs/en-us/sysdm_advancd_environmnt_addchange_variable.mspx
.. _lxml: http://lxml.de/
.. _OpenSSL: https://pypi.python.org/pypi/pyOpenSSL
.. _setuptools: https://pypi.python.org/pypi/setuptools
.. _AUR Scrapy package: https://aur.archlinux.org/packages/scrapy/
.. _homebrew: http://brew.sh/
.. _zsh: http://www.zsh.org/
.. _virtualenv: https://virtualenv.pypa.io/en/latest/
.. _Scrapinghub: http://scrapinghub.com
.. _Anaconda: http://docs.continuum.io/anaconda/index
.. _Miniconda: http://conda.pydata.org/docs/install/quick.html
