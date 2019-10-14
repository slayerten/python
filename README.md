# python
# -*- coding: utf-8 -*-
import scrapy
import re
import requests
from manuela.items import ManuelaItem
from scrapy.pipelines.images import ImagesPipeline

class GetpicSpider(scrapy.Spider):
    name = 'getpic'
#    allowed_domains = ['www.meitulu.com/t/manuela']

    def get_urls(who):

        code = requests.get("https://www.meitulu.com/t/{}".format(who)).content
        patt = re.compile(r'https:.*\d{3,5}\.html')
        urls = set(re.findall(patt, code.decode('utf-8')))
        return urls

    start_urls = get_urls('wangyuchun') | get_urls('wangyuchun/2.html') | get_urls('huangke-christine')

    def parse(self, response):
        item = ManuelaItem()
        item['image_urls'] = response.css('.content center img::attr(src)').extract()
        yield item
        next = "https://www.meitulu.com" + response.css('#pages a::attr(href)').extract()[-1]
        yield scrapy.Request(next)

