.. _topics-firefox:

==========================
借助Firefox来爬取
==========================

这里介绍一些使用Firefox进行爬取的点子及建议，以及一些帮助爬取的Firefox实用插件。

.. _topics-firefox-livedom:

在浏览器中检查DOM的注意事项
============================================

Firefox插件操作的是活动的浏览器DOM(live browser DOM)，这意味着当您检查网页源码的时候，
其已经不是原始的HTML，而是经过浏览器清理并执行一些Javascript代码后的结果。
Firefox是个典型的例子，其会在table中添加 ``<tbody>`` 元素。
而Scrapy相反，其并不修改原始的HTML，因此如果在XPath表达式中使用
``<tbody>`` ，您将获取不到任何数据。

所以，当XPath配合Firefox使用时您需要记住以下几点:

* 当检查DOM来查找Scrapy使用的XPath时，禁用Firefox的Javascrpit。

* 永远不要用完整的XPath路径。使用相对及基于属性(例如 ``id`` ， ``class`` ， ``width`` 等)的路径
  或者具有区别性的特性例如 ``contains(@href, 'image')`` 。

* 永远不要在XPath表达式中加入 ``<tbody>`` 元素，除非您知道您在做什么

.. _topics-firefox-addons:

对爬取有帮助的实用Firefox插件
===================================

Firebug
-------

`Firebug`_ 是一个在web开发者间很著名的工具，其对抓取也十分有用。
尤其是 `检查元素(Inspect Element)`_ 特性对构建抓取数据的XPath十分方便。
当移动鼠标在页面元素时，您能查看相应元素的HTML源码。

查看 :ref:`topics-firebug` ，了解如何配合Scrapy使用Firebug的详细教程。

XPather
-------

`XPather`_ 能让你在页面上直接测试XPath表达式。

XPath Checker
-------------

`XPath Checker`_ 是另一个用于测试XPath表达式的Firefox插件。

Tamper Data
-----------

`Tamper Data`_ 是一个允许您查看及修改Firefox发送的header的插件。Firebug能查看HTTP header，但无法修改。

Firecookie
----------

`Firecookie`_ 使得查看及管理cookie变得简单。您可以使用这个插件来创建新的cookie，
删除存在的cookie，查看当前站点的cookie，管理cookie的权限及其他功能。

.. _Firebug: http://getfirebug.com
.. _检查元素(Inspect Element): http://www.youtube.com/watch?v=-pT_pDe54aA
.. _XPather: https://addons.mozilla.org/firefox/addon/1192 
.. _XPath Checker: https://addons.mozilla.org/firefox/addon/1095
.. _Tamper Data: http://addons.mozilla.org/firefox/addon/966
.. _Firecookie: https://addons.mozilla.org/firefox/addon/6683

