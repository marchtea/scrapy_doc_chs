.. _topics-link-extractors:

===============
Link Extractors
===============

Link Extractors 是那些目的仅仅是从网页(:class:`scrapy.http.Response` 对象)中抽取最终将会被follow链接的对象｡


Scrapy默认提供2种可用的 Link Extractor, 但你通过实现一个简单的接口创建自己定制的Link Extractor来满足需求｡


每个LinkExtractor有唯一的公共方法是 ``extract_links`` ,它接收一个 :class:`~scrapy.http.Response` 对象,并返回一个 :class:`scrapy.link.Link` 对象｡Link Extractors,要实例化一次并且 ``extract_links`` 方法会根据不同的response调用多次提取链接｡


Link Extractors在 :class:`~scrapy.contrib.spiders.CrawlSpider` 类(在Scrapy可用)中使用, 通过一套规则,但你也可以用它在你的Spider中,即使你不是从 :class:`~scrapy.contrib.spiders.CrawlSpider` 继承的子类, 因为它的目的很简单: 提取链接｡


.. _topics-link-extractors-ref:

内置Link Extractor 参考
==================================

.. module:: scrapy.contrib.linkextractors
   :synopsis: Link extractors classes

所有与Scrapy绑定且可用的Link Extractors类在 :mod:`scrapy.contrib.linkextractors` 模块提供｡
如果您不知道选择哪个link extractor,使用默认的即可(其实就是LxmlLinkExtractor(参照下面))::

    from scrapy.contrib.linkextractors import LinkExtractor


LxmlLinkExtractor
-----------------

.. module:: scrapy.contrib.linkextractors.lxmlhtml
   :synopsis: lxml's HTMLParser-based link extractors


.. class:: LxmlLinkExtractor(allow=(), deny=(), allow_domains=(), deny_domains=(), deny_extensions=None, restrict_xpaths=(), tags=('a', 'area'), attrs=('href',), canonicalize=True, unique=True, process_value=None)


    LxmlLinkExtractor is the recommended link extractor with handy filtering
    options. It is implemented using lxml's robust HTMLParser.

    :param allow: a single regular expression (or list of regular expressions)
        that the (absolute) urls must match in order to be extracted. If not
        given (or empty), it will match all links.
    :type allow: a regular expression (or list of)

    :param deny: a single regular expression (or list of regular expressions)
        that the (absolute) urls must match in order to be excluded (ie. not
        extracted). It has precedence over the ``allow`` parameter. If not
        given (or empty) it won't exclude any links.
    :type deny: a regular expression (or list of)

    :param allow_domains: a single value or a list of string containing
        domains which will be considered for extracting the links
    :type allow_domains: str or list

    :param deny_domains: a single value or a list of strings containing
        domains which won't be considered for extracting the links
    :type deny_domains: str or list

    :param deny_extensions: a single value or list of strings containing
        extensions that should be ignored when extracting links.
        If not given, it will default to the
        ``IGNORED_EXTENSIONS`` list defined in the `scrapy.linkextractor`_
        module.
    :type deny_extensions: list

    :param restrict_xpaths: is a XPath (or list of XPath's) which defines
        regions inside the response where links should be extracted from.
        If given, only the text selected by those XPath will be scanned for
        links. See examples below.
    :type restrict_xpaths: str or list

    :param tags: a tag or a list of tags to consider when extracting links.
        Defaults to ``('a', 'area')``.
    :type tags: str or list

    :param attrs: an attribute or list of attributes which should be considered when looking
        for links to extract (only for those tags specified in the ``tags``
        parameter). Defaults to ``('href',)``
    :type attrs: list

    :param canonicalize: canonicalize each extracted url (using
        scrapy.utils.url.canonicalize_url). Defaults to ``True``.
    :type canonicalize: boolean

    :param unique: whether duplicate filtering should be applied to extracted
        links.
    :type unique: boolean

    :param process_value: see ``process_value`` argument of
        :class:`BaseSgmlLinkExtractor` class constructor
    :type process_value: callable


SgmlLinkExtractor
-----------------

.. module:: scrapy.contrib.linkextractors.sgml
   :synopsis: SGMLParser-based link extractors

.. warning:: SGMLParser based link extractors are unmantained and its usage is discouraged.
    It is recommended to migrate to :class:`LxmlLinkExtractor` if you are still
    using :class:`SgmlLinkExtractor`.

.. class:: SgmlLinkExtractor(allow=(), deny=(), allow_domains=(), deny_domains=(), deny_extensions=None, restrict_xpaths=(), tags=('a', 'area'), attrs=('href'), canonicalize=True, unique=True, process_value=None)

    SgmlLinkExtractor继承于 :class:`BaseSgmlLinkExtractor`,其提供了过滤器(filter),以便于提取包括符合正则表达式的链接。
    过滤器通过以下构造函数的参数配置:

    :param allow: 必须要匹配这个正则表达式(或正则表达式列表)的URL才会被提取｡如果没有给出(或为空), 它会匹配所有的链接｡

    :type allow: a regular expression (or list of)

    :param deny: 与这个正则表达式(或正则表达式列表)的(绝对)不匹配的URL必须被排除在外(即不提取)｡它的优先级高于 ``allow`` 的参数｡如果没有给出(或None), 将不排除任何链接｡

    :type deny: a regular expression (or list of)

    :param allow_domains: 单值或者包含字符串域的列表表示会被提取的链接的domains｡
    :type allow_domains: str or list

    :param deny_domains: 单值或包含域名的字符串,将不考虑提取链接的domains｡
    :type deny_domains: str or list

    :param deny_extensions: 应提取链接时,可以忽略扩展名的列表｡如果没有给出, 它会默认为 `scrapy.linkextractor`_ 模块中定义的 ``IGNORED_EXTENSIONS`` 列表｡
    :type deny_extensions: list

    :param restrict_xpaths: 一个的XPath (或XPath的列表),它定义了链路应该从提取的响应内的区域｡如果给定的,只有那些XPath的选择的文本将被扫描的链接｡见下面的例子｡
    :type restrict_xpaths: str or list

    :param tags: 提取链接时要考虑的标记或标记列表｡默认为 ``( 'a' , 'area')`` ｡
    :type tags: str or list
 
    :param attrs: 提取链接时应该寻找的attrbitues列表(仅在 ``tag`` 参数中指定的标签)｡默认为 ``('href')``｡

    :type attrs: list

    :param canonicalize: 规范化每次提取的URL(使用scrapy.utils.url.canonicalize_url )｡默认为 ``True`` ｡
    :type canonicalize: boolean

    :param unique: 重复过滤是否应适用于提取的链接｡
    :type unique: boolean

    :param process_value: 见:class:`BaseSgmlLinkExtractor` 类的构造函数 ``process_value`` 参数｡
    :type process_value: callable

BaseSgmlLinkExtractor
---------------------

.. class:: BaseSgmlLinkExtractor(tag="a", attr="href", unique=False, process_value=None)

    这个Link Extractor的目的只是充当了Sgml Link Extractor的基类｡你应该使用 :class:`SgmlLinkExtractor`｡

    
    该构造函数的参数是:

    :param tag: 是一个字符串(带标签的名称)或接收一个标签名, 如果链接应该从标签中提取返回 ``True`` 的函数或 ``False`` 如果他们不应该｡默认为 ``'a'`` ｡请求(一旦它被下载)作为其第一个参数｡欲了解更多信息, 请参阅 :ref:`topics-request-response-ref-request-callback-arguments`｡
    :type tag: str or callable

    :param attr:  无论是字符串(带有tag属性的名称), 或接收到一个属性名称, 如果链接应该从中提取返回 ``True`` 的函数或 ``False`` 如果他们不应该｡默认设置为 ``href`` ｡
    :type attr: str or callable

    :param unique: 是一个布尔值,指定是否重复过滤, 应用于提取链接｡
    :type unique: boolean

    :param process_value: 它接收来自扫描标签和属性提取每个值, 可以修改该值, 并返回一个新的, 或返回 ``None`` 完全忽略链接的功能｡如果没有给出,  ``process_value`` 默认是 ``lambda x: x``｡

        .. highlight:: html

        例如,从这段代码中提取链接::

            <a href="javascript:goToPage('../other/page.html'); return false">Link text</a>
        
        .. highlight:: python

        你可以使用下面的这个 ``process_value`` 函数::
        
            def process_value(value):
                m = re.search("javascript:goToPage\('(.*?)'", value)
                if m:
                    return m.group(1) 

    :type process_value: callable

.. _scrapy.linkextractor: https://github.com/scrapy/scrapy/blob/master/scrapy/linkextractor.py
