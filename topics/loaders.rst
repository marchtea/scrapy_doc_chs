.. _topics-loaders:

============
Item Loaders
============

.. module:: scrapy.contrib.loader
   :synopsis: Item Loader class

Item Loaders提供了一种便捷的方式填充抓取到的 ::ref:`Items<topics-items>` 。
虽然Items可以使用自带的类字典形式API填充，但是Items Loaders提供了更便捷的API，
可以分析原始数据并对Item进行赋值。

从另一方面来说， :ref:`Items <topics-items>` 提供保存抓取数据的 *容器* ，
而 Item Loaders提供的是 *填充* 容器的机制。

Item Loaders提供的是一种灵活，高效的机制，可以更方便的被spider或source format
(HTML, XML, etc)扩展，并override更易于维护的、不同的内容分析规则。

Using Item Loaders to populate items
====================================

要使用Item Loader, 你必须先将它实例化. 你可以使用类似字典的对象(例如: Item or dict)来进行实例化, 或者不使用对象也可以, 当不用对象进行实例化的时候,Item会自动使用 :attr:`ItemLoader.default_item_class`
属性中指定的Item 类在Item Loader constructor中实例化.

然后,你开始收集数值到Item Loader时,通常使用
:ref:`Selectors <topics-selectors>`. 你可以在同一个item field 里面添加多个数值;Item Loader将知道如何用合适的处理函数来“添加”这些数值.

下面是在 :ref:`Spider <topics-spiders>` 中典型的Item Loader的用法, 使用 :ref:`Items chapter <topics-items>` 中声明的 :ref:`Product item <topics-items-declaring>`::

    from scrapy.contrib.loader import ItemLoader
    from myproject.items import Product

    def parse(self, response):
        l = ItemLoader(item=Product(), response=response)
        l.add_xpath('name', '//div[@class="product_name"]')
        l.add_xpath('name', '//div[@class="product_title"]')
        l.add_xpath('price', '//p[@id="price"]')
        l.add_css('stock', 'p#stock]')
        l.add_value('last_updated', 'today') # you can also use literal values
        return l.load_item()

快速查看这些代码之后,我们可以看到发现 ``name``  字段被从页面中两个不同的XPath位置提取:

1. ``//div[@class="product_name"]``
2. ``//div[@class="product_title"]``

换言之,数据通过用 :meth:`~ItemLoader.add_xpath` 的方法,把从两个不同的XPath位置提取的数据收集起来. 这是将在以后分配给 ``name`` 字段中的数据｡


之后,类似的请求被用于 ``price`` 和 ``stock`` 字段
(后者使用 CSS selector 和 :meth:`~ItemLoader.add_css` 方法),
最后使用不同的方法 :meth:`~ItemLoader.add_value` 对 ``last_update`` 填充文本值( ``today`` ).

最终, 当所有数据被收集起来之后, 调用 :meth:`ItemLoader.load_item` 方法, 实际上填充并且返回了之前通过调用 :meth:`~ItemLoader.add_xpath`,
:meth:`~ItemLoader.add_css`, and :meth:`~ItemLoader.add_value` 所提取和收集到的数据的Item.

.. _topics-loaders-processors:

Input and Output processors
===========================

Item Loader在每个(Item)字段中都包含了一个输入处理器和一个输出处理器｡ 输入处理器收到数据时立刻提取数据 (通过 :meth:`~ItemLoader.add_xpath`, :meth:`~ItemLoader.add_css` 或者
:meth:`~ItemLoader.add_value` 方法) 之后输入处理器的结果被收集起来并且保存在ItemLoader内. 收集到所有的数据后, 调用
:meth:`ItemLoader.load_item` 方法来填充,并得到填充后的
:class:`~scrapy.item.Item` 对象.  这是当输出处理器被和之前收集到的数据(和用输入处理器处理的)被调用.输出处理器的结果是被分配到Item的最终值｡


让我们看一个例子来说明如何输入和输出处理器被一个特定的字段调用(同样适用于其他field):::

    l = ItemLoader(Product(), some_selector)
    l.add_xpath('name', xpath1) # (1)
    l.add_xpath('name', xpath2) # (2)
    l.add_css('name', css) # (3)
    l.add_value('name', 'test') # (4)
    return l.load_item() # (5)

发生了这些事情:

1. 从 ``xpath1`` 提取出的数据,传递给 *输入处理器* 的 ``name`` 字段.输入处理器的结果被收集和保存在Item Loader中(但尚未分配给该Item)｡

2. 从 ``xpath2`` 提取出来的数据,传递给(1)中使用的相同的 *输入处理器* .输入处理器的结果被附加到在(1)中收集的数据(如果有的话) ｡

3. 和之前相似，只不过这里的数据是通过 ``css`` CSS selector抽取，之后传输到在(1)和(2)使用
   的 *input processor* 中。最终输入处理器的结果被附加到在(1)和(2)中收集的数据之后
   (如果存在数据的话)。

4. 这里的处理方式也和之前相似，但是此处的值是通过add_value直接赋予的，
   而不是利用XPath表达式或CSS selector获取。得到的值仍然是被传送到输入处理器。
   在这里例程中，因为得到的值并非可迭代，所以在传输到输入处理器之前需要将其
   转化为可迭代的单个元素，这才是它所接受的形式。

5. 在之前步骤中所收集到的数据被传送到 *output processor* 的 ``name`` field中。
   输出处理器的结果就是赋到item中 ``name`` field的值。

需要注意的是，输入和输出处理器都是可调用对象，调用时传入需要被分析的数据，
处理后返回分析得到的值。因此你可以使用任意函数作为输入、输出处理器。
唯一需注意的是它们必须接收一个（并且只是一个）迭代器性质的positional参数。

.. note:: Both input and output processors must receive an iterator as their
   first argument. The output of those functions can be anything. The result of
   input processors will be appended to an internal list (in the Loader)
   containing the collected values (for that field). The result of the output
   processors is the value that will be finally assigned to the item.

The other thing you need to keep in mind is that the values returned by input
processors are collected internally (in lists) and then passed to output
processors to populate the fields.

Last, but not least, Scrapy comes with some :ref:`commonly used processors
<topics-loaders-available-processors>` built-in for convenience.


Declaring Item Loaders
======================

Item Loaders are declared like Items, by using a class definition syntax. Here
is an example::

    from scrapy.contrib.loader import ItemLoader
    from scrapy.contrib.loader.processor import TakeFirst, MapCompose, Join

    class ProductLoader(ItemLoader):

        default_output_processor = TakeFirst()

        name_in = MapCompose(unicode.title)
        name_out = Join()

        price_in = MapCompose(unicode.strip)

        # ...

As you can see, input processors are declared using the ``_in`` suffix while
output processors are declared using the ``_out`` suffix. And you can also
declare a default input/output processors using the
:attr:`ItemLoader.default_input_processor` and
:attr:`ItemLoader.default_output_processor` attributes.

.. _topics-loaders-processors-declaring:

Declaring Input and Output Processors
=====================================

As seen in the previous section, input and output processors can be declared in
the Item Loader definition, and it's very common to declare input processors
this way. However, there is one more place where you can specify the input and
output processors to use: in the :ref:`Item Field <topics-items-fields>`
metadata. Here is an example::

    import scrapy
    from scrapy.contrib.loader.processor import Join, MapCompose, TakeFirst
    from w3lib.html import remove_tags

    def filter_price(value):
        if value.isdigit():
            return value
            
    class Product(scrapy.Item):
        name = scrapy.Field(
            input_processor=MapCompose(remove_tags),
            output_processor=Join(),
        )
        price = scrapy.Field(
            input_processor=MapCompose(remove_tags, filter_price),
            output_processor=TakeFirst(),
        )
        
::

    >>> from scrapy.contrib.loader import ItemLoader
    >>> il = ItemLoader(item=Product())
    >>> il.add_value('name', [u'Welcome to my', u'<strong>website</strong>'])
    >>> il.add_value('price', [u'&euro;', u'<span>1000</span>'])
    >>> il.load_item()
    {'name': u'Welcome to my website', 'price': u'1000'}


The precedence order, for both input and output processors, is as follows:

1. Item Loader field-specific attributes: ``field_in`` and ``field_out`` (most
   precedence)
2. Field metadata (``input_processor`` and ``output_processor`` key)
3. Item Loader defaults: :meth:`ItemLoader.default_input_processor` and
   :meth:`ItemLoader.default_output_processor` (least precedence)

See also: :ref:`topics-loaders-extending`.

.. _topics-loaders-context:

Item Loader Context
===================

The Item Loader Context is a dict of arbitrary key/values which is shared among
all input and output processors in the Item Loader. It can be passed when
declaring, instantiating or using Item Loader. They are used to modify the
behaviour of the input/output processors.

For example, suppose you have a function ``parse_length`` which receives a text
value and extracts a length from it::

    def parse_length(text, loader_context):
        unit = loader_context.get('unit', 'm')
        # ... length parsing code goes here ...
        return parsed_length

By accepting a ``loader_context`` argument the function is explicitly telling
the Item Loader that it's able to receive an Item Loader context, so the Item
Loader passes the currently active context when calling it, and the processor
function (``parse_length`` in this case) can thus use them.

There are several ways to modify Item Loader context values:

1. By modifying the currently active Item Loader context
   (:attr:`~ItemLoader.context` attribute)::

      loader = ItemLoader(product)
      loader.context['unit'] = 'cm'

2. On Item Loader instantiation (the keyword arguments of Item Loader
   constructor are stored in the Item Loader context)::

      loader = ItemLoader(product, unit='cm')

3. On Item Loader declaration, for those input/output processors that support
   instantiating them with an Item Loader context. :class:`~processor.MapCompose` is one of
   them::

       class ProductLoader(ItemLoader):
           length_out = MapCompose(parse_length, unit='cm')


ItemLoader objects
==================

.. class:: ItemLoader([item, selector, response], \**kwargs)

    Return a new Item Loader for populating the given Item. If no item is
    given, one is instantiated automatically using the class in
    :attr:`default_item_class`.

    When instantiated with a `selector` or a `response` parameters
    the :class:`ItemLoader` class provides convenient mechanisms for extracting
    data from web pages using :ref:`selectors <topics-selectors>`.

    :param item: The item instance to populate using subsequent calls to
        :meth:`~ItemLoader.add_xpath`, :meth:`~ItemLoader.add_css`,
        or :meth:`~ItemLoader.add_value`.
    :type item: :class:`~scrapy.item.Item` object

    :param selector: The selector to extract data from, when using the
        :meth:`add_xpath` (resp. :meth:`add_css`) or :meth:`replace_xpath`
        (resp. :meth:`replace_css`) method.
    :type selector: :class:`~scrapy.selector.Selector` object

    :param response: The response used to construct the selector using the
        :attr:`default_selector_class`, unless the selector argument is given,
        in which case this argument is ignored.
    :type response: :class:`~scrapy.http.Response` object

    The item, selector, response and the remaining keyword arguments are
    assigned to the Loader context (accessible through the :attr:`context` attribute).

    :class:`ItemLoader` instances have the following methods:

    .. method:: get_value(value, \*processors, \**kwargs)

        Process the given ``value`` by the given ``processors`` and keyword
        arguments.

        Available keyword arguments:

        :param re: a regular expression to use for extracting data from the
            given value using :meth:`~scrapy.utils.misc.extract_regex` method,
            applied before processors
        :type re: str or compiled regex

        Examples::

            >>> from scrapy.contrib.loader.processor import TakeFirst
            >>> loader.get_value(u'name: foo', TakeFirst(), unicode.upper, re='name: (.+)')
            'FOO`

    .. method:: add_value(field_name, value, \*processors, \**kwargs)

        Process and then add the given ``value`` for the given field.

        The value is first passed through :meth:`get_value` by giving the
        ``processors`` and ``kwargs``, and then passed through the
        :ref:`field input processor <topics-loaders-processors>` and its result
        appended to the data collected for that field. If the field already
        contains collected data, the new data is added.

        The given ``field_name`` can be ``None``, in which case values for
        multiple fields may be added. And the processed value should be a dict
        with field_name mapped to values.

        Examples::

            loader.add_value('name', u'Color TV')
            loader.add_value('colours', [u'white', u'blue'])
            loader.add_value('length', u'100')
            loader.add_value('name', u'name: foo', TakeFirst(), re='name: (.+)')
            loader.add_value(None, {'name': u'foo', 'sex': u'male'})

    .. method:: replace_value(field_name, value, \*processors, \**kwargs)

        Similar to :meth:`add_value` but replaces the collected data with the
        new value instead of adding it.
    .. method:: get_xpath(xpath, \*processors, \**kwargs)

        Similar to :meth:`ItemLoader.get_value` but receives an XPath instead of a
        value, which is used to extract a list of unicode strings from the
        selector associated with this :class:`ItemLoader`.

        :param xpath: the XPath to extract data from
        :type xpath: str

        :param re: a regular expression to use for extracting data from the
            selected XPath region
        :type re: str or compiled regex

        Examples::

            # HTML snippet: <p class="product-name">Color TV</p>
            loader.get_xpath('//p[@class="product-name"]')
            # HTML snippet: <p id="price">the price is $1200</p>
            loader.get_xpath('//p[@id="price"]', TakeFirst(), re='the price is (.*)')

    .. method:: add_xpath(field_name, xpath, \*processors, \**kwargs)

        Similar to :meth:`ItemLoader.add_value` but receives an XPath instead of a
        value, which is used to extract a list of unicode strings from the
        selector associated with this :class:`ItemLoader`.

        See :meth:`get_xpath` for ``kwargs``.

        :param xpath: the XPath to extract data from
        :type xpath: str

        Examples::

            # HTML snippet: <p class="product-name">Color TV</p>
            loader.add_xpath('name', '//p[@class="product-name"]')
            # HTML snippet: <p id="price">the price is $1200</p>
            loader.add_xpath('price', '//p[@id="price"]', re='the price is (.*)')

    .. method:: replace_xpath(field_name, xpath, \*processors, \**kwargs)

        Similar to :meth:`add_xpath` but replaces collected data instead of
        adding it.

    .. method:: get_css(css, \*processors, \**kwargs)

        Similar to :meth:`ItemLoader.get_value` but receives a CSS selector
        instead of a value, which is used to extract a list of unicode strings
        from the selector associated with this :class:`ItemLoader`.

        :param css: the CSS selector to extract data from
        :type css: str

        :param re: a regular expression to use for extracting data from the
            selected CSS region
        :type re: str or compiled regex

        Examples::

            # HTML snippet: <p class="product-name">Color TV</p>
            loader.get_css('p.product-name')
            # HTML snippet: <p id="price">the price is $1200</p>
            loader.get_css('p#price', TakeFirst(), re='the price is (.*)')

    .. method:: add_css(field_name, css, \*processors, \**kwargs)

        Similar to :meth:`ItemLoader.add_value` but receives a CSS selector
        instead of a value, which is used to extract a list of unicode strings
        from the selector associated with this :class:`ItemLoader`.

        See :meth:`get_css` for ``kwargs``.

        :param css: the CSS selector to extract data from
        :type css: str

        Examples::

            # HTML snippet: <p class="product-name">Color TV</p>
            loader.add_css('name', 'p.product-name')
            # HTML snippet: <p id="price">the price is $1200</p>
            loader.add_css('price', 'p#price', re='the price is (.*)')

    .. method:: replace_css(field_name, css, \*processors, \**kwargs)

        Similar to :meth:`add_css` but replaces collected data instead of
        adding it.

    .. method:: load_item()

        Populate the item with the data collected so far, and return it. The
        data collected is first passed through the :ref:`output processors
        <topics-loaders-processors>` to get the final value to assign to each
        item field.

    .. method:: get_collected_values(field_name)

        Return the collected values for the given field.

    .. method:: get_output_value(field_name)

        Return the collected values parsed using the output processor, for the
        given field. This method doesn't populate or modify the item at all.

    .. method:: get_input_processor(field_name)

        Return the input processor for the given field.

    .. method:: get_output_processor(field_name)

        Return the output processor for the given field.

    :class:`ItemLoader` instances have the following attributes:

    .. attribute:: item

        The :class:`~scrapy.item.Item` object being parsed by this Item Loader.

    .. attribute:: context

        The currently active :ref:`Context <topics-loaders-context>` of this
        Item Loader.

    .. attribute:: default_item_class

        An Item class (or factory), used to instantiate items when not given in
        the constructor.

    .. attribute:: default_input_processor

        The default input processor to use for those fields which don't specify
        one.

    .. attribute:: default_output_processor

        The default output processor to use for those fields which don't specify
        one.

    .. attribute:: default_selector_class

        The class used to construct the :attr:`selector` of this
        :class:`ItemLoader`, if only a response is given in the constructor.
        If a selector is given in the constructor this attribute is ignored.
        This attribute is sometimes overridden in subclasses.

    .. attribute:: selector

        The :class:`~scrapy.selector.Selector` object to extract data from.
        It's either the selector given in the constructor or one created from
        the response given in the constructor using the
        :attr:`default_selector_class`. This attribute is meant to be
        read-only.

.. _topics-loaders-extending:

Reusing and extending Item Loaders
==================================

As your project grows bigger and acquires more and more spiders, maintenance
becomes a fundamental problem, especially when you have to deal with many
different parsing rules for each spider, having a lot of exceptions, but also
wanting to reuse the common processors.

Item Loaders are designed to ease the maintenance burden of parsing rules,
without losing flexibility and, at the same time, providing a convenient
mechanism for extending and overriding them. For this reason Item Loaders
support traditional Python class inheritance for dealing with differences of
specific spiders (or groups of spiders).

Suppose, for example, that some particular site encloses their product names in
three dashes (e.g. ``---Plasma TV---``) and you don't want to end up scraping
those dashes in the final product names.

Here's how you can remove those dashes by reusing and extending the default
Product Item Loader (``ProductLoader``)::

    from scrapy.contrib.loader.processor import MapCompose
    from myproject.ItemLoaders import ProductLoader

    def strip_dashes(x):
        return x.strip('-')

    class SiteSpecificLoader(ProductLoader):
        name_in = MapCompose(strip_dashes, ProductLoader.name_in)

Another case where extending Item Loaders can be very helpful is when you have
multiple source formats, for example XML and HTML. In the XML version you may
want to remove ``CDATA`` occurrences. Here's an example of how to do it::

    from scrapy.contrib.loader.processor import MapCompose
    from myproject.ItemLoaders import ProductLoader
    from myproject.utils.xml import remove_cdata

    class XmlProductLoader(ProductLoader):
        name_in = MapCompose(remove_cdata, ProductLoader.name_in)

And that's how you typically extend input processors.

As for output processors, it is more common to declare them in the field metadata,
as they usually depend only on the field and not on each specific site parsing
rule (as input processors do). See also:
:ref:`topics-loaders-processors-declaring`.

There are many other possible ways to extend, inherit and override your Item
Loaders, and different Item Loaders hierarchies may fit better for different
projects. Scrapy only provides the mechanism; it doesn't impose any specific
organization of your Loaders collection - that's up to you and your project's
needs.

.. _topics-loaders-available-processors:

Available built-in processors
=============================

.. module:: scrapy.contrib.loader.processor
   :synopsis: A collection of processors to use with Item Loaders

Even though you can use any callable function as input and output processors,
Scrapy provides some commonly used processors, which are described below. Some
of them, like the :class:`MapCompose` (which is typically used as input
processor) compose the output of several functions executed in order, to
produce the final parsed value.

Here is a list of all built-in processors:

.. class:: Identity

    The simplest processor, which doesn't do anything. It returns the original
    values unchanged. It doesn't receive any constructor arguments nor accepts
    Loader contexts.

    Example::

        >>> from scrapy.contrib.loader.processor import Identity
        >>> proc = Identity()
        >>> proc(['one', 'two', 'three'])
        ['one', 'two', 'three']

.. class:: TakeFirst

    Returns the first non-null/non-empty value from the values received,
    so it's typically used as an output processor to single-valued fields.
    It doesn't receive any constructor arguments, nor accept Loader contexts.

    Example::

        >>> from scrapy.contrib.loader.processor import TakeFirst
        >>> proc = TakeFirst()
        >>> proc(['', 'one', 'two', 'three'])
        'one'

.. class:: Join(separator=u' ')

    Returns the values joined with the separator given in the constructor, which
    defaults to ``u' '``. It doesn't accept Loader contexts.

    When using the default separator, this processor is equivalent to the
    function: ``u' '.join``

    Examples::

        >>> from scrapy.contrib.loader.processor import Join
        >>> proc = Join()
        >>> proc(['one', 'two', 'three'])
        u'one two three'
        >>> proc = Join('<br>')
        >>> proc(['one', 'two', 'three'])
        u'one<br>two<br>three'

.. class:: Compose(\*functions, \**default_loader_context)

    A processor which is constructed from the composition of the given
    functions. This means that each input value of this processor is passed to
    the first function, and the result of that function is passed to the second
    function, and so on, until the last function returns the output value of
    this processor.

    By default, stop process on ``None`` value. This behaviour can be changed by
    passing keyword argument ``stop_on_none=False``.

    Example::

        >>> from scrapy.contrib.loader.processor import Compose
        >>> proc = Compose(lambda v: v[0], str.upper)
        >>> proc(['hello', 'world'])
        'HELLO'

    Each function can optionally receive a ``loader_context`` parameter. For
    those which do, this processor will pass the currently active :ref:`Loader
    context <topics-loaders-context>` through that parameter.

    The keyword arguments passed in the constructor are used as the default
    Loader context values passed to each function call. However, the final
    Loader context values passed to functions are overridden with the currently
    active Loader context accessible through the :meth:`ItemLoader.context`
    attribute.

.. class:: MapCompose(\*functions, \**default_loader_context)

    A processor which is constructed from the composition of the given
    functions, similar to the :class:`Compose` processor. The difference with
    this processor is the way internal results are passed among functions,
    which is as follows:

    The input value of this processor is *iterated* and the first function is
    applied to each element. The results of these function calls (one for each element)
    are concatenated to construct a new iterable, which is then used to apply the
    second function, and so on, until the last function is applied to each
    value of the list of values collected so far. The output values of the last
    function are concatenated together to produce the output of this processor.

    Each particular function can return a value or a list of values, which is
    flattened with the list of values returned by the same function applied to
    the other input values. The functions can also return ``None`` in which
    case the output of that function is ignored for further processing over the
    chain.

    This processor provides a convenient way to compose functions that only
    work with single values (instead of iterables). For this reason the
    :class:`MapCompose` processor is typically used as input processor, since
    data is often extracted using the
    :meth:`~scrapy.selector.Selector.extract` method of :ref:`selectors
    <topics-selectors>`, which returns a list of unicode strings.

    The example below should clarify how it works::

        >>> def filter_world(x):
        ...     return None if x == 'world' else x
        ...
        >>> from scrapy.contrib.loader.processor import MapCompose
        >>> proc = MapCompose(filter_world, unicode.upper)
        >>> proc([u'hello', u'world', u'this', u'is', u'scrapy'])
        [u'HELLO, u'THIS', u'IS', u'SCRAPY']

    As with the Compose processor, functions can receive Loader contexts, and
    constructor keyword arguments are used as default context values. See
    :class:`Compose` processor for more info.

