---
title: scrapy笔记
date: 2022-07-15 11:21:28
tags:
---
# scrapy笔记

## 1. 添加随机UserAgent

~~~python
from fake_useragent import UserAgent


def get_random_user_agent(path):
    res = UserAgent(path=path).random
    # print(res)
    return res
~~~

~~~python
from fictiondemo.utils import get_random_user_agent


class RandomUserAgentMiddleware:
    @classmethod
    def from_crawler(cls, crawler):
        # This method is used by Scrapy to create your spiders.
        s = cls()
        crawler.signals.connect(s.spider_opened, signal=signals.spider_opened)
        return s

    def process_request(self, request, spider):
        random_user_agent = get_random_user_agent(path='config/fake_useragent.json')
        request.headers['User-Agent'] = random_user_agent
        return None

    def process_response(self, request, response, spider):
        return response

    def process_exception(self, request, exception, spider):
        pass

    def spider_opened(self, spider):
        spider.logger.info('Spider opened: %s' % spider.name)

~~~

```python
DOWNLOADER_MIDDLEWARES = {
    # None表示禁用
    'fictiondemo.middlewares.FictiondemoDownloaderMiddleware': None,
    'fictiondemo.middlewares.RandomUserAgentMiddleware': 1,
}
```

## 2. crawlspider实例

```python
import re

from scrapy.http import HtmlResponse
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule


class TxbcSpider(CrawlSpider):
    name = 'txbc'
    # allowed_domains = ['tianxiabachang.zuopinj.com']
    start_urls = ['http://tianxiabachang.zuopinj.com/']

    rules = (
        # \d{3,5} 表示3~5（不含5）个数字  也就是一个前闭后开区间
        # callback 指定是否需要回调函数来处理本次的响应
        # follow=True 拿到本次提取到的链接 继续发请求 如果有callback就将请求得到的响应放到callback函数中
        Rule(LinkExtractor(allow=r'/\d{3,5}/$'), follow=True),
        Rule(LinkExtractor(allow=r'/\d+\.html$'), callback='parse_item', follow=True),
    )

    def parse_item(self, response: HtmlResponse):
        item = {}
        # http://tianxiabachang.zuopinj.com/11350/299356.html
        url = response.url
        res = re.findall(r'http://tianxiabachang.zuopinj.com/(.*?)/(.*?).html', url)[0]
        book_id = res[0]
        chapter_id = res[1]
        title = response.xpath('//h1/text()').get(None)
        ps = response.xpath('//*[@id="htmlContent"]//p')
        for index, p in enumerate(ps, start=1):
            content = p.xpath('.//text()').get(None)
            if any([content is None, content == '\xa0\xa0']):
                continue
            # print(book_id, chapter_id, index, title, res)
            item.setdefault('book_id', book_id)
            item.setdefault('chapter_id', chapter_id)
            item.setdefault('index', index)
            item.setdefault('title', title)
            item.setdefault('content', content)

        return item

```
