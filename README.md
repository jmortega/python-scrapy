# python-scrapy

items.py

```objectivec

import scrapy

class PostugrItem(scrapy.Item):
		# define the fields for your item here like:
		# name = scrapy.Field()
		titulo = scrapy.Field()
		autor = scrapy.Field()
		contenido = scrapy.Field()
		etiquetas = scrapy.Field()	
		categorias = scrapy.Field()
		
```

pipelines.py

```objectivec

import scrapy
from scrapy import signals
from scrapy.contrib.exporter import CsvItemExporter
from scrapy.contrib.exporter import XmlItemExporter
import codecs
import json
import csv

class PostugrPipeline(object):
    def __init__(self):
        self.file = codecs.open('postUGR_items.json', 'w+b', encoding='utf-8')

    def process_item(self, item, spider):
        line = json.dumps(dict(item), ensure_ascii=False) + "\n"
        self.file.write(line)
        return item

    def spider_closed(self, spider):
        self.file.close()
		

class CVSExport(object):

    def __init__(self):
        self.files = {}

    @classmethod
    def from_crawler(cls, crawler):
         pipeline = cls()
         crawler.signals.connect(pipeline.spider_opened, signals.spider_opened)
         crawler.signals.connect(pipeline.spider_closed, signals.spider_closed)
         return pipeline

    def spider_opened(self, spider):
        file = open('postUGR_items.csv', 'w+b')
        self.files[spider] = file
        self.exporter = CsvItemExporter(file)
        self.exporter.start_exporting()

    def spider_closed(self, spider):
        self.exporter.finish_exporting()
        file = self.files.pop(spider)
        file.close()

    def process_item(self, item, spider):
    	self.exporter.export_item(item)
    	return item
		
class XmlExportWithLabels(object):

    def __init__(self):
        self.files = {}

    @classmethod
    def from_crawler(cls, crawler):
         pipeline = cls()
         crawler.signals.connect(pipeline.spider_opened, signals.spider_opened)
         crawler.signals.connect(pipeline.spider_closed, signals.spider_closed)
         return pipeline

    def spider_opened(self, spider):
        file = open('postUGR_withLabel.xml', 'w+b')
        self.files[spider] = file
        self.exporter = XmlItemExporter(file)
        self.exporter.start_exporting()

    def spider_closed(self, spider):
		self.exporter.finish_exporting()
		file = self.files.pop(spider)
		file.close()
		
		#Construcción de la tabla html
		ficheroHTML = open('postUGR_items.html', "w")
		ficheroHTML.write("<!DOCTYPE html>\n<html>\n<header>\n<title>Fichero CSV a HTML</title>\n</header>\n<body>\n<table border=\"1\">\n")
       
		hdlFicheroCSV = open('postUGR_items.csv', "rb")
		objFilaCSV = csv.reader(hdlFicheroCSV)
		rownum = 0
		header = "Campo"
		for row in objFilaCSV:
			ficheroHTML.write("<tr>")
			colnum = 0
			for col in row:
				ficheroHTML.write("<td>")
				ficheroHTML.write(col)
				ficheroHTML.write("</td>")
				colnum += 1
		ficheroHTML.write("</tr>")      
		rownum += 1
		ficheroHTML.write("</table>\n</body>\n</html>")
		ficheroHTML.close()
			
    def process_item(self, item, spider):
    	if item['etiquetas']:
    		self.exporter.export_item(item)
    	return item

class XmlExportWithOutLabels(object):

    def __init__(self):
        self.files = {}

    @classmethod
    def from_crawler(cls, crawler):
         pipeline = cls()
         crawler.signals.connect(pipeline.spider_opened, signals.spider_opened)
         crawler.signals.connect(pipeline.spider_closed, signals.spider_closed)
         return pipeline

    def spider_opened(self, spider):
        file = open('postUGR_withOutLabel.xml', 'w+b')
        self.files[spider] = file
        self.exporter = XmlItemExporter(file)
        self.exporter.start_exporting()

    def spider_closed(self, spider):
        self.exporter.finish_exporting()
        file = self.files.pop(spider)
        file.close()

    def process_item(self, item, spider):
    	if not item['etiquetas']:
    		self.exporter.export_item(item)
    	return item
    	
```


settings.py

```objectivec

BOT_NAME = 'postUGR'

SPIDER_MODULES = ['postUGR.spiders']
NEWSPIDER_MODULE = 'postUGR.spiders'

ITEM_PIPELINES = ['postUGR.pipelines.PostugrPipeline','postUGR.pipelines.CVSExport','postUGR.pipelines.XmlExportWithLabels','postUGR.pipelines.XmlExportWithOutLabels']


```


Spyder


```objectivec

import scrapy
from scrapy.contrib.spiders import CrawlSpider, Rule
from scrapy.contrib.linkextractors import LinkExtractor
from scrapy.contrib.linkextractors.lxmlhtml import LxmlLinkExtractor
from scrapy.selector import HtmlXPathSelector
from postUGR.items import PostugrItem

class PostugrSpyderSpider(CrawlSpider):
    name = "postUGR_spyder"
    allowed_domains = ["osl.ugr.es"]
    start_urls = ['http://osl.ugr.es']

	# Patrón para las entradas que cumplan el formato de fecha aaaa/mm/dd
    rules = [Rule(LxmlLinkExtractor(allow=[r'\d{4}/\d{2}/\d{2}/*']),callback='process_response')]
	
    def process_response(self, response):
		item = PostugrItem()
		item['titulo'] = response.xpath("//h1/text()").extract()
		item['autor'] = response.xpath("//div[contains(@class, 'entry-meta')]//a[contains(@rel, 'author')]/text()").extract()
		item['contenido'] = response.xpath("//section[contains(@class, 'entry-content')]//p/text()").extract()
		item['categorias'] = response.xpath("//div[contains(@class, 'entry-meta')]/a[1]/text()").extract()
		item['etiquetas'] = response.xpath("//div[contains(@class, 'entry-meta')]/a/text()").extract()
		return item
		
```
