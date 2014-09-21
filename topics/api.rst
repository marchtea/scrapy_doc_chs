.. _topics-api:

========
核心API
========

.. versionadded:: 0.15

该节文档讲述Scrapy核心API，目标用户是开发Scrapy扩展(extensions)和中间件(middlewares)的开发人员。

.. _topics-api-crawler:

Crawler API
===========

Scrapy API的主要入口是 :class:`~scrapy.crawler.Crawler` 的实例对象，
通过类方法 ``from_crawler`` 将它传递给扩展(extensions)。
该对象提供对所有Scrapy核心组件的访问，
也是扩展访问Scrapy核心组件和挂载功能到Scrapy的唯一途径。

.. module:: scrapy.crawler
   :synopsis: The Scrapy crawler

Extension Manager负责加载和跟踪已经安装的扩展，
它通过 :setting:`EXTENSIONS` 配置，包含一个所有可用扩展的字典，
字典的顺序跟你在 :ref:`configure the downloader middlewares <topics-downloader-middleware-setting>` 配置的顺序一致。

.. class:: Crawler(spidercls, settings)

    Crawler必须使用 
    :class:`scrapy.spider.Spider` 子类及
    :class:`scrapy.settings.Settings` 的对象进行实例化

    .. attribute:: settings

        crawler的配置管理器。

        扩展(extensions)和中间件(middlewares)使用它用来访问Scrapy的配置。

        关于Scrapy配置的介绍参考这里 :ref:`topics-settings`。

        API参考 :class:`~scrapy.settings.Settings`。

    .. attribute:: signals

        crawler的信号管理器。

        扩展和中间件使用它将自己的功能挂载到Scrapy。

        关于信号的介绍参考 :ref:`topics-signals`。

        API参考 :class:`~scrapy.signalmanager.SignalManager`。

    .. attribute:: stats

        crawler的统计信息收集器。

        扩展和中间件使用它记录操作的统计信息，或者访问由其他扩展收集的统计信息。

        关于统计信息收集器的介绍参考 :ref:`topics-stats`。

        API参考类 :class:`~scrapy.statscol.StatsCollector` class。

    .. attribute:: extensions

        扩展管理器，跟踪所有开启的扩展。

        大多数扩展不需要访问该属性。

        关于扩展和可用扩展列表器的介绍参考 :ref:`topics-extensions`。

    .. attribute:: engine

        执行引擎，协调crawler的核心逻辑，包括调度，下载和spider。

        某些扩展可能需要访问Scrapy的引擎属性，以修改检查(modify inspect)或修改下载器和调度器的行为，
        这是该API的高级使用，但还不稳定。

    .. attribute:: spider
        正在爬取的spider。该spider类的实例由创建crawler时所提供，
        在调用 :meth:`crawl` 方法是所创建。

    .. method:: crawl(\*args, \**kwargs)

        根据给定的
        `args` , `kwargs` 的参数来初始化spider类，启动执行引擎，启动crawler。

        返回一个延迟deferred对象，当爬取结束时触发它。

.. class:: CrawlerRunner(settings)

    This is a convenient helper class that creates, configures and runs
    crawlers inside an already setup Twisted `reactor`_.

    The CrawlerRunner object must be instantiated with a
    :class:`~scrapy.settings.Settings` object.

    This class shouldn't be needed (since Scrapy is responsible of using it
    accordingly) unless writing scripts that manually handle the crawling
    process. See :ref:`run-from-script` for an example.

    .. attribute:: crawlers

       Set of :class:`crawlers <scrapy.crawler.Crawler>` created by the
       :meth:`crawl` method.

    .. attribute:: crawl_deferreds

       Set of the `deferreds`_ return by the :meth:`crawl` method. This
       collection it's useful for keeping track of current crawling state.

    .. method:: crawl(spidercls, \*args, \**kwargs)

       This method sets up the crawling of the given `spidercls` with the
       provided arguments.

       It takes care of loading the spider class while configuring and starting
       a crawler for it.

       Returns a deferred that is fired when the crawl is finished.

       :param spidercls: spider class or spider's name inside the project
       :type spidercls: :class:`~scrapy.spider.Spider` subclass or str

       :param args: arguments to initializate the spider
       :type args: list

       :param kwargs: keyword arguments to initializate the spider
       :type kwargs: dict

    .. method:: stop()

       Stops simultaneously all the crawling jobs taking place.

       Returns a deferred that is fired when they all have ended.

.. _topics-api-settings:

设置(Settings) API
=======================

.. module:: scrapy.settings
   :synopsis: Settings manager

.. attribute:: SETTINGS_PRIORITIES

    Dictionary that sets the key name and priority level of the default
    settings priorities used in Scrapy.

    Each item defines a settings entry point, giving it a code name for
    identification and an integer priority. Greater priorities take more
    precedence over lesser ones when setting and retrieving values in the
    :class:`~scrapy.settings.Settings` class.

    .. highlight:: python

    ::

        SETTINGS_PRIORITIES = {
            'default': 0,
            'command': 10,
            'project': 20,
            'spider': 30,
            'cmdline': 40,
        }

    For a detailed explanation on each settings sources, see:
    :ref:`topics-settings`.

.. class:: Settings(values={}, priority='project')

    This object stores Scrapy settings for the configuration of internal
    components, and can be used for any further customization.

    After instantiation of this class, the new object will have the global
    default settings described on :ref:`topics-settings-ref` already
    populated.

    Additional values can be passed on initialization with the ``values``
    argument, and they would take the ``priority`` level.  If the latter
    argument is a string, the priority name will be looked up in
    :attr:`~scrapy.settings.SETTINGS_PRIORITIES`. Otherwise, a expecific
    integer should be provided.

    Once the object is created, new settings can be loaded or updated with the
    :meth:`~scrapy.settings.Settings.set` method, and can be accessed with the
    square bracket notation of dictionaries, or with the
    :meth:`~scrapy.settings.Settings.get` method of the instance and its value
    conversion variants.  When requesting a stored key, the value with the
    highest priority will be retrieved.

    .. method:: set(name, value, priority='project')

       Store a key/value attribute with a given priority.
       Settings should be populated *before* configuring the Crawler object
       (through the :meth:`~scrapy.crawler.Crawler.configure` method),
       otherwise they won't have any effect.

       :param name: the setting name
       :type name: string

       :param value: the value to associate with the setting
       :type value: any

       :param priority: the priority of the setting. Should be a key of
           :attr:`~scrapy.settings.SETTINGS_PRIORITIES` or an integer
       :type priority: string or int

    .. method:: setdict(values, priority='project')

       Store key/value pairs with a given priority.

       This is a helper function that calls
       :meth:`~scrapy.settings.Settings.set` for every item of ``values``
       with the provided ``priority``.

       :param values: the settings names and values
       :type values: dict

       :param priority: the priority of the settings. Should be a key of
           :attr:`~scrapy.settings.SETTINGS_PRIORITIES` or an integer
       :type priority: string or int

    .. method:: setmodule(module, priority='project')

       Store settings from a module with a given priority.

       This is a helper function that calls
       :meth:`~scrapy.settings.Settings.set` for every globally declared
       uppercase variable of ``module`` with the provided ``priority``.

       :param module: the module or the path of the module
       :type module: module object or string

       :param priority: the priority of the settings. Should be a key of
           :attr:`~scrapy.settings.SETTINGS_PRIORITIES` or an integer
       :type priority: string or int

    .. method:: get(name, default=None)

       获取某项配置的值，且不修改其原有的值。

       :param name: 配置名
       :type name: 字符串

       :param default: 如果没有该项配置时返回的缺省值
       :type default: 任何

    .. method:: getbool(name, default=False)

       return ``False``
       将某项配置的值以布尔值形式返回。比如，``1`` 和 ``'1'``，``True`` 都返回``True``，
       而 ``0``，``'0'``，``False`` 和 ``None`` 返回 ``False``。

       比如，通过环境变量计算将某项配置设置为 ``'0'``，通过该方法获取得到 ``False``。

       :param name: 配置名
       :type name: 字符串

       :param default: 如果该配置项未设置，返回的缺省值
       :type default: 任何

    .. method:: getint(name, default=0)

       将某项配置的值以整数形式返回

       :param name: 配置名
       :type name: 字符串

       :param default: 如果该配置项未设置，返回的缺省值
       :type default: 任何

    .. method:: getfloat(name, default=0.0)

       将某项配置的值以浮点数形式返回

       :param name: 配置名
       :type name: 字符串

       :param default: 如果该配置项未设置，返回的缺省值
       :type default: 任何

    .. method:: getlist(name, default=None)

       将某项配置的值以列表形式返回。如果配置值本来就是list则将返回其拷贝。
       如果是字符串，则返回被 "," 分割后的列表。

       比如，某项值通过环境变量的计算被设置为 ``'one,two'`` ，该方法返回['one', 'two']。

       :param name: 配置名
       :type name: 字符串

       :param default: 如果该配置项未设置，返回的缺省值
       :type default: 任何类型

    .. method:: getdict(name, default=None)

       Get a setting value as a dictionary. If the setting original type is a
       dictionary, a copy of it will be returned. If it's a string it will
       evaluated as a json dictionary.

       :param name: the setting name
       :type name: string

       :param default: the value to return if no setting is found
       :type default: any

    .. method:: copy()

       Make a deep copy of current settings.

       This method returns a new instance of the :class:`Settings` class,
       populated with the same values and their priorities.

       Modifications to the new object won't be reflected on the original
       settings.

    .. method:: freeze()

       Disable further changes to the current settings.

       After calling this method, the present state of the settings will become
       immutable. Trying to change values through the :meth:`~set` method and
       its variants won't be possible and will be alerted.

    .. method:: frozencopy()

       Return an immutable copy of the current settings.

       Alias for a :meth:`~freeze` call in the object returned by :meth:`copy`

.. _topics-api-spidermanager:

SpiderManager API
=================

.. module:: scrapy.spidermanager
   :synopsis: The spider manager

.. class:: SpiderManager

    This class is in charge of retrieving and handling the spider classes
    defined across the project.

    Custom spider managers can be employed by specifying their path in the
    :setting:`SPIDER_MANAGER_CLASS` project setting. They must fully implement
    the :class:`scrapy.interfaces.ISpiderManager` interface to guarantee an
    errorless execution.

    .. method:: from_settings(settings)

       This class method is used by Scrapy to create an instance of the class.
       It's called with the current project settings, and it loads the spiders
       found in the modules of the :setting:`SPIDER_MODULES` setting.

       :param settings: project settings
       :type settings: :class:`~scrapy.settings.Settings` instance

    .. method:: load(spider_name)

       Get the Spider class with the given name. It'll look into the previously
       loaded spiders for a spider class with name `spider_name` and will raise
       a KeyError if not found.

       :param spider_name: spider class name
       :type spider_name: str

    .. method:: list()

       Get the names of the available spiders in the project.

    .. method:: find_by_request(request)

       List the spiders' names that can handle the given request. Will try to
       match the request's url against the domains of the spiders.

       :param request: queried request
       :type request: :class:`~scrapy.http.Request` instance


.. _topics-api-signals:

信号(Signals) API
=====================

.. module:: scrapy.signalmanager
   :synopsis: The signal manager

.. class:: SignalManager

    .. method:: connect(receiver, signal)

        链接一个接收器函数(receiver function) 到一个信号(signal)。

        signal可以是任何对象，虽然Scrapy提供了一些预先定义好的信号，
        参考文档 :ref:`topics-signals`。

        :param receiver: 被链接到的函数
        :type receiver: 可调用对象

        :param signal: 链接的信号
        :type signal: 对象

    .. method:: send_catch_log(signal, \*\*kwargs)

        发送一个信号，捕获异常并记录日志。

        关键字参数会传递给信号处理者(signal handlers)(通过方法 :meth:`connect` 关联)。

    .. method:: send_catch_log_deferred(signal, \*\*kwargs)

        跟 :meth:`send_catch_log` 相似但支持返回 `deferreds`_ 形式的信号处理器。

        返回一个 `deferred`_ ，当所有的信号处理器的延迟被触发时调用。
        发送一个信号，处理异常并记录日志。

        关键字参数会传递给信号处理者(signal handlers)(通过方法 :meth:`connect` 关联)。

    .. method:: disconnect(receiver, signal)

        解除一个接收器函数和一个信号的关联。这跟方法 :meth:`connect` 有相反的作用，
        参数也相同。

    .. method:: disconnect_all(signal)

        取消给定信号绑定的所有接收器。

        :param signal: 要取消绑定的信号
        :type signal: object

.. _topics-api-stats:

状态收集器(Stats Collector) API
===================================

模块 `scrapy.statscol` 下有好几种状态收集器，
它们都实现了状态收集器API对应的类 :class:`~scrapy.statscol.Statscollector` (即它们都继承至该类)。

.. module:: scrapy.statscol
   :synopsis: Stats Collectors

.. class:: StatsCollector

    .. method:: get_value(key, default=None)

        返回指定key的统计值，如果key不存在则返回缺省值。

    .. method:: get_stats()

        以dict形式返回当前spider的所有统计值。

    .. method:: set_value(key, value)

        设置key所指定的统计值为value。

    .. method:: set_stats(stats)

        使用dict形式的 ``stats`` 参数覆盖当前的统计值。

    .. method:: inc_value(key, count=1, start=0)

        增加key所对应的统计值，增长值由count指定。
        如果key未设置，则使用start的值设置为初始值。

    .. method:: max_value(key, value)

        如果key所对应的当前value小于参数所指定的value，则设置value。
        如果没有key所对应的value，设置value。

    .. method:: min_value(key, value)

        如果key所对应的当前value大于参数所指定的value，则设置value。
        如果没有key所对应的value，设置value。

    .. method:: clear_stats()

        清除所有统计信息。

    以下方法不是统计收集api的一部分，但实现自定义的统计收集器时会使用到：

    .. method:: open_spider(spider)

        打开指定spider进行统计信息收集。

    .. method:: close_spider(spider)

        关闭指定spider。调用后，不能访问和收集统计信息。

.. _deferreds: http://twistedmatrix.com/documents/current/core/howto/defer.html
.. _deferred: http://twistedmatrix.com/documents/current/core/howto/defer.html
.. _reactor: http://twistedmatrix.com/documents/current/core/howto/reactor-basics.html
