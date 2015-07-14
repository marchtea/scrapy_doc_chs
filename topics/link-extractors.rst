.. _topics-link-extractors:

===============
Link Extractors
===============

Link Extractors 是那些目的仅仅是从网页(:class:`scrapy.http.Response` 对象)中抽取最终将会被follow链接的对象｡

Scrapy提供了 ``scrapy.linkextractors import LinkExtractor`` , 但你通过实现一个简单的接口创建自己定制的Link Extractor来满足需求｡


每个link extractor有唯一的公共方法是 ``extract_links`` ,它接收一个 :class:`~scrapy.http.Response` 对象,并返回一个 :class:`scrapy.link.Link` 对象｡Link Extractors,要实例化一次并且 ``extract_links`` 方法会根据不同的response调用多次提取链接｡


Link Extractors在 :class:`~scrapy.spiders.CrawlSpider` 类(在Scrapy可用)中使用,
通过一套规则,但你也可以用它在你的Spider中,
即使你不是从 
:class:`~scrapy.spiders.CrawlSpider` 
继承的子类, 因为它的目的很简单: 提取链接｡


.. _topics-link-extractors-ref:

内置Link Extractor 参考
==================================

.. module:: scrapy.linkextractors
   :synopsis: Link extractors classes

Scrapy提供的Link Extractor类在 :mod:`scrapy.linkextractors` 模块提供｡
默认的link extractor是 ``LinkExtractor`` , 其实就是 :class:`~.LxmlLinkExtractor`::

    from scrapy.linkextractors import LinkExtractor

There used to be other link extractor classes in previous Scrapy versions,
but they are deprecated now.

LxmlLinkExtractor
-----------------

.. module:: scrapy.linkextractors.lxmlhtml
   :synopsis: lxml's HTMLParser-based link extractors


.. class:: LxmlLinkExtractor(allow=(), deny=(), allow_domains=(), deny_domains=(), deny_extensions=None, restrict_xpaths=(), restrict_css=(), tags=('a', 'area'), attrs=('href',), canonicalize=True, unique=True, process_value=None)


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
        ``IGNORED_EXTENSIONS`` list defined in the `scrapy.linkextractors`_
        package.
    :type deny_extensions: list

    :param restrict_xpaths: is a XPath (or list of XPath's) which defines
        regions inside the response where links should be extracted from.
        If given, only the text selected by those XPath will be scanned for
        links. See examples below.
    :type restrict_xpaths: str or list

    :param restrict_css: a CSS selector (or list of selectors) which defines
        regions inside the response where links should be extracted from.
        Has the same behaviour as ``restrict_xpaths``.
    :type restrict_css: str or list

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

.. _scrapy.linkextractors: https://github.com/scrapy/scrapy/blob/master/scrapy/linkextractors/__init__.py
