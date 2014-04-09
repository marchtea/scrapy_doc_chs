.. _benchmarking:

============
Benchmarking
============

.. versionadded:: 0.17

Scrapy提供了一个简单的性能测试工具。其创建了一个本地HTTP服务器，并以最大可能的速度进行爬取。
该测试性能工具目的是测试Scrapy在您的硬件上的效率，来获得一个基本的底线用于对比。
其使用了一个简单的spider，仅跟进链接，不做任何处理。

运行::

    scrapy bench

您能看到类似的输出::

    2013-05-16 13:08:46-0300 [scrapy] INFO: Scrapy 0.17.0 started (bot: scrapybot)
    2013-05-16 13:08:47-0300 [follow] INFO: Spider opened
    2013-05-16 13:08:47-0300 [follow] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:48-0300 [follow] INFO: Crawled 74 pages (at 4440 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:49-0300 [follow] INFO: Crawled 143 pages (at 4140 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:50-0300 [follow] INFO: Crawled 210 pages (at 4020 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:51-0300 [follow] INFO: Crawled 274 pages (at 3840 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:52-0300 [follow] INFO: Crawled 343 pages (at 4140 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:53-0300 [follow] INFO: Crawled 410 pages (at 4020 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:54-0300 [follow] INFO: Crawled 474 pages (at 3840 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:55-0300 [follow] INFO: Crawled 538 pages (at 3840 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:56-0300 [follow] INFO: Crawled 602 pages (at 3840 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:57-0300 [follow] INFO: Closing spider (closespider_timeout)
    2013-05-16 13:08:57-0300 [follow] INFO: Crawled 666 pages (at 3840 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:57-0300 [follow] INFO: Dumping Scrapy stats:
        {'downloader/request_bytes': 231508,
         'downloader/request_count': 682,
         'downloader/request_method_count/GET': 682,
         'downloader/response_bytes': 1172802,
         'downloader/response_count': 682,
         'downloader/response_status_count/200': 682,
         'finish_reason': 'closespider_timeout',
         'finish_time': datetime.datetime(2013, 5, 16, 16, 8, 57, 985539),
         'log_count/INFO': 14,
         'request_depth_max': 34,
         'response_received_count': 682,
         'scheduler/dequeued': 682,
         'scheduler/dequeued/memory': 682,
         'scheduler/enqueued': 12767,
         'scheduler/enqueued/memory': 12767,
         'start_time': datetime.datetime(2013, 5, 16, 16, 8, 47, 676539)}
    2013-05-16 13:08:57-0300 [follow] INFO: Spider closed (closespider_timeout)

这说明了您的Scrapy能以3900页面/分钟的速度爬取。注意，这是一个非常简单，仅跟进链接的spider。
任何您所编写的spider会做更多处理，从而减慢爬取的速度。
减慢的程度取决于spider做的处理以及其是如何被编写的。

未来会有更多的用例会被加入到性能测试套装中，以覆盖更多常见的情景。
