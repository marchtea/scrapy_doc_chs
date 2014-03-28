.. _topics-email:

==============
发送email
==============

.. module:: scrapy.mail
   :synopsis: Email sending facility

虽然Python通过 `smtplib`_ 库使得发送email变得很简单，Scrapy仍然提供了自己的实现。
该功能十分易用，同时由于采用了 `Twisted非阻塞式(non-blocking)IO`_ ，其避免了对爬虫的非阻塞式IO的影响。
另外，其也提供了简单的API来发送附件。
通过一些 :ref:`settings <topics-email-settings>` 设置，您可以很简单的进行配置。

.. _smtplib: http://docs.python.org/library/smtplib.html
.. _Twisted非阻塞式(non-blocking)IO: http://twistedmatrix.com/projects/core/documentation/howto/async.html

简单例子
=============

有两种方法可以创建邮件发送器(mail sender)。
您可以通过标准构造器(constructor)创建::

    from scrapy.mail import MailSender
    mailer = MailSender()

或者您可以传递一个Scrapy设置对象，其会参考 
:ref:`settings <topics-email-settings>`::

    mailer = MailSender.from_settings(settings)

这是如何来发送邮件了(不包括附件)::

    mailer.send(to=["someone@example.com"], subject="Some subject", body="Some body", cc=["another@example.com"])

MailSender类参考手册
==========================

在Scrapy中发送email推荐使用MailSender。其同框架中其他的部分一样，使用了
`Twisted非阻塞式(non-blocking)IO`_ 。

.. class:: MailSender(smtphost=None, mailfrom=None, smtpuser=None, smtppass=None, smtpport=None)

    :param smtphost: 发送email的SMTP主机(host)。如果忽略，则使用 :setting:`MAIL_HOST` 。
    :type smtphost: str

    :param mailfrom: 用于发送email的地址(address)(填入 ``From:``) 。
      如果忽略，则使用 :setting:`MAIL_FROM` 。
    :type mailfrom: str

    :param smtpuser: SMTP用户。如果忽略,则使用 :setting:`MAIL_USER` 。
      如果未给定，则将不会进行SMTP认证(authentication)。
    :type smtphost: str

    :param smtppass: SMTP认证的密码
    :type smtppass: str

    :param smtpport: SMTP连接的短裤 
    :type smtpport: int

    :param smtptls: 强制使用STARTTLS
    :type smtpport: boolean

    :param smtpssl: 强制使用SSL连接 
    :type smtpport: boolean

    .. classmethod:: from_settings(settings)

        使用Scrapy设置对象来初始化对象。其会参考
        :ref:`这些Scrapy设置 <topics-email-settings>`.

        :param settings: the e-mail recipients
        :type settings: :class:`scrapy.settings.Settings` object

    .. method:: send(to, subject, body, cc=None, attachs=(), mimetype='text/plain')

        发送email到给定的接收者。

        :param to: email接收者
        :type to: list

        :param subject: email内容
        :type subject: str

        :param cc: 抄送的人
        :type cc: list

        :param body: email的内容
        :type body: str

        :param attachs: 可迭代的元组 ``(attach_name, mimetype, file_object)``。
              ``attach_name`` 是一个在email的附件中显示的名字的字符串，
              ``mimetype`` 是附件的mime类型，
              ``file_object`` 是包含附件内容的可读的文件对象。
        :type attachs: iterable

        :param mimetype: email的mime类型
        :type mimetype: str


.. _topics-email-settings:

Mail设置
=============

这些设置定义了
:class:`MailSender` 构造器的默认值。其使得在您不编写任何一行代码的情况下，为您的项目配置实现email通知的功能。

.. setting:: MAIL_FROM

MAIL_FROM
---------

默认值: ``'scrapy@localhost'``

用于发送email的地址(address)(填入 ``From:``) 。

.. setting:: MAIL_HOST

MAIL_HOST
---------

默认值: ``'localhost'``

发送email的SMTP主机(host)。

.. setting:: MAIL_PORT

MAIL_PORT
---------

默认值: ``25``

发用邮件的SMTP端口。

.. setting:: MAIL_USER

MAIL_USER
---------

默认值: ``None``

SMTP用户。如果未给定，则将不会进行SMTP认证(authentication)。

.. setting:: MAIL_PASS

MAIL_PASS
---------

默认值: ``None``

用于SMTP认证，与 :setting:`MAIL_USER` 配套的密码。

.. setting:: MAIL_TLS

MAIL_TLS
---------

默认值: ``False``

强制使用STARTTLS。STARTTLS能使得在已经存在的不安全连接上，通过使用SSL/TLS来实现安全连接。

.. setting:: MAIL_SSL

MAIL_SSL
---------

默认值: ``False``

强制使用SSL加密连接。
