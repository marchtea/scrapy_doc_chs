.. _topics-spiders:

=======
Spiders
=======

Spider类定义了如何爬取某个(或某些)网站。包括了爬取的动作(例如:是否跟进链接)以及如何从网页的内容中提取结构化数据(爬取item)。
换句话说，Spider就是您定义爬取的动作及分析某个网页(或者是有些网页)的地方。

对spider来说，爬取的循环类似下文:

1. 以初始的URL初始化Request，并设置回调函数。
   当该request下载完毕并返回时，将生成response，并作为参数传给该回调函数。

   spider中初始的request是通过调用 :meth:`~scrapy.spiders.Spider.start_requests` 来获取的。
   :meth:`~scrapy.spiders.Spider.start_requests` 读取 :attr:`~scrapy.spiders.Spider.start_urls` 中的URL，
   并以 :attr:`~scrapy.spiders.Spider.parse` 为回调函数生成 :class:`~scrapy.http.Request` 。

2. 在回调函数内分析返回的(网页)内容，返回 :class:`~scrapy.item.Item` 对象、dict、 :class:`~scrapy.http.Request` 或者一个包括三者的可迭代容器。
   返回的Request对象之后会经过Scrapy处理，下载相应的内容，并调用设置的callback函数(函数可相同)。

3. 在回调函数内，您可以使用 :ref:`topics-selectors`
   (您也可以使用BeautifulSoup, lxml 或者您想用的任何解析器) 来分析网页内容，并根据分析的数据生成item。

4. 最后，由spider返回的item将被存到数据库(由某些
   :ref:`Item Pipeline <topics-item-pipeline>` 处理)或使用
   :ref:`topics-feed-exports` 存入到文件中。

虽然该循环对任何类型的spider都(多少)适用，但Scrapy仍然为了不同的需求提供了多种默认spider。
之后将讨论这些spider。

.. module:: scrapy.spiders
   :synopsis: Spiders base class, spider manager and spider middleware

.. _topics-spiders-ref:

scrapy.Spider
=============

.. class:: Spider()

   Spider是最简单的spider。每个其他的spider必须继承自该类(包括Scrapy自带的其他spider以及您自己编写的spider)。
   Spider并没有提供什么特殊的功能。
   其仅仅提供了 :meth:`start_requests` 的默认实现，读取并请求spider属性中的 :attr:`start_urls`, 并根据返回的结果(resulting responses)调用spider的 ``parse`` 方法。

   .. attribute:: name

       定义spider名字的字符串(string)。spider的名字定义了Scrapy如何定位(并初始化)spider，所以其必须是唯一的。
       不过您可以生成多个相同的spider实例(instance)，这没有任何限制。
       name是spider最重要的属性，而且是必须的。

       如果该spider爬取单个网站(single domain)，一个常见的做法是以该网站(domain)(加或不加 `后缀`_ )来命名spider。
       例如，如果spider爬取 ``mywebsite.com`` ，该spider通常会被命名为 ``mywebsite`` 。

   .. attribute:: allowed_domains

       可选。包含了spider允许爬取的域名(domain)列表(list)。
       当 :class:`~scrapy.spidermiddlewares.offsite.OffsiteMiddleware` 启用时，
       域名不在列表中的URL不会被跟进。

   .. attribute:: start_urls

       URL列表。当没有制定特定的URL时，spider将从该列表中开始进行爬取。
       因此，第一个被获取到的页面的URL将是该列表之一。
       后续的URL将会从获取到的数据中提取。

   .. attribute:: custom_settings

      A dictionary of settings that will be overridden from the project wide
      configuration when running this spider. It must be defined as a class
      attribute since the settings are updated before instantiation.

      For a list of available built-in settings see:
      :ref:`topics-settings-ref`.

   .. attribute:: crawler

      This attribute is set by the :meth:`from_crawler` class method after
      initializating the class, and links to the
      :class:`~scrapy.crawler.Crawler` object to which this spider instance is
      bound.

      Crawlers encapsulate a lot of components in the project for their single
      entry access (such as extensions, middlewares, signals managers, etc).
      See :ref:`topics-api-crawler` to know more about them.

   .. attribute:: settings

      Configuration on which this spider is been ran. This is a
      :class:`~scrapy.settings.Settings` instance, see the
      :ref:`topics-settings` topic for a detailed introduction on this subject.

   .. attribute:: logger

      Python logger created with the Spider's :attr:`name`. You can use it to
      send log messages through it as described on
      :ref:`topics-logging-from-spiders`.

   .. method:: from_crawler(crawler, \*args, \**kwargs)

       This is the class method used by Scrapy to create your spiders.

       You probably won't need to override this directly, since the default
       implementation acts as a proxy to the :meth:`__init__` method, calling
       it with the given arguments `args` and named arguments `kwargs`.

       Nonetheless, this method sets the :attr:`crawler` and :attr:`settings`
       attributes in the new instance, so they can be accessed later inside the
       spider's code.

       :param crawler: crawler to which the spider will be bound
       :type crawler: :class:`~scrapy.crawler.Crawler` instance

       :param args: arguments passed to the :meth:`__init__` method
       :type args: list

       :param kwargs: keyword arguments passed to the :meth:`__init__` method
       :type kwargs: dict

   .. method:: start_requests()

       该方法必须返回一个可迭代对象(iterable)。该对象包含了spider用于爬取的第一个Request。

       当spider启动爬取并且未制定URL时，该方法被调用。
       当指定了URL时，:meth:`make_requests_from_url` 将被调用来创建Request对象。
       该方法仅仅会被Scrapy调用一次，因此您可以将其实现为生成器。

       该方法的默认实现是使用 :attr:`start_urls` 的url生成Request。

       如果您想要修改最初爬取某个网站的Request对象，您可以重写(override)该方法。
       例如，如果您需要在启动时以POST登录某个网站，你可以这么写::

           class MySpider(scrapy.Spider):
               name = 'myspider'
                
               def start_requests(self):
                   return [scrapy.FormRequest("http://www.example.com/login",
                                              formdata={'user': 'john', 'pass': 'secret'},
                                              callback=self.logged_in)]

               def logged_in(self, response):
                   # here you would extract links to follow and return Requests for
                   # each of them, with another callback
                   pass

   .. method:: make_requests_from_url(url)

       该方法接受一个URL并返回用于爬取的 :class:`~scrapy.http.Request` 对象。
       该方法在初始化request时被 :meth:`start_requests` 调用，也被用于转化url为request。

       默认未被复写(overridden)的情况下，该方法返回的Request对象中， 
       :meth:`parse` 作为回调函数，dont_filter参数也被设置为开启。
       (详情参见 :class:`~scrapy.http.Request`).

   .. method:: parse(response)

       当response没有指定回调函数时，该方法是Scrapy处理下载的response的默认方法。

       ``parse`` 负责处理response并返回处理的数据以及(/或)跟进的URL。
       :class:`Spider` 对其他的Request的回调函数也有相同的要求。

       该方法及其他的Request回调函数必须返回一个包含 
       :class:`~scrapy.http.Request`、dict 或 :class:`~scrapy.item.Item`
       的可迭代的对象。

       :param response: 用于分析的response
       :type response: :class:`~scrapy.http.Response`

   .. method:: log(message, [level, component])

       使用 :func:`scrapy.log.msg` 方法记录(log)message。
       log中自动带上该spider的 :attr:`name` 属性。
       更多数据请参见 :ref:`topics-logging` 。
       封装了通过Spiders的 :attr:`logger` 来发送log消息的方法，并且保持了向后兼容性。
       更多内容请参考 
       :ref:`topics-logging-from-spiders`.

   .. method:: closed(reason)

       当spider关闭时，该函数被调用。
       该方法提供了一个替代调用signals.connect()来监听 :signal:`spider_closed` 信号的快捷方式。


让我们来看一个例子::

    import scrapy

    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = [
            'http://www.example.com/1.html',
            'http://www.example.com/2.html',
            'http://www.example.com/3.html',
        ]

        def parse(self, response):
            self.logger.info('A response from %s just arrived!', response.url)

在单个回调函数中返回多个Request以及Item的例子::

    import scrapy

    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = [
            'http://www.example.com/1.html',
            'http://www.example.com/2.html',
            'http://www.example.com/3.html',
        ]

        def parse(self, response):
            sel = scrapy.Selector(response)
            for h3 in response.xpath('//h3').extract():
                yield {"title": h3}

            for url in response.xpath('//a/@href').extract():
                yield scrapy.Request(url, callback=self.parse)
                
除了 :attr:`~.start_urls` ，你也可以直接使用 :meth:`~.start_requests` ;
您也可以使用 :ref:`topics-items` 来给予数据更多的结构性(give data more structure)::

    import scrapy
    from myproject.items import MyItem

    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']
        
        def start_requests(self):
            yield scrapy.Request('http://www.example.com/1.html', self.parse)
            yield scrapy.Request('http://www.example.com/2.html', self.parse)
            yield scrapy.Request('http://www.example.com/3.html', self.parse)

        def parse(self, response):
            for h3 in response.xpath('//h3').extract():
                yield MyItem(title=h3)

            for url in response.xpath('//a/@href').extract():
                yield scrapy.Request(url, callback=self.parse)
    
.. _spiderargs:

Spider arguments
================

Spiders can receive arguments that modify their behaviour. Some common uses for
spider arguments are to define the start URLs or to restrict the crawl to
certain sections of the site, but they can be used to configure any
functionality of the spider.

Spider arguments are passed through the :command:`crawl` command using the
``-a`` option. For example::

    scrapy crawl myspider -a category=electronics

Spiders receive arguments in their constructors::

    import scrapy

    class MySpider(scrapy.Spider):
        name = 'myspider'

        def __init__(self, category=None, *args, **kwargs):
            super(MySpider, self).__init__(*args, **kwargs)
            self.start_urls = ['http://www.example.com/categories/%s' % category]
            # ...

Spider arguments can also be passed through the Scrapyd ``schedule.json`` API.
See `Scrapyd documentation`_.

.. _builtin-spiders:
                
Generic Spiders
===============

Scrapy comes with some useful generic spiders that you can use, to subclass
your spiders from. Their aim is to provide convenient functionality for a few
common scraping cases, like following all links on a site based on certain
rules, crawling from `Sitemaps`_, or parsing a XML/CSV feed.

For the examples used in the following spiders, we'll assume you have a project
with a ``TestItem`` declared in a ``myproject.items`` module::

    import scrapy

    class TestItem(scrapy.Item):
        id = scrapy.Field()
        name = scrapy.Field()
        description = scrapy.Field()


.. currentmodule:: scrapy.spiders

CrawlSpider
-----------

.. class:: CrawlSpider

   爬取一般网站常用的spider。其定义了一些规则(rule)来提供跟进link的方便的机制。
   也许该spider并不是完全适合您的特定网站或项目，但其对很多情况都使用。
   因此您可以以其为起点，根据需求修改部分方法。当然您也可以实现自己的spider。

   除了从Spider继承过来的(您必须提供的)属性外，其提供了一个新的属性:

   .. attribute:: rules

      一个包含一个(或多个) :class:`Rule` 对象的集合(list)。
      每个 :class:`Rule` 对爬取网站的动作定义了特定表现。
      Rule对象在下边会介绍。
      如果多个rule匹配了相同的链接，则根据他们在本属性中被定义的顺序，第一个会被使用。

   该spider也提供了一个可复写(overrideable)的方法:

   .. method:: parse_start_url(response)

      当start_url的请求返回时，该方法被调用。
      该方法分析最初的返回值并必须返回一个
      :class:`~scrapy.item.Item` 对象或者
      一个 :class:`~scrapy.http.Request` 对象或者
      一个可迭代的包含二者对象。

爬取规则(Crawling rules)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. class:: Rule(link_extractor, callback=None, cb_kwargs=None, follow=None, process_links=None, process_request=None)

   ``link_extractor`` 是一个 :ref:`Link Extractor <topics-link-extractors>` 对象。
   其定义了如何从爬取到的页面提取链接。 

   ``callback`` 是一个callable或string(该spider中同名的函数将会被调用)。
   从link_extractor中每获取到链接时将会调用该函数。该回调函数接受一个response作为其第一个参数，
   并返回一个包含 :class:`~scrapy.item.Item` 以及(或) :class:`~scrapy.http.Request` 对象(或者这两者的子类)的列表(list)。

   .. warning:: 当编写爬虫规则时，请避免使用 ``parse`` 作为回调函数。
       由于 :class:`CrawlSpider` 使用 ``parse`` 方法来实现其逻辑，如果
       您覆盖了 ``parse`` 方法，crawl spider 将会运行失败。

   ``cb_kwargs`` 包含传递给回调函数的参数(keyword argument)的字典。

   ``follow`` 是一个布尔(boolean)值，指定了根据该规则从response提取的链接是否需要跟进。
   如果 ``callback`` 为None， ``follow`` 默认设置为 ``True`` ，否则默认为 ``False`` 。

   ``process_links`` 是一个callable或string(该spider中同名的函数将会被调用)。
   从link_extractor中获取到链接列表时将会调用该函数。该方法主要用来过滤。

   ``process_request`` 是一个callable或string(该spider中同名的函数将会被调用)。
   该规则提取到每个request时都会调用该函数。该函数必须返回一个request或者None。
   (用来过滤request)

CrawlSpider样例
~~~~~~~~~~~~~~~~~~~

接下来给出配合rule使用CrawlSpider的例子::

    import scrapy
    from scrapy.spiders import CrawlSpider, Rule
    from scrapy.linkextractors import LinkExtractor

    class MySpider(CrawlSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com']

        rules = (
            # 提取匹配 'category.php' (但不匹配 'subsection.php') 的链接并跟进链接(没有callback意味着follow默认为True)
            Rule(LinkExtractor(allow=('category\.php', ), deny=('subsection\.php', ))),

            # 提取匹配 'item.php' 的链接并使用spider的parse_item方法进行分析
            Rule(LinkExtractor(allow=('item\.php', )), callback='parse_item'),
        )

        def parse_item(self, response):
            self.logger.info('Hi, this is an item page! %s', response.url)

            item = scrapy.Item()
            item['id'] = response.xpath('//td[@id="item_id"]/text()').re(r'ID: (\d+)')
            item['name'] = response.xpath('//td[@id="item_name"]/text()').extract()
            item['description'] = response.xpath('//td[@id="item_description"]/text()').extract()
            return item


该spider将从example.com的首页开始爬取，获取category以及item的链接并对后者使用 ``parse_item`` 方法。
当item获得返回(response)时，将使用XPath处理HTML并生成一些数据填入 :class:`~scrapy.item.Item` 中。

XMLFeedSpider
-------------

.. class:: XMLFeedSpider

    XMLFeedSpider被设计用于通过迭代各个节点来分析XML源(XML feed)。
    迭代器可以从 ``iternodes`` ， ``xml`` ， ``html`` 选择。
    鉴于 ``xml`` 以及 ``html`` 迭代器需要先读取所有DOM再分析而引起的性能问题，
    一般还是推荐使用 ``iternodes`` 。
    不过使用 ``html`` 作为迭代器能有效应对错误的XML。

    您必须定义下列类属性来设置迭代器以及标签名(tag name):

    .. attribute:: iterator

        用于确定使用哪个迭代器的string。可选项有:

           - ``'iternodes'`` - 一个高性能的基于正则表达式的迭代器

           - ``'html'`` - 使用 :class:`~scrapy.selector.Selector` 的迭代器。
             需要注意的是该迭代器使用DOM进行分析，其需要将所有的DOM载入内存，
             当数据量大的时候会产生问题。

           - ``'xml'`` - 使用 :class:`~scrapy.selector.Selector` 的迭代器。
             需要注意的是该迭代器使用DOM进行分析，其需要将所有的DOM载入内存，
             当数据量大的时候会产生问题。

        默认值为 ``iternodes`` 。

    .. attribute:: itertag

        一个包含开始迭代的节点名的string。例如::

            itertag = 'product'

    .. attribute:: namespaces

        一个由 ``(prefix, url)`` 元组(tuple)所组成的list。
        其定义了在该文档中会被spider处理的可用的namespace。
        ``prefix`` 及 ``uri`` 会被自动调用
        :meth:`~scrapy.selector.Selector.register_namespace` 生成namespace。

        您可以通过在 :attr:`itertag` 属性中制定节点的namespace。

        例如::

            class YourSpider(XMLFeedSpider):

                namespaces = [('n', 'http://www.sitemaps.org/schemas/sitemap/0.9')]
                itertag = 'n:url'
                # ...

    除了这些新的属性之外，该spider也有以下可以覆盖(overrideable)的方法:

    .. method:: adapt_response(response)

        该方法在spider分析response前被调用。您可以在response被分析之前使用该函数来修改内容(body)。
        该方法接受一个response并返回一个response(可以相同也可以不同)。

    .. method:: parse_node(response, selector)
       
        当节点符合提供的标签名时(``itertag``)该方法被调用。
        接收到的response以及相应的 :class:`~scrapy.selector.Selector` 作为参数传递给该方法。
        该方法返回一个 :class:`~scrapy.item.Item` 对象或者
        :class:`~scrapy.http.Request` 对象 或者一个包含二者的可迭代对象(iterable)。

    .. method:: process_results(response, results)
       
        当spider返回结果(item或request)时该方法被调用。
        设定该方法的目的是在结果返回给框架核心(framework core)之前做最后的处理，
        例如设定item的ID。其接受一个结果的列表(list of results)及对应的response。
        其结果必须返回一个结果的列表(list of results)(包含Item或者Request对象)。


XMLFeedSpider例子
~~~~~~~~~~~~~~~~~~~~~

该spider十分易用。下边是其中一个例子::

    from scrapy.spiders import XMLFeedSpider
    from myproject.items import TestItem

    class MySpider(XMLFeedSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com/feed.xml']
        iterator = 'iternodes' # This is actually unnecessary, since it's the default value
        itertag = 'item'

        def parse_node(self, response, node):
            self.logger.info('Hi, this is a <%s> node!: %s', self.itertag, ''.join(node.extract()))

            item = TestItem()
            item['id'] = node.xpath('@id').extract()
            item['name'] = node.xpath('name').extract()
            item['description'] = node.xpath('description').extract()
            return item

简单来说，我们在这里创建了一个spider，从给定的 ``start_urls`` 中下载feed，
并迭代feed中每个 ``item`` 标签，输出，并在 :class:`~scrapy.item.Item` 中存储有些随机数据。

CSVFeedSpider
-------------

.. class:: CSVFeedSpider

   该spider除了其按行遍历而不是节点之外其他和XMLFeedSpider十分类似。
   而其在每次迭代时调用的是 :meth:`parse_row` 。

   .. attribute:: delimiter
       
       在CSV文件中用于区分字段的分隔符。类型为string。
       默认为 ``','`` (逗号)。

   .. attribute:: quotechar

       A string with the enclosure character for each field in the CSV file
       Defaults to ``'"'`` (quotation mark).

   .. attribute:: headers
      
       在CSV文件中包含的用来提取字段的行的列表。参考下边的例子。

   .. method:: parse_row(response, row)
      
       该方法接收一个response对象及一个以提供或检测出来的header为键的字典(代表每行)。
       该spider中，您也可以覆盖 ``adapt_response`` 及 
       ``process_results`` 方法来进行预处理(pre-processing)及后(post-processing)处理。

CSVFeedSpider例子
~~~~~~~~~~~~~~~~~~~~~

下面的例子和之前的例子很像，但使用了
:class:`CSVFeedSpider`::

    from scrapy.spiders import CSVFeedSpider
    from myproject.items import TestItem

    class MySpider(CSVFeedSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com/feed.csv']
        delimiter = ';'
        quotechar = "'"
        headers = ['id', 'name', 'description']

        def parse_row(self, response, row):
            self.logger.info('Hi, this is a row!: %r', row)

            item = TestItem()
            item['id'] = row['id']
            item['name'] = row['name']
            item['description'] = row['description']
            return item


SitemapSpider
-------------

.. class:: SitemapSpider

    SitemapSpider使您爬取网站时可以通过 `Sitemaps`_ 来发现爬取的URL。

    其支持嵌套的sitemap，并能从 `robots.txt`_ 中获取sitemap的url。

    .. attribute:: sitemap_urls

        包含您要爬取的url的sitemap的url列表(list)。
        您也可以指定为一个 `robots.txt`_ ，spider会从中分析并提取url。

    .. attribute:: sitemap_rules

        一个包含 ``(regex, callback)`` 元组的列表(list):

        * ``regex`` 是一个用于匹配从sitemap提供的url的正则表达式。
          ``regex`` 可以是一个字符串或者编译的正则对象(compiled regex object)。

        * callback指定了匹配正则表达式的url的处理函数。
          ``callback`` 可以是一个字符串(spider中方法的名字)或者是callable。

        例如::

            sitemap_rules = [('/product/', 'parse_product')]

        规则按顺序进行匹配，之后第一个匹配才会被应用。

        如果您忽略该属性，sitemap中发现的所有url将会被 ``parse`` 函数处理。

    .. attribute:: sitemap_follow

        一个用于匹配要跟进的sitemap的正则表达式的列表(list)。其仅仅被应用在
        使用 `Sitemap index files` 来指向其他sitemap文件的站点。

        默认情况下所有的sitemap都会被跟进。

    .. attribute:: sitemap_alternate_links

        指定当一个 ``url`` 有可选的链接时，是否跟进。
        有些非英文网站会在一个 ``url`` 块内提供其他语言的网站链接。
        
        例如::
       
            <url>
                <loc>http://example.com/</loc>
                <xhtml:link rel="alternate" hreflang="de" href="http://example.com/de"/>
            </url>

        当 ``sitemap_alternate_links`` 设置时，两个URL都会被获取。
        当 ``sitemap_alternate_links`` 关闭时，只有 ``http://example.com/`` 会被获取。

        默认 ``sitemap_alternate_links`` 关闭。


SitemapSpider样例
~~~~~~~~~~~~~~~~~~~~~~

简单的例子: 使用 ``parse`` 处理通过sitemap发现的所有url::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/sitemap.xml']

        def parse(self, response):
            pass # ... scrape item here ...

用特定的函数处理某些url，其他的使用另外的callback::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/sitemap.xml']
        sitemap_rules = [
            ('/product/', 'parse_product'),
            ('/category/', 'parse_category'),
        ]

        def parse_product(self, response):
            pass # ... scrape product ...

        def parse_category(self, response):
            pass # ... scrape category ...

跟进 `robots.txt`_ 文件定义的sitemap并只跟进包含有 ``..sitemap_shop`` 的url::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/robots.txt']
        sitemap_rules = [
            ('/shop/', 'parse_shop'),
        ]
        sitemap_follow = ['/sitemap_shops']

        def parse_shop(self, response):
            pass # ... scrape shop here ...

在SitemapSpider中使用其他url::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/robots.txt']
        sitemap_rules = [
            ('/shop/', 'parse_shop'),
        ]

        other_urls = ['http://www.example.com/about']

        def start_requests(self):
            requests = list(super(MySpider, self).start_requests())
            requests += [scrapy.Request(x, self.parse_other) for x in self.other_urls]
            return requests

        def parse_shop(self, response):
            pass # ... scrape shop here ...

        def parse_other(self, response):
            pass # ... scrape other here ...

.. _Sitemaps: http://www.sitemaps.org
.. _Sitemap index files: http://www.sitemaps.org/protocol.html#index
.. _robots.txt: http://www.robotstxt.org/
.. _后缀: http://en.wikipedia.org/wiki/Top-level_domain
.. _Scrapyd documentation: http://scrapyd.readthedocs.org/en/latest/
