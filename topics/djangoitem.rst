.. _topics-djangoitem:

==========
DjangoItem
==========

:class:`DjangoItem` 是一个item的类，其从Django模型中获取字段(field)定义。
您可以简单地创建一个 :class:`DjangoItem` 并指定其关联的Django模型。

除了获得您item中定义的字段外， :class:`DjangoItem`
提供了创建并获得一个具有item数据的Django模型实例(Django model instance)的方法。

使用DjangoItem
================

:class:`DjangoItem` 使用方法与Django中的ModelForms类似。您创建一个子类，
并定义其 ``django_model`` 属性。这样，您就可以得到一个字段与Django模型字段(model field)一一对应的item了。

另外，您可以定义模型中没有的字段，甚至是覆盖模型中已经定义的字段。

让我们来看个例子:

创造一个Django模型::
   
   from django.db import models

   class Person(models.Model):
       name = models.CharField(max_length=255)
       age = models.IntegerField()

定义一个基本的 :class:`DjangoItem`::

   from scrapy.contrib.djangoitem import DjangoItem

   class PersonItem(DjangoItem):
       django_model = Person

       
:class:`DjangoItem` 的使用方法和 :class:`~scrapy.item.Item` 类似::

   >>> p = PersonItem()
   >>> p['name'] = 'John'
   >>> p['age'] = '22'

要从item中获取Django模型，调用 :class:`DjangoItem` 中额外的方法 :meth:`~DjangoItem.save`::

   >>> person = p.save()
   >>> person.name
   'John'
   >>> person.age
   '22'
   >>> person.id
   1

当我们调用 :meth:`~DjangoItem.save` 时，模型已经保存了。我们可以在调用时带上 ``commit=False`` 来避免保存，
并获取到一个未保存的模型::

   >>> person = p.save(commit=False)
   >>> person.name
   'John'
   >>> person.age
   '22'
   >>> person.id
   None

正如之前所说的，我们可以在item中加入字段::

   import scrapy
   from scrapy.contrib.djangoitem import DjangoItem

   class PersonItem(DjangoItem):
       django_model = Person
       sex = scrapy.Field()

::

   >>> p = PersonItem()
   >>> p['name'] = 'John'
   >>> p['age'] = '22'
   >>> p['sex'] = 'M'

.. note:: 当执行 :meth:`~DjangoItem.save` 时添加到item的字段不会有作用(taken into account)。

并且我们可以覆盖模型中的字段::

   class PersonItem(DjangoItem):
       django_model = Person
       name = scrapy.Field(default='No Name')

这在提供字段属性时十分有用，例如您项目中使用的默认或者其他属性一样。

DjangoItem注意事项
==================

DjangoItem提供了在Scrapy项目中集成DjangoItem的简便方法，不过需要注意的是，
如果在Scrapy中爬取大量(百万级)的item时，Django ORM扩展得并不是很好(not scale well)。
这是因为关系型后端对于一个密集型(intensive)应用(例如web爬虫)并不是一个很好的选择，
尤其是具有大量的索引的数据库。

配置Django的设置
======================

在Django应用之外使用Django模型(model)，您需要设置
``DJANGO_SETTINGS_MODULE`` 环境变量以及 --大多数情况下-- 修改
``PYTHONPATH`` 环境变量来导入设置模块。

完成这个配置有很多方法，具体选择取决您的情况及偏好。
下面详细给出了完成这个配置的最简单方法。

假设您项目的名称为 ``mysite`` ，位于
``/home/projects/mysite`` 且用 ``Person`` 模型创建了一个应用 ``myapp`` 。
这意味着您的目录结构类似于::

    /home/projects/mysite
    ├── manage.py
    ├── myapp
    │   ├── __init__.py
    │   ├── models.py
    │   ├── tests.py
    │   └── views.py
    └── mysite
        ├── __init__.py
        ├── settings.py
        ├── urls.py
        └── wsgi.py

接着您需要将 ``/home/projects/mysite`` 加入到 ``PYTHONPATH``
环境变量中并将 ``mysite.settings`` 设置为 ``DJANGO_SETTINGS_MODULE`` 环境变量。
这可以在Scrapy设置文件中添加下列代码::

  import sys
  sys.path.append('/home/projects/mysite')

  import os
  os.environ['DJANGO_SETTINGS_MODULE'] = 'mysite.settings'

注意，由于我们在python运行环境中，所以我们修改 ``sys.path`` 变量而不是 ``PYTHONPATH`` 环境变量。
如果所有设置正确，您应该可以运行 ``scrapy shell`` 命令并且导入 ``Person`` 模型(例如 ``from myapp.models import Person``)。
