# Scrapy传递参数给Spider对象

覆盖原先构造函数，再执行命令时使用`-a`选项

```python
class MySpider(scrapy.Spider):
    name = 'myspider'

    def __init__(self, category='', domain=None, *args, **kwargs):
        super(MySpider, self).__init__(*args, **kwargs)
        self.start_urls = ['http://www.example.com/categories/%s' % category]
        self.domain = domain
```

运行如下命令执行

```bash
scrapy crawl myspider -a category=electronics -a domain=system
```

[https://doc.scrapy.org/en/latest/topics/spiders.html#spider-arguments](https://doc.scrapy.org/en/latest/topics/spiders.html#spider-arguments)