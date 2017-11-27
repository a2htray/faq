# Scrapy读取配置信息

## 继承于`scrapy.Spider`对象

直接通过访问`self.settings`，便可访问

```python
class RecommendSpider(scrapy.Spider):

    def start_request(self):
        print self.settings
```

## pipelines中读取

通过创建类方法`from_settings`

```python
class StockspiderPipeLine(object):

    @classmethod
    def from_settings(cls, settings):
        print settings
```