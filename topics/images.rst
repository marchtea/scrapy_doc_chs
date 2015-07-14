.. _topics-images:

=======================
下载项目图片
=======================

.. currentmodule:: scrapy.contrib.pipeline.images

Scrapy提供了一个 :doc:`item pipeline </topics/item-pipeline>` ，来下载属于某个特定项目的图片，比如，当你抓取产品时，也想把它们的图片下载到本地。

这条管道，被称作图片管道，在 :class:`ImagesPipeline` 类中实现，提供了一个方便并具有额外特性的方法，来下载并本地存储图片:

* 将所有下载的图片转换成通用的格式（JPG）和模式（RGB）
* 避免重新下载最近已经下载过的图片
* 缩略图生成
* 检测图像的宽/高，确保它们满足最小限制

这个管道也会为那些当前安排好要下载的图片保留一个内部队列，并将那些到达的包含相同图片的项目连接到那个队列中。
这可以避免多次下载几个项目共享的同一个图片。

`Pillow`_ 是用来生成缩略图，并将图片归一化为JPEG/RGB格式，因此为了使用图片管道，你需要安装这个库。
`Python Imaging Library`_ (PIL) 在大多数情况下是有效的，但众所周知，在一些设置里会出现问题，因此我们推荐使用 `Pillow`_ 而不是PIL.

.. _Pillow: https://github.com/python-imaging/Pillow
.. _Python Imaging Library: http://www.pythonware.com/products/pil/

使用图片管道
=========================

当使用 :class:`ImagesPipeline` ，典型的工作流程如下所示:

1. 在一个爬虫里，你抓取一个项目，把其中图片的URL放入 ``image_urls`` 组内。

2. 项目从爬虫内返回，进入项目管道。

3. 当项目进入 :class:`ImagesPipeline`，``image_urls`` 组内的URLs将被Scrapy的调度器和下载器（这意味着调度器和下载器的中间件可以复用）安排下载，当优先级更高，会在其他页面被抓取前处理。项目会在这个特定的管道阶段保持“locker”的状态，直到完成图片的下载（或者由于某些原因未完成下载）。

4. 当图片下载完，另一个组(``images``)将被更新到结构中。这个组将包含一个字典列表，其中包括下载图片的信息，比如下载路径、源抓取地址（从 ``image_urls`` 组获得）和图片的校验码。
   ``images`` 列表中的图片顺序将和源 ``image_urls`` 组保持一致。如果某个图片下载失败，将会记录下错误信息，图片也不会出现在 ``images`` 组中。


使用样例
=============

为了使用图片管道，你仅需要 :ref:`启用<topics-images-enabling>` .

接着，如果spider返回一个具有 'image_urls' 键的dict，则pipeline会提取相对应的结果。

如果你更喜欢使用 :class:`~.Item` 来自定义item， 则需要设置 ``image_urls`` 和 ``images`` 字段::

    import scrapy

    class MyItem(scrapy.Item):

        # ... other item fields ...
        image_urls = scrapy.Field()
        images = scrapy.Field()

如果你需要更加复杂的功能，想重写定制图片管道行为，参见 :ref:`topics-images-override` 。

.. _topics-images-enabling:

开启你的图片管道
=============================

.. setting:: IMAGES_STORE

为了开启你的图片管道，你首先需要在项目中添加它 :setting:`ITEM_PIPELINES` setting::

    ITEM_PIPELINES = {'scrapy.contrib.pipeline.images.ImagesPipeline': 1}

并将 :setting:`IMAGES_STORE` 设置为一个有效的文件夹，用来存储下载的图片。否则管道将保持禁用状态，即使你在 :setting:`ITEM_PIPELINES` 设置中添加了它。

比如::

   IMAGES_STORE = '/path/to/valid/dir'

图片存储
==============

文件系统是当前官方唯一支持的存储系统，但也支持（非公开的） `Amazon S3`_ 。

.. _Amazon S3: https://s3.amazonaws.com/

文件系统存储
-------------------

图片存储在文件中（一个图片一个文件），并使用它们URL的 `SHA1 hash`_ 作为文件名。

比如，对下面的图片URL::

    http://www.example.com/image.jpg

它的 `SHA1 hash` 值为::

    3afec3b4765f8f0a07b78f98c07b83f013567a0a

将被下载并存为下面的文件::

   <IMAGES_STORE>/full/3afec3b4765f8f0a07b78f98c07b83f013567a0a.jpg

其中:

* ``<IMAGES_STORE>`` 是定义在 :setting:`IMAGES_STORE` 设置里的文件夹

* ``full`` 是用来区分图片和缩略图（如果使用的话）的一个子文件夹。详情参见 :ref:`topics-images-thumbnails`.

额外的特性
===================

图片失效
----------------

.. setting:: IMAGES_EXPIRES

图像管道避免下载最近已经下载的图片。使用 :setting:`IMAGES_EXPIRES` 设置可以调整失效期限，可以用天数来指定::

    # 90天的图片失效期限
    IMAGES_EXPIRES = 90

.. _topics-images-thumbnails:

缩略图生成
--------------------

图片管道可以自动创建下载图片的缩略图。

.. setting:: IMAGES_THUMBS

为了使用这个特性，你需要设置 :setting:`IMAGES_THUMBS` 字典，其关键字为缩略图名字，值为它们的大小尺寸。

比如::

   IMAGES_THUMBS = {
       'small': (50, 50),
       'big': (270, 270),
   }

当你使用这个特性时，图片管道将使用下面的格式来创建各个特定尺寸的缩略图::

    <IMAGES_STORE>/thumbs/<size_name>/<image_id>.jpg

其中:

* ``<size_name>`` 是 :setting:`IMAGES_THUMBS` 字典关键字（``small``， ``big`` ，等）

* ``<image_id>`` 是图像url的 `SHA1 hash`_ 

.. _SHA1 hash: http://en.wikipedia.org/wiki/SHA_hash_functions

例如使用 ``small`` 和 ``big`` 缩略图名字的图片文件::

   <IMAGES_STORE>/full/63bbfea82b8880ed33cdb762aa11fab722a90a24.jpg
   <IMAGES_STORE>/thumbs/small/63bbfea82b8880ed33cdb762aa11fab722a90a24.jpg
   <IMAGES_STORE>/thumbs/big/63bbfea82b8880ed33cdb762aa11fab722a90a24.jpg

第一个是从网站下载的完整图片。

滤出小图片
--------------------------

.. setting:: IMAGES_MIN_HEIGHT

.. setting:: IMAGES_MIN_WIDTH

你可以丢掉那些过小的图片，只需在:setting:`IMAGES_MIN_HEIGHT` 和 :setting:`IMAGES_MIN_WIDTH` 设置中指定最小允许的尺寸。

比如::

   IMAGES_MIN_HEIGHT = 110
   IMAGES_MIN_WIDTH = 110

注意：这些尺寸一点也不影响缩略图的生成。

默认情况下，没有尺寸限制，因此所有图片都将处理。

.. _topics-images-override:

实现定制图片管道
========================================

.. module:: scrapy.contrib.pipeline.images
   :synopsis: Images Pipeline

下面是你可以在定制的图片管道里重写的方法：

.. class:: ImagesPipeline

   .. method:: get_media_requests(item, info)

      在工作流程中可以看到，管道会得到图片的URL并从项目中下载。为了这么做，你需要重写 :meth:`~get_media_requests` 方法，并对各个图片URL返回一个Request::

         def get_media_requests(self, item, info):
             for image_url in item['image_urls']:
                 yield scrapy.Request(image_url)

      这些请求将被管道处理，当它们完成下载后，结果将以2-元素的元组列表形式传送到 :meth:`~item_completed` 方法:

      * ``success`` 是一个布尔值，当图片成功下载时为 ``True`` ，因为某个原因下载失败为``False`` 

      * ``image_info_or_error`` 是一个包含下列关键字的字典（如果成功为 ``True`` ）或者出问题时为 `Twisted Failure`_ 。

        * ``url`` - 图片下载的url。这是从 :meth:`~get_media_requests` 方法返回请求的url。

        * ``path`` - 图片存储的路径（类似 :setting:`IMAGES_STORE`）

        * ``checksum`` - 图片内容的 `MD5 hash`_ 

      :meth:`~item_completed` 接收的元组列表需要保证与 :meth:`~get_media_requests` 方法返回请求的顺序相一致。下面是 ``results`` 参数的一个典型值::

          [(True,
            {'checksum': '2b00042f7481c7b056c4b410d28f33cf',
             'path': 'full/7d97e98f8af710c7e7fe703abc8f639e0ee507c4.jpg',
             'url': 'http://www.example.com/images/product1.jpg'}),
           (True,
            {'checksum': 'b9628c4ab9b595f72f280b90c4fd093d',
             'path': 'full/1ca5879492b8fd606df1964ea3c1e2f4520f076f.jpg',
             'url': 'http://www.example.com/images/product2.jpg'}),
           (False,
            Failure(...))]

      默认 :meth:`get_media_requests` 方法返回 ``None`` ，这意味着项目中没有图片可下载。

   .. method:: item_completed(results, items, info)

      当一个单独项目中的所有图片请求完成时（要么完成下载，要么因为某种原因下载失败）， :meth:`ImagesPipeline.item_completed` 方法将被调用。

       :meth:`~item_completed` 方法需要返回一个输出，其将被送到随后的项目管道阶段，因此你需要返回（或者丢弃）项目，如你在任意管道里所做的一样。

      这里是一个 :meth:`~item_completed` 方法的例子，其中我们将下载的图片路径（传入到results中）存储到 ``image_paths`` 项目组中，如果其中没有图片，我们将丢弃项目::
      
          from scrapy.exceptions import DropItem

          def item_completed(self, results, item, info):
              image_paths = [x['path'] for ok, x in results if ok]
              if not image_paths:
                  raise DropItem("Item contains no images")
              item['image_paths'] = image_paths
              return item

      默认情况下， :meth:`item_completed` 方法返回项目。


定制图片管道的例子
==============================

下面是一个图片管道的完整例子，其方法如上所示::

    import scrapy
    from scrapy.contrib.pipeline.images import ImagesPipeline
    from scrapy.exceptions import DropItem

    class MyImagesPipeline(ImagesPipeline):

        def get_media_requests(self, item, info):
            for image_url in item['image_urls']:
                yield scrapy.Request(image_url)

        def item_completed(self, results, item, info):
            image_paths = [x['path'] for ok, x in results if ok]
            if not image_paths:
                raise DropItem("Item contains no images")
            item['image_paths'] = image_paths
            return item

.. _Twisted Failure: http://twistedmatrix.com/documents/current/api/twisted.python.failure.Failure.html
.. _MD5 hash: http://en.wikipedia.org/wiki/MD5
