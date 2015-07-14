.. _topics-selectors:

============================
选择器(Selectors)
============================

当抓取网页时，你做的最常见的任务是从HTML源码中提取数据。现有的一些库可以达到这个目的：

 * `BeautifulSoup`_ 是在程序员间非常流行的网页分析库，它基于HTML代码的结构来构造一个Python对象， 对不良标记的处理也非常合理，但它有一个缺点：慢。

 * `lxml`_ 是一个基于 `ElementTree`_ (不是Python标准库的一部分)的python化的XML解析库(也可以解析HTML)。

Scrapy提取数据有自己的一套机制。它们被称作选择器(seletors)，因为他们通过特定的 `XPath`_ 或者 `CSS`_ 表达式来“选择” HTML文件中的某个部分。

`XPath`_ 是一门用来在XML文件中选择节点的语言，也可以用在HTML上。 `CSS`_ 是一门将HTML文档样式化的语言。选择器由它定义，并与特定的HTML元素的样式相关连。

Scrapy选择器构建于 `lxml`_ 库之上，这意味着它们在速度和解析准确性上非常相似。

本页面解释了选择器如何工作，并描述了相应的API。不同于 `lxml`_ API的臃肿，该API短小而简洁。这是因为 `lxml`_ 库除了用来选择标记化文档外，还可以用到许多任务上。

选择器API的完全参考详见
:ref:`Selector reference <topics-selectors-ref>`

.. _BeautifulSoup: http://www.crummy.com/software/BeautifulSoup/
.. _lxml: http://lxml.de/
.. _ElementTree: https://docs.python.org/library/xml.etree.elementtree.html
.. _cssselect: https://pypi.python.org/pypi/cssselect/
.. _XPath: http://www.w3.org/TR/xpath
.. _CSS: http://www.w3.org/TR/selectors


使用选择器(selectors)
===================================

构造选择器(selectors)
--------------------------------------

.. highlight:: python

Scrapy selector是以 **文字(text)** 或 :class:`~scrapy.http.TextResponse` 构造的
:class:`~scrapy.selector.Selector` 实例。
其根据输入的类型自动选择最优的分析方法(XML vs HTML)::

    >>> from scrapy.selector import Selector
    >>> from scrapy.http import HtmlResponse

以文字构造::

    >>> body = '<html><body><span>good</span></body></html>'
    >>> Selector(text=body).xpath('//span/text()').extract()
    [u'good']

以response构造::

    >>> response = HtmlResponse(url='http://example.com', body=body)
    >>> Selector(response=response).xpath('//span/text()').extract()
    [u'good']

为了方便起见，response对象以 `.selector` 属性提供了一个selector，
您可以随时使用该快捷方法::

    >>> response.selector.xpath('//span/text()').extract()
    [u'good']
    


使用选择器(selectors)
-------------------------------------

我们将使用 `Scrapy shell` (提供交互测试)和位于Scrapy文档服务器的一个样例页面，来解释如何使用选择器：

    http://doc.scrapy.org/en/latest/_static/selectors-sample1.html

.. _topics-selectors-htmlcode:

这里是它的HTML源码:

.. literalinclude:: ../_static/selectors-sample1.html
   :language: html

.. highlight:: sh

首先, 我们打开shell::

    scrapy shell http://doc.scrapy.org/en/latest/_static/selectors-sample1.html

接着，当shell载入后，您将获得名为 ``response`` 的shell变量，其为响应的response，
并且在其 ``response.selector`` 属性上绑定了一个selector。

因为我们处理的是HTML，选择器将自动使用HTML语法分析。

.. highlight:: python

那么，通过查看 :ref:`HTML code <topics-selectors-htmlcode>` 该页面的源码，我们构建一个XPath来选择title标签内的文字::

    >>> response.selector.xpath('//title/text()')
    [<Selector (text) xpath=//title/text()>]

由于在response中使用XPath、CSS查询十分普遍，因此，Scrapy提供了两个实用的快捷方式:
``response.xpath()`` 及 ``response.css()``::

    >>> response.xpath('//title/text()')
    [<Selector (text) xpath=//title/text()>]
    >>> response.css('title::text')
    [<Selector (text) xpath=//title/text()>]

如你所见， ``.xpath()`` 及 ``.css()`` 方法返回一个类
:class:`~scrapy.selector.SelectorList` 的实例, 它是一个新选择器的列表。这个API可以用来快速的提取嵌套数据。

为了提取真实的原文数据，你需要调用 ``.extract()`` 方法如下::

    >>> response.xpath('//title/text()').extract()
    [u'Example website']

如果想要提取到第一个匹配到的元素, 必须调用 ``.extract_first()`` selector

    >>> response.xpath('//div[@id="images"]/a/text()').extract_first()
    u'Name: My image 1 '

如果没有匹配的元素，则返回 ``None``:

    >>> response.xpath('//div/[id="not-exists"]/text()').extract_first() is None
    True

您也可以设置默认的返回值，替代 ``None`` :

    >>> sel.xpath('//div/[id="not-exists"]/text()').extract_first(default='not-found')
    'not-found'

注意CSS选择器可以使用CSS3伪元素(pseudo-elements)来选择文字或者属性节点::

    >>> response.css('title::text').extract()
    [u'Example website']

现在我们将得到根URL(base URL)和一些图片链接::

    >>> response.xpath('//base/@href').extract()
    [u'http://example.com/']

    >>> response.css('base::attr(href)').extract()
    [u'http://example.com/']

    >>> response.xpath('//a[contains(@href, "image")]/@href').extract()
    [u'image1.html',
     u'image2.html',
     u'image3.html',
     u'image4.html',
     u'image5.html']

    >>> response.css('a[href*=image]::attr(href)').extract()
    [u'image1.html',
     u'image2.html',
     u'image3.html',
     u'image4.html',
     u'image5.html']

    >>> response.xpath('//a[contains(@href, "image")]/img/@src').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']

    >>> response.css('a[href*=image] img::attr(src)').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']

.. _topics-selectors-nesting-selectors:

嵌套选择器(selectors)
--------------------------------------------

选择器方法( ``.xpath()`` or ``.css()`` )返回相同类型的选择器列表，因此你也可以对这些选择器调用选择器方法。下面是一个例子::

    >>> links = response.xpath('//a[contains(@href, "image")]')
    >>> links.extract()
    [u'<a href="image1.html">Name: My image 1 <br><img src="image1_thumb.jpg"></a>',
     u'<a href="image2.html">Name: My image 2 <br><img src="image2_thumb.jpg"></a>',
     u'<a href="image3.html">Name: My image 3 <br><img src="image3_thumb.jpg"></a>',
     u'<a href="image4.html">Name: My image 4 <br><img src="image4_thumb.jpg"></a>',
     u'<a href="image5.html">Name: My image 5 <br><img src="image5_thumb.jpg"></a>']

    >>> for index, link in enumerate(links):
            args = (index, link.xpath('@href').extract(), link.xpath('img/@src').extract())
            print 'Link number %d points to url %s and image %s' % args

    Link number 0 points to url [u'image1.html'] and image [u'image1_thumb.jpg']
    Link number 1 points to url [u'image2.html'] and image [u'image2_thumb.jpg']
    Link number 2 points to url [u'image3.html'] and image [u'image3_thumb.jpg']
    Link number 3 points to url [u'image4.html'] and image [u'image4_thumb.jpg']
    Link number 4 points to url [u'image5.html'] and image [u'image5_thumb.jpg']

结合正则表达式使用选择器(selectors)
-------------------------------------------------------

:class:`~scrapy.selector.Selector` 也有一个 ``.re()`` 方法，用来通过正则表达式来提取数据。然而，不同于使用 ``.xpath()`` 或者 ``.css()`` 方法, ``.re()`` 方法返回unicode字符串的列表。所以你无法构造嵌套式的 ``.re()`` 调用。

下面是一个例子，从上面的 :ref:`HTML code
<topics-selectors-htmlcode>` 中提取图像名字::

    >>> response.xpath('//a[contains(@href, "image")]/text()').re(r'Name:\s*(.*)')
    [u'My image 1',
     u'My image 2',
     u'My image 3',
     u'My image 4',
     u'My image 5']

另外还有一个糅合了 ``.extract_first()`` 与 ``.re()`` 的函数
``.re_first()`` . 使用该函数可以提取第一个匹配到的字符串::

    >>> response.xpath('//a[contains(@href, "image")]/text()').re_first(r'Name:\s*(.*)')
    u'My image 1'

.. _topics-selectors-relative-xpaths:

使用相对XPaths
---------------------------

记住如果你使用嵌套的选择器，并使用起始为 ``/`` 的XPath，那么该XPath将对文档使用绝对路径，而且对于你调用的 ``Selector`` 不是相对路径。

比如，假设你想提取在 ``<div>`` 元素中的所有 ``<p>`` 元素。首先，你将先得到所有的 ``<div>`` 元素::

    >>> divs = response.xpath('//div')

开始时，你可能会尝试使用下面的错误的方法，因为它其实是从整篇文档中，而不仅仅是从那些 ``<div>`` 元素内部提取所有的 ``<p>`` 元素::

    >>> for p in divs.xpath('//p'):  # this is wrong - gets all <p> from the whole document
    ...     print p.extract()

下面是比较合适的处理方法(注意 ``.//p`` XPath的点前缀)::

    >>> for p in divs.xpath('.//p'):  # extracts all <p> inside
    ...     print p.extract()

另一种常见的情况将是提取所有直系 ``<p>`` 的结果::

    >>> for p in divs.xpath('p'):
    ...     print p.extract()

更多关于相对XPaths的细节详见XPath说明中的 `Location Paths`_ 部分。

.. _Location Paths: http://www.w3.org/TR/xpath#location-paths

使用EXSLT扩展
-------------------------------

因建于 `lxml`_ 之上, Scrapy选择器也支持一些 `EXSLT`_ 扩展，可以在XPath表达式中使用这些预先制定的命名空间：


======  =====================================    =======================
前缀    命名空间                                  用途
======  =====================================    =======================
re      \http://exslt.org/regular-expressions    `正则表达式`_
set     \http://exslt.org/sets                   `集合操作`_
======  =====================================    =======================

正则表达式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

例如在XPath的 ``starts-with()`` 或 ``contains()`` 无法满足需求时， ``test()`` 函数可以非常有用。

例如在列表中选择有"class"元素且结尾为一个数字的链接::

    >>> from scrapy import Selector
    >>> doc = """
    ... <div>
    ...     <ul>
    ...         <li class="item-0"><a href="link1.html">first item</a></li>
    ...         <li class="item-1"><a href="link2.html">second item</a></li>
    ...         <li class="item-inactive"><a href="link3.html">third item</a></li>
    ...         <li class="item-1"><a href="link4.html">fourth item</a></li>
    ...         <li class="item-0"><a href="link5.html">fifth item</a></li>
    ...     </ul>
    ... </div>
    ... """
    >>> sel = Selector(text=doc, type="html")
    >>> sel.xpath('//li//@href').extract()
    [u'link1.html', u'link2.html', u'link3.html', u'link4.html', u'link5.html']
    >>> sel.xpath('//li[re:test(@class, "item-\d$")]//@href').extract()
    [u'link1.html', u'link2.html', u'link4.html', u'link5.html']
    >>>

.. warning:: C语言库 ``libxslt`` 不原生支持EXSLT正则表达式，因此 `lxml`_ 在实现时使用了Python ``re`` 模块的钩子。
    因此，在XPath表达式中使用regexp函数可能会牺牲少量的性能。

集合操作
~~~~~~~~~~~~~~

集合操作可以方便地用于在提取文字元素前从文档树中去除一些部分。

例如使用itemscopes组和对应的itemprops来提取微数据(来自http://schema.org/Product的样本内容)::

    >>> doc = """
    ... <div itemscope itemtype="http://schema.org/Product">
    ...   <span itemprop="name">Kenmore White 17" Microwave</span>
    ...   <img src="kenmore-microwave-17in.jpg" alt='Kenmore 17" Microwave' />
    ...   <div itemprop="aggregateRating"
    ...     itemscope itemtype="http://schema.org/AggregateRating">
    ...    Rated <span itemprop="ratingValue">3.5</span>/5
    ...    based on <span itemprop="reviewCount">11</span> customer reviews
    ...   </div>
    ...
    ...   <div itemprop="offers" itemscope itemtype="http://schema.org/Offer">
    ...     <span itemprop="price">$55.00</span>
    ...     <link itemprop="availability" href="http://schema.org/InStock" />In stock
    ...   </div>
    ...
    ...   Product description:
    ...   <span itemprop="description">0.7 cubic feet countertop microwave.
    ...   Has six preset cooking categories and convenience features like
    ...   Add-A-Minute and Child Lock.</span>
    ...
    ...   Customer reviews:
    ...
    ...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
    ...     <span itemprop="name">Not a happy camper</span> -
    ...     by <span itemprop="author">Ellie</span>,
    ...     <meta itemprop="datePublished" content="2011-04-01">April 1, 2011
    ...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
    ...       <meta itemprop="worstRating" content = "1">
    ...       <span itemprop="ratingValue">1</span>/
    ...       <span itemprop="bestRating">5</span>stars
    ...     </div>
    ...     <span itemprop="description">The lamp burned out and now I have to replace
    ...     it. </span>
    ...   </div>
    ...
    ...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
    ...     <span itemprop="name">Value purchase</span> -
    ...     by <span itemprop="author">Lucas</span>,
    ...     <meta itemprop="datePublished" content="2011-03-25">March 25, 2011
    ...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
    ...       <meta itemprop="worstRating" content = "1"/>
    ...       <span itemprop="ratingValue">4</span>/
    ...       <span itemprop="bestRating">5</span>stars
    ...     </div>
    ...     <span itemprop="description">Great microwave for the price. It is small and
    ...     fits in my apartment.</span>
    ...   </div>
    ...   ...
    ... </div>
    ... """
    >>> sel = Selector(text=doc, type="html")
    >>> for scope in sel.xpath('//div[@itemscope]'):
    ...     print "current scope:", scope.xpath('@itemtype').extract()
    ...     props = scope.xpath('''
    ...                 set:difference(./descendant::*/@itemprop,
    ...                                .//*[@itemscope]/*/@itemprop)''')
    ...     print "    properties:", props.extract()
    ...     print
    ...

    current scope: [u'http://schema.org/Product']
        properties: [u'name', u'aggregateRating', u'offers', u'description', u'review', u'review']

    current scope: [u'http://schema.org/AggregateRating']
        properties: [u'ratingValue', u'reviewCount']

    current scope: [u'http://schema.org/Offer']
        properties: [u'price', u'availability']

    current scope: [u'http://schema.org/Review']
        properties: [u'name', u'author', u'datePublished', u'reviewRating', u'description']

    current scope: [u'http://schema.org/Rating']
        properties: [u'worstRating', u'ratingValue', u'bestRating']

    current scope: [u'http://schema.org/Review']
        properties: [u'name', u'author', u'datePublished', u'reviewRating', u'description']

    current scope: [u'http://schema.org/Rating']
        properties: [u'worstRating', u'ratingValue', u'bestRating']

    >>>

在这里，我们首先在 ``itemscope`` 元素上迭代，对于其中的每一个元素，我们寻找所有的 ``itemprops`` 元素，并排除那些本身在另一个 ``itemscope`` 内的元素。 

.. _EXSLT: http://exslt.org/
.. _正则表达式: http://exslt.org/regexp/index.html
.. _集合操作: http://exslt.org/set/index.html


Some XPath tips
---------------

Here are some tips that you may find useful when using XPath
with Scrapy selectors, based on `this post from ScrapingHub's blog`_.
If you are not much familiar with XPath yet,
you may want to take a look first at this `XPath tutorial`_.


.. _`XPath tutorial`: http://www.zvon.org/comp/r/tut-XPath_1.html
.. _`this post from ScrapingHub's blog`: http://blog.scrapinghub.com/2014/07/17/xpath-tips-from-the-web-scraping-trenches/


Using text nodes in a condition
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When you need to use the text content as argument to a `XPath string function`_,
avoid using ``.//text()`` and use just ``.`` instead.

This is because the expression ``.//text()`` yields a collection of text elements -- a *node-set*.
And when a node-set is converted to a string, which happens when it is passed as argument to
a string function like ``contains()`` or ``starts-with()``, it results in the text for the first element only.

Example::

    >>> from scrapy import Selector
    >>> sel = Selector(text='<a href="#">Click here to go to the <strong>Next Page</strong></a>')

Converting a *node-set* to string::

    >>> sel.xpath('//a//text()').extract() # take a peek at the node-set
    [u'Click here to go to the ', u'Next Page']
    >>> sel.xpath("string(//a[1]//text())").extract() # convert it to string
    [u'Click here to go to the ']

A *node* converted to a string, however, puts together the text of itself plus of all its descendants::

    >>> sel.xpath("//a[1]").extract() # select the first node
    [u'<a href="#">Click here to go to the <strong>Next Page</strong></a>']
    >>> sel.xpath("string(//a[1])").extract() # convert it to string
    [u'Click here to go to the Next Page']

So, using the ``.//text()`` node-set won't select anything in this case::
    >>> sel.xpath("//a[contains(.//text(), 'Next Page')]").extract()
    []

But using the ``.`` to mean the node, works::

    >>> sel.xpath("//a[contains(., 'Next Page')]").extract()
    [u'<a href="#">Click here to go to the <strong>Next Page</strong></a>']

.. _`XPath string function`: http://www.w3.org/TR/xpath/#section-String-Functions

Beware the difference between //node[1] and (//node)[1]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``//node[1]`` selects all the nodes occurring first under their respective parents.

``(//node)[1]`` selects all the nodes in the document, and then gets only the first of them.

Example::

    >>> from scrapy import Selector
    >>> sel = Selector(text="""
    ....:     <ul class="list">
    ....:         <li>1</li>
    ....:         <li>2</li>
    ....:         <li>3</li>
    ....:     </ul>
    ....:     <ul class="list">
    ....:         <li>4</li>
    ....:         <li>5</li>
    ....:         <li>6</li>
    ....:     </ul>""")
    >>> xp = lambda x: sel.xpath(x).extract()

This gets all first ``<li>``  elements under whatever it is its parent::

    >>> xp("//li[1]")
    [u'<li>1</li>', u'<li>4</li>']

And this gets the first ``<li>``  element in the whole document::

    >>> xp("(//li)[1]")
    [u'<li>1</li>']

This gets all first ``<li>``  elements under an ``<ul>``  parent::

    >>> xp("//ul/li[1]")
    [u'<li>1</li>', u'<li>4</li>']

And this gets the first ``<li>``  element under an ``<ul>``  parent in the whole document::

    >>> xp("(//ul/li)[1]")
    [u'<li>1</li>']

When querying by class, consider using CSS
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Because an element can contain multiple CSS classes, the XPath way to select elements
by class is the rather verbose::

    *[contains(concat(' ', normalize-space(@class), ' '), ' someclass ')]

If you use ``@class='someclass'`` you may end up missing elements that have
other classes, and if you just use ``contains(@class, 'someclass')`` to make up
for that you may end up with more elements that you want, if they have a different
class name that shares the string ``someclass``.

As it turns out, Scrapy selectors allow you to chain selectors, so most of the time
you can just select by class using CSS and then switch to XPath when needed::

    >>> from scrapy import Selector
    >>> sel = Selector(text='<div class="hero shout"><time datetime="2014-07-23 19:00">Special date</time></div>')
    >>> sel.css('.shout').xpath('./time/@datetime').extract()
    [u'2014-07-23 19:00']

This is cleaner than using the verbose XPath trick shown above. Just remember
to use the ``.`` in the XPath expressions that will follow.


.. _topics-selectors-ref:

内建选择器的参考
=============================

.. module:: scrapy.selector
   :synopsis: Selector class

.. class:: Selector(response=None, text=None, type=None)

   :class:`Selector` 的实例是对选择某些内容响应的封装。

  ``response`` 是 :class:`~scrapy.http.HtmlResponse` 或
  :class:`~scrapy.http.XmlResponse` 的一个对象，将被用来选择和提取数据。

  ``text`` 是在 ``response`` 不可用时的一个unicode字符串或utf-8编码的文字。将 ``text`` 和 ``response`` 一起使用是未定义行为。

  ``type`` 定义了选择器类型，可以是 ``"html"``, ``"xml"`` or ``None`` (默认).

    如果 ``type`` 是 ``None`` ，选择器会根据 ``response`` 类型(参见下面)自动选择最佳的类型，或者在和 ``text`` 一起使用时，默认为 ``"html"`` 。

    如果 ``type`` 是 ``None`` ，并传递了一个 ``response`` ，选择器类型将从response类型中推导如下：

        * ``"html"`` for :class:`~scrapy.http.HtmlResponse` type
        * ``"xml"`` for :class:`~scrapy.http.XmlResponse` type
        * ``"html"`` for anything else

   其他情况下，如果设定了 ``type`` ，选择器类型将被强制设定，而不进行检测。

  .. method:: xpath(query)

      寻找可以匹配xpath ``query`` 的节点，并返回 :class:`SelectorList` 的一个实例结果，单一化其所有元素。列表元素也实现了 :class:`Selector` 的接口。

      ``query`` 是包含XPATH查询请求的字符串。

      .. note::
        
          为了方便起见，该方法也可以通过 ``response.xpath()`` 调用


  .. method:: css(query)

      应用给定的CSS选择器，返回 :class:`SelectorList` 的一个实例。

      ``query`` 是一个包含CSS选择器的字符串。

      在后台，通过 `cssselect` 库和运行 ``.xpath()`` 方法，CSS查询会被转换为XPath查询。

      .. note::
        
          为了方便起见，该方法也可以通过 ``response.css()`` 调用


  .. method:: extract()

     串行化并将匹配到的节点返回一个unicode字符串列表。
     结尾是编码内容的百分比。

  .. method:: re(regex)

     应用给定的regex，并返回匹配到的unicode字符串列表。、

     ``regex`` 可以是一个已编译的正则表达式，也可以是一个将被 ``re.compile(regex)`` 编译为正则表达式的字符串。

  .. method:: register_namespace(prefix, uri)

     注册给定的命名空间，其将在 :class:`Selector` 中使用。
     不注册命名空间，你将无法从非标准命名空间中选择或提取数据。参见下面的例子。

  .. method:: remove_namespaces()

     移除所有的命名空间，允许使用少量的命名空间xpaths遍历文档。参加下面的例子。

  .. method:: __nonzero__()

     如果选择了任意的真实文档，将返回 ``True`` ，否则返回 ``False`` 。
     也就是说， :class:`Selector` 的布尔值是通过它选择的内容确定的。


SelectorList对象
----------------------------

.. class:: SelectorList

    :class:`SelectorList` 类是内建 ``list`` 类的子类，提供了一些额外的方法。

   .. method:: xpath(query)

       对列表中的每个元素调用 ``.xpath()`` 方法，返回结果为另一个单一化的 :class:`SelectorList` 。

       ``query`` 和 :meth:`Selector.xpath` 中的参数相同。

   .. method:: css(query)

       对列表中的各个元素调用 ``.css()`` 方法，返回结果为另一个单一化的 :class:`SelectorList` 。

       ``query`` 和 :meth:`Selector.css` 中的参数相同。

   .. method:: extract()

       对列表中的各个元素调用 ``.extract()`` 方法，返回结果为单一化的unicode字符串列表。

   .. method:: re()

       对列表中的各个元素调用 ``.re()`` 方法，返回结果为单一化的unicode字符串列表。

   .. method:: __nonzero__()

        列表非空则返回True，否则返回False。


在HTML响应上的选择器样例
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

这里是一些 :class:`Selector` 的样例，用来说明一些概念。
在所有的例子中，我们假设已经有一个通过 :class:`~scrapy.http.HtmlResponse` 对象实例化的 :class:`Selector` ，如下::

      sel = Selector(html_response)

1. 从HTML响应主体中提取所有的 ``<h1>`` 元素，返回:class:`Selector` 对象(即 :class:`SelectorList` 的一个对象)的列表::

      sel.xpath("//h1")

2. 从HTML响应主体上提取所有 ``<h1>`` 元素的文字，返回一个unicode字符串的列表::

      sel.xpath("//h1").extract()         # this includes the h1 tag
      sel.xpath("//h1/text()").extract()  # this excludes the h1 tag

3. 在所有 ``<p>`` 标签上迭代，打印它们的类属性::

      for node in sel.xpath("//p"):
          print node.xpath("@class").extract()

在XML响应上的选择器样例
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

这里是一些样例，用来说明一些概念。在两个例子中，我们假设已经有一个通过 :class:`~scrapy.http.XmlResponse` 对象实例化的 :class:`Selector` ，如下::

      sel = Selector(xml_response)

1. 从XML响应主体中选择所有的 ``<product>`` 元素，返回 :class:`Selector` 对象(即 :class:`SelectorList` 对象)的列表::

      sel.xpath("//product")

2. 从 `Google Base XML feed`_ 中提取所有的价钱，这需要注册一个命名空间::

      sel.register_namespace("g", "http://base.google.com/ns/1.0")
      sel.xpath("//g:price").extract()

.. _removing-namespaces:

移除命名空间
~~~~~~~~~~~~~~~~~~~~~~~

在处理爬虫项目时，完全去掉命名空间而仅仅处理元素名字，写更多简单/实用的XPath会方便很多。你可以为此使用 :meth:`Selector.remove_namespaces` 方法。

让我们来看一个例子，以Github博客的atom订阅来解释这个情况。

首先，我们使用想爬取的url来打开shell::

    $ scrapy shell https://github.com/blog.atom

一旦进入shell，我们可以尝试选择所有的 ``<link>`` 对象，可以看到没有结果(因为Atom XML命名空间混淆了这些节点)::

    >>> response.xpath("//link")
    []

但一旦我们调用 :meth:`Selector.remove_namespaces` 方法，所有的节点都可以直接通过他们的名字来访问::

    >>> response.selector.remove_namespaces()
    >>> response.xpath("//link")
    [<Selector xpath='//link' data=u'<link xmlns="http://www.w3.org/2005/Atom'>,
     <Selector xpath='//link' data=u'<link xmlns="http://www.w3.org/2005/Atom'>,
     ...

如果你对为什么命名空间移除操作并不总是被调用，而需要手动调用有疑惑。这是因为存在如下两个原因，按照相关顺序如下：

1. 移除命名空间需要迭代并修改文件的所有节点，而这对于Scrapy爬取的所有文档操作需要一定的性能消耗

2. 会存在这样的情况，确实需要使用命名空间，但有些元素的名字与命名空间冲突。尽管这些情况非常少见。

.. _Google Base XML feed: https://support.google.com/merchants/answer/160589?hl=en&ref_topic=2473799
