.. _topics-ubuntu:

===============
Ubuntu 软件包
===============

.. versionadded:: 0.10

`Scrapinghub`_ 发布的apt-get可获取版本通常比Ubuntu里更新，并且在比 `Github 仓库`_
(master & stable branches) 稳定的同时还包括了最新的漏洞修复。

用法:

1. 把Scrapy签名的GPG密钥添加到APT的钥匙环中::

    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 627220E7

2. 执行如下命令，创建 `/etc/apt/sources.list.d/scrapy.list` 文件::

    echo 'deb http://archive.scrapy.org/ubuntu scrapy main' | sudo tee /etc/apt/sources.list.d/scrapy.list

3. 更新包列表并安装 `scrapy-VERSION`, 用Scrapy的版本号(如: `scrapy-0.22`等)替换 `VERSION` ::

    sudo apt-get update && sudo apt-get install scrapy-VERSION

.. note:: 如果你要升级Scrapy，请重复步骤3。

.. warning:: debian官方源提供的 `python-scrapy` 是一个非常老的版本且不再获得Scrapy团队支持。

.. _Scrapinghub: http://scrapinghub.com/
.. _Github 仓库: https://github.com/scrapy/scrapy
