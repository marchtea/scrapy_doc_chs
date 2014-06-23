.. _topics-exporters:

==============
Item Exporters
==============

.. module:: scrapy.contrib.exporter
   :synopsis: Item Exporters

当你抓取了你要的数据(Items)，你就会想要将他们持久化或导出它们，并应用在其他的程序。这是整个抓取过程的目的。

为此，Scrapy提供了Item Exporters 来创建不同的输出格式，如XML，CSV或JSON。

使用 Item Exporter
====================

如果你很忙，只想使用 Item Exporter 输出数据，请查看 :ref:`topics-feed-exports`. 相反，如果你想知道Item Exporter 是如何工作的，或需要更多的自定义功能（不包括默认的 exports），请继续阅读下文。

为了使用 Item Exporter，你必须对 Item Exporter 及其参数 (args) 实例化。每个 Item Exporter 需要不同的参数，详细请查看 :ref:`topics-exporters-reference` 。在实例化了 exporter 之后，你必须：

1. 调用方法 :meth:`~BaseItemExporter.start_exporting` 以标识 exporting 过程的开始。

2. 对要导出的每个项目调用 :meth:`~BaseItemExporter.export_item` 方法。

3. 最后调用 :meth:`~BaseItemExporter.finish_exporting` 表示 exporting 过程的结束

这里，你可以看到一个 :doc:`Item Pipeline <item-pipeline>` ，它使用 Item Exporter 导出 items 到不同的文件，每个 spider 一个::

   from scrapy import signals
   from scrapy.contrib.exporter import XmlItemExporter

   class XmlExportPipeline(object):

       def __init__(self):
           self.files = {}

        @classmethod
        def from_crawler(cls, crawler):
            pipeline = cls()
            crawler.signals.connect(pipeline.spider_opened, signals.spider_opened)
            crawler.signals.connect(pipeline.spider_closed, signals.spider_closed)
            return pipeline

       def spider_opened(self, spider):
           file = open('%s_products.xml' % spider.name, 'w+b')
           self.files[spider] = file
           self.exporter = XmlItemExporter(file)
           self.exporter.start_exporting()

       def spider_closed(self, spider):
           self.exporter.finish_exporting()
           file = self.files.pop(spider)
           file.close()

       def process_item(self, item, spider):
           self.exporter.export_item(item)
           return item


.. _topics-exporters-field-serialization:

序列化 item fields

============================

B默认情况下，该字段值将不变的传递到序列化库，如何对其进行序列化的决定被委托给每一个特定的序列化库。

但是，你可以自定义每个字段值如何序列化在它被传递到序列化库中之前。

有两种方法可以自定义一个字段如何被序列化，请看下文。

.. _topics-exporters-serializers:

1. 在 field 类中声明一个 serializer
--------------------------------------

您可以在 :ref:`field metadata <topics-items-fields>` 声明一个 serializer。该 serializer 必须可调用，并返回它的序列化形式。


实例::

      from scrapy.item import Item, Field

      def serialize_price(value):
         return '$ %s' % str(value)

      class Product(Item):
          name = Field()
          price = Field(serializer=serialize_price)

2. 覆盖(overriding) serialize_field() 方法
------------------------------------------

你可以覆盖 :meth:`~BaseItemExporter.serialize_field()` 方法来自定义如何输出你的数据。

在你的自定义代码后确保你调用父类的 :meth:`~BaseItemExporter.serialize_field()` 方法。

实例::

      from scrapy.contrib.exporter import XmlItemExporter

      class ProductXmlExporter(XmlItemExporter):

          def serialize_field(self, field, name, value):
              if field == 'price':
                  return '$ %s' % str(value)
              return super(Product, self).serialize_field(field, name, value)

.. _topics-exporters-reference:

Item Exporters 参考资料
=================================

下面是一些Scrapy内置的 Item Exporters类. 其中一些包括了实例, 假设你要输出以下2个Items::

    Item(name='Color TV', price='1200')
    Item(name='DVD player', price='200')

BaseItemExporter
----------------

.. class:: BaseItemExporter(fields_to_export=None, export_empty_fields=False, encoding='utf-8')

   这是一个对所有 Item Exporters 的(抽象)父类。它对所有(具体) Item Exporters 提供基本属性，如定义export什么fields, 是否export空fields, 或是否进行编码。

   你可以在构造器中设置它们不同的属性值: :attr:`fields_to_export` ,
   :attr:`export_empty_fields`, :attr:`encoding`.

   .. method:: export_item(item)

      输出给定item. 此方法必须在子类中实现.

   .. method:: serialize_field(field, name, value)

      返回给定field的序列化值. 你可以覆盖此方法来控制序列化或输出指定的field.

      默认情况下, 此方法寻找一个 serializer :ref:`在 item
      field 中声明 <topics-exporters-serializers>` 并返回它的值. 如果没有发现   serializer, 则值不会改变，除非你使用 ``unicode`` 值并编码到
      ``str``， 编码可以在 :attr:`encoding` 属性中声明.

      :param field: the field being serialized
      :type field: :class:`~scrapy.item.Field` object

      :param name: the name of the field being serialized
      :type name: str

      :param value: the value being serialized

   .. method:: start_exporting()

      表示exporting过程的开始. 一些exporters用于产生需要的头元素(例如
      :class:`XmlItemExporter`). 在实现exporting item前必须调用此方法.

   .. method:: finish_exporting()

      表示exporting过程的结束. 一些exporters用于产生需要的尾元素 (例如
      :class:`XmlItemExporter`). 在完成exporting item后必须调用此方法.

   .. attribute:: fields_to_export

      列出export什么fields值, None表示export所有fields. 默认值为None.

      一些 exporters (例如 :class:`CsvItemExporter`) 按照定义在属性中fields的次序依次输出.

   .. attribute:: export_empty_fields

      是否在输出数据中包含为空的item fields.
      默认值是 ``False``. 一些 exporters (例如 :class:`CsvItemExporter`)
      会忽略此属性并输出所有fields.

   .. attribute:: encoding

      Encoding 属性将用于编码 unicode 值. (仅用于序列化字符串).其他值类型将不变的传递到指定的序列化库.

.. highlight:: none

XmlItemExporter
---------------

.. class:: XmlItemExporter(file, item_element='item', root_element='items', \**kwargs)

   以XML格式 exports Items 到指定的文件类.

   :param file: 文件类.

   :param root_element: XML 根元素名.
   :type root_element: str

   :param item_element: XML item 的元素名.
   :type item_element: str

   构造器额外的关键字参数将传给 :class:`BaseItemExporter` 构造器.

   一个典型的 exporter 实例::

       <?xml version="1.0" encoding="utf-8"?>
       <items>
         <item>
           <name>Color TV</name>
           <price>1200</price>
        </item>
         <item>
           <name>DVD player</name>
           <price>200</price>
        </item>
       </items>

   除了覆盖 :meth:`serialize_field` 方法, 多个值的 fields 会转化每个值到 ``<value>`` 元素.

   例如, item::

        Item(name=['John', 'Doe'], age='23')

   将被转化为::

       <?xml version="1.0" encoding="utf-8"?>
       <items>
         <item>
           <name>
             <value>John</value>
             <value>Doe</value>
           </name>
           <age>23</age>
         </item>
       </items>

CsvItemExporter
---------------

.. class:: CsvItemExporter(file, include_headers_line=True, join_multivalued=',', \**kwargs)

   输出 csv 文件格式. 如果添加 :attr:`fields_to_export` 属性, 它会按顺序定义CSV的列名. :attr:`export_empty_fields` 属性在此没有作用.

   :param file: 文件类.

   :param include_headers_line: 启用后 exporter 会输出第一行为列名, 列名从 :attr:`BaseItemExporter.fields_to_export` 或第一个 item fields 获取.
   :type include_headers_line: boolean

   :param join_multivalued: char 将用于连接多个值的fields.
   :type include_headers_line: str

   此构造器额外的关键字参数将传给 :class:`BaseItemExporter` 构造器 , 其余的将传给 `csv.writer`_ 构造器, 以此来定制 exporter.

   一个典型的 exporter 实例::

      product,price
      Color TV,1200
      DVD player,200

.. _csv.writer: http://docs.python.org/library/csv.html#csv.writer

PickleItemExporter
------------------

.. class:: PickleItemExporter(file, protocol=0, \**kwargs)

   输出 pickle 文件格式.

   :param file: 文件类.

   :param protocol: pickle 协议.
   :type protocol: int

   更多信息请看 `pickle module documentation`_.

   此构造器额外的关键字参数将传给 :class:`BaseItemExporter` 构造器.

   Pickle 不是可读的格式，这里不提供实例.

.. _pickle module documentation: http://docs.python.org/library/pickle.html

PprintItemExporter
------------------

.. class:: PprintItemExporter(file, \**kwargs)

   输出整齐打印的文件格式.

   :param file: 文件类.

   此构造器额外的关键字参数将传给 :class:`BaseItemExporter` 构造器.

   一个典型的 exporter 实例::

        {'name': 'Color TV', 'price': '1200'}
        {'name': 'DVD player', 'price': '200'}

   此格式会根据行的长短进行调整.

JsonItemExporter
----------------

.. class:: JsonItemExporter(file, \**kwargs)

   输出 JSON 文件格式, 所有对象将写进一个对象的列表. 此构造器额外的关键字参数将传给 :class:`BaseItemExporter` 构造器, 其余的将传给 `JSONEncoder`_ 构造器, 以此来定制 exporter.

   :param file: 文件类.

   一个典型的 exporter 实例::

        [{"name": "Color TV", "price": "1200"},
        {"name": "DVD player", "price": "200"}]

   .. _json-with-large-data:

   .. warning:: JSON 是一个简单而有弹性的格式, 但对大量数据的扩展性不是很好，因为这里会将整个对象放入内存. 如果你要JSON既强大又简单,可以考虑 :class:`JsonLinesItemExporter` , 或把输出对象分为多个块.

.. _JSONEncoder: http://docs.python.org/library/json.html#json.JSONEncoder

JsonLinesItemExporter
---------------------

.. class:: JsonLinesItemExporter(file, \**kwargs)

   输出 JSON 文件格式, 每行写一个 JSON-encoded 项. 此构造器额外的关键字参数将传给 :class:`BaseItemExporter` 构造器, 其余的将传给 `JSONEncoder`_ 构造器, 以此来定制 exporter.

   :param file: 文件类.

   一个典型的 exporter 实例::

        {"name": "Color TV", "price": "1200"}
        {"name": "DVD player", "price": "200"}

   这个类能很好的处理大量数据. 

.. _JSONEncoder: http://docs.python.org/library/json.html#json.JSONEncoder
