# Scrapy不同Spider指定不同pipeline

有时需要在同一个项目中有多个`Spider`类，为不同的`Spider`类指定不同的pipeline，只要将配置`settings.py`文件中对`pipeline`
配置去除，加到不同的`Spider`类即可。

```python
class GupiaoBaiduSpider(scrapy.Spider):
    name = 'gupiao_baidu'
    allowed_domains = ['gupiao.baidu.com']
    start_urls = []
    
    custom_settings = {
        'ITEM_PIPELINES': {
            'stockSpider.pipelines.StockspiderPipeline': 300
        }
    }
```

[原答案](https://stackoverflow.com/questions/8372703/how-can-i-use-different-pipelines-for-different-spiders-in-a-single-scrapy-proje)