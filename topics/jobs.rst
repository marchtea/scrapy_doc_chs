
.. _topics-jobs:

=================================
Jobs: 暂停，恢复爬虫
=================================

在抓大站的一些情况下，我们希望能够临时暂停爬取并在不久后还能恢复。

Scrapy通过如下工具支持这个功能:

* 一个把调度请求保存在磁盘的调度器

* 一个把访问请求保存在磁盘的副本过滤器[duplicates filter]

* 一个能持续保持爬虫状态(键/值对)的扩展

Job 路径
=============

要启用持久化支持，你只需要通过 ``JOBDIR`` 设置 *job directory* 选项。这个路径将要存储
所有的请求数据来保持一个单独任务的状态(以一个爬虫工作为例)。值得注意的是，这个目录不是必须对不同的爬
虫共享，甚至同一个爬虫的不同jobs/runs也不必共享，也就是这个目录就是存储一个 *单独的* job的状态信
息。

怎么使用
=============

运行以下的命令，启用一个爬虫的持久化::

    scrapy crawl somespider -s JOBDIR=crawls/somespider-1

然后，你就能在任何时候安全的停止爬虫(按Ctrl-C或者发送一个信号)，而恢复这个爬虫是同样的命令::

    scrapy crawl somespider -s JOBDIR=crawls/somespider-1

保持状态
========================================

有的时候，你希望持续保持一些运行长时间的蜘蛛的状态。这时你用 ``spider.state`` 属性实现,
这个属性必须是字典类型. 当爬虫开始或者结束的时候，有个内置扩展负责从job路径序列化,存储,
加载那个属性。

下面这个例子展示了如何让爬虫有状态（这里省略了其他爬虫的代码）的回调::

    def parse_item(self, response):
        # parse item here
        self.state['items_count'] = self.state.get('items_count', 0) + 1

持久化的一些坑
===================

如果你想要使用Scrapy的状态保持还有一些东西你需要知道:
持久化支持：

Cookies的有效期
------------------

Cookies是有有效期的(可能过期)。所以如果你没有把你的爬虫及时恢复，那么他可能在被调度回去的时候
就不能工作了。当然如果你的爬虫不依赖cookies就不会有这个问题了。

请求序列化
---------------------

请求必须用 `pickle` 模块序列化，所以你需要确保你的请求都是可被序列化的。
这里最常见的问题是在请求无法持久化的回调函数中使用 ``lambda`` 方法

例如, 下面这样就是不正确的::

    def some_callback(self, response):
        somearg = 'test'
        return Request('http://www.example.com', callback=lambda r: self.other_callback(r, somearg))

    def other_callback(self, response, somearg):
        print "the argument passed is:", somearg

这样才对::

    def some_callback(self, response):
        somearg = 'test'
        return Request('http://www.example.com', meta={'somearg': somearg})

    def other_callback(self, response):
        somearg = response.meta['somearg']
        print "the argument passed is:", somearg

.. _pickle: http://docs.python.org/library/pickle.html
