
.. _topics-jobs:

=================================
Jobs: 暂停，恢复爬虫
=================================

有些情况下，例如爬取大的站点，我们希望能暂停爬取，之后再恢复运行。

Scrapy通过如下工具支持这个功能:

* 一个把调度请求保存在磁盘的调度器

* 一个把访问请求保存在磁盘的副本过滤器[duplicates filter]

* 一个能持续保持爬虫状态(键/值对)的扩展

Job 路径
=============

要启用持久化支持，你只需要通过 ``JOBDIR`` 设置 *job directory* 选项。这个路径将会存储
所有的请求数据来保持一个单独任务的状态(例如：一次spider爬取(a spider run))。必须要注意的是，这个目录不允许被不同的spider
共享，甚至是同一个spider的不同jobs/runs也不行。也就是说，这个目录就是存储一个 *单独* job的状态信息。

怎么使用
=============

要启用一个爬虫的持久化，运行以下命令::

    scrapy crawl somespider -s JOBDIR=crawls/somespider-1

然后，你就能在任何时候安全地停止爬虫(按Ctrl-C或者发送一个信号)。恢复这个爬虫也是同样的命令::

    scrapy crawl somespider -s JOBDIR=crawls/somespider-1

保持状态
========================================

有的时候，你希望持续保持一些运行长时间的蜘蛛的状态。这时您可以使用 ``spider.state`` 属性,
该属性的类型必须是dict. scrapy提供了内置扩展负责在spider启动或结束时，从工作路径(job directory)中序列化、存储、加载属性。

下面这个例子展示了使用spider state的回调函数(callback)(简洁起见，省略了其他的代码)::

    def parse_item(self, response):
        # parse item here
        self.state['items_count'] = self.state.get('items_count', 0) + 1

持久化的一些坑
===================

如果你想要使用Scrapy的持久化支持,还有一些东西您需要了解:

Cookies的有效期
------------------

Cookies是有有效期的(可能过期)。所以如果你没有把你的爬虫及时恢复，那么他可能在被调度回去的时候
就不能工作了。当然如果你的爬虫不依赖cookies就不会有这个问题了。

请求序列化
---------------------

请求是由 `pickle` 进行序列化的，所以你需要确保你的请求是可被pickle序列化的。
这里最常见的问题是在在request回调函数中使用 ``lambda`` 方法，导致无法序列化。

例如, 这样就会有问题::

    def some_callback(self, response):
        somearg = 'test'
        return scrapy.Request('http://www.example.com', callback=lambda r: self.other_callback(r, somearg))

    def other_callback(self, response, somearg):
        print "the argument passed is:", somearg

这样才对::

    def some_callback(self, response):
        somearg = 'test'
        return scrapy.Request('http://www.example.com', meta={'somearg': somearg})

    #这里的实例代码有错，应该是(译者注)
    #   return scrapy.Request('http://www.example.com', meta={'somearg': somearg}, callback=self.other_callback)

    def other_callback(self, response):
        somearg = response.meta['somearg']
        print "the argument passed is:", somearg

.. _pickle: http://docs.python.org/library/pickle.html
