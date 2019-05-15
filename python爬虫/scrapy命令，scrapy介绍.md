scrapy命令，scrapy介绍



一，命令

1.创建一个新的项目

scrapy startproject [项目名]



2.生成爬虫

scrapy genspider +文件名+网址



3.运行(crawl)

scrapy crawl +爬虫名称

scrapy crawl [爬虫名] -o zufang.json

\# -o output

scrapy crawl [爬虫名] -o zufang.csv



4.check检查错误

scrapy check



5.list返回项目所有spider名称

scrapy list



\6. view 存储、打开网页

scrapy view https://www.baidu.com



\7. scrapy shell，进入终端

scrapy shell https://www.baidu.com



\8. scrapy runspider

scrapy runspider zufang_spider.py





二，中间件

\# 参考文章：

\# https://www.jianshu.com/p/a94d7de5560f



下载器中间件按照优先级被调用的：当request从引擎向下载器传递时，数字小的下载器中间件先执行，数字大的后执行；当下载器将response向引擎传递，数字大的下载器中间件先执行，小的后执行。





scrapy提供了一套基本的下载器中间件，



{

​    'scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware': 100,

​    'scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware': 300,

​    'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware': 350,

​    'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware': 400,

​    'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': 500,

​    'scrapy.downloadermiddlewares.retry.RetryMiddleware': 550,

​    'scrapy.downloadermiddlewares.ajaxcrawl.AjaxCrawlMiddleware': 560,

​    'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware': 580,

​    'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 590,

​    'scrapy.downloadermiddlewares.redirect.RedirectMiddleware': 600,

​    'scrapy.downloadermiddlewares.cookies.CookiesMiddleware': 700,

​    'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware': 750,

​    'scrapy.downloadermiddlewares.stats.DownloaderStats': 850,

​    'scrapy.downloadermiddlewares.httpcache.HttpCacheMiddleware': 900,

}



见链接

<https://docs.scrapy.org/en/latest/topics/settings.html#std:setting-DOWNLOADER_MIDDLEWARES_BASE>









下载器中间件是个类，类里可以定义方法，例如process_request()，process_response()，process_exception()



process_request():



process_request()的参数是request, spider



参数request是个字典，字典里包含了headers、url等信息



process_request()可以利用参数request里面的信息，对请求做修改，这时一般返回的是None，典型的任务是修改User-agent、变换代理



如果根据参数request里的url直接就去做抓取，返回response对象，返回的response对象就会不经过剩下的下载器中间件，直接返回到引擎



如果对请求做了修改，返回的是request对象，就会发回到调度器，等待调度







process_response(request, response, spider)



返回的必须是Response、Request或IgnoreRequest异常





三，爬虫中间件

爬虫中间件的作用：

处理引擎传递给爬虫的响应；

处理爬虫传递给引擎的请求；

处理爬虫传递给引擎的数据项。



scrapy提供的基本爬虫中间件

<https://docs.scrapy.org/en/latest/topics/settings.html#std:setting-SPIDER_MIDDLEWARES_BASE>



如何自定义爬虫中间件

<https://docs.scrapy.org/en/latest/topics/spider-middleware.html#writing-your-own-spider-middleware>







四，管道

<https://docs.scrapy.org/en/latest/topics/item-pipeline.html>



每个管道组件都是一个实现了某个功能的Python类，常见功能有：

清理html数据

做确认

查重

存入数据库



每个管道组件的类，必须要有以下方法：

process_item(self, item, spider)

open_spider(self, spider)

close_spider(self, spider)

from_crawler(cls, crawler)





\# 丢弃数据项

from scrapy.exceptions import DropItem



class PricePipeline(object):



​    vat_factor = 1.15



​    def process_item(self, item, spider):

​        if item['price']:

​            if item['price_excludes_vat']:

​                item['price'] = item['price'] * self.vat_factor

​            return item

​        else:

​            raise DropItem("Missing price in %s" % item)





\# 存储到MongoDB



import pymongo



class MongoPipeline(object):



​    collection_name = 'scrapy_items'



​    def __init__(self, mongo_uri, mongo_db):

​        self.mongo_uri = mongo_uri

​        self.mongo_db = mongo_db



​    @classmethod

​    def from_crawler(cls, crawler):

​        return cls(

​            mongo_uri=crawler.settings.get('MONGO_URI'),

​            mongo_db=crawler.settings.get('MONGO_DATABASE', 'items')

​        )



​    def open_spider(self, spider):

​        self.client = pymongo.MongoClient(self.mongo_uri)

​        self.db = self.client[self.mongo_db]



​    def close_spider(self, spider):

​        self.client.close()



​    def process_item(self, item, spider):

​        self.db[self.collection_name].insert_one(dict(item))

​        return item





\# 存储到MySQL

class MysqlPipeline():

​    def __init__(self, host, database, user, password, port):

​        self.host = host

​        self.database = database

​        self.user = user

​        self.password = password

​        self.port = port



​    @classmethod

​    def from_crawler(cls, crawler):

​        return cls(

​            host=crawler.settings.get('MYSQL_HOST'),

​            database=crawler.settings.get('MYSQL_DATABASE'),

​            user=crawler.settings.get('MYSQL_USER'),

​            password=crawler.settings.get('MYSQL_PASSWORD'),

​            port=crawler.settings.get('MYSQL_PORT'),

​        )



​    def open_spider(self, spider):

​        self.db = pymysql.connect(self.host, self.user, self.password, self.database, charset='utf8',

​                                  port=self.port)

​        self.cursor = self.db.cursor()



​    def close_spider(self, spider):

​        self.db.close()



​    def process_item(self, item, spider):

​        print(item['title'])

​        data = dict(item)

​        keys = ', '.join(data.keys())

​        values = ', '.join(['%s'] * len(data))

​        sql = 'insert into %s (%s) values (%s)' % (item.table, keys, values)

​        self.cursor.execute(sql, tuple(data.values()))

​        self.db.commit()

​        return item



\# 去重

from scrapy.exceptions import DropItem



class DuplicatesPipeline(object):



​    def __init__(self):

​        self.ids_seen = set()



​    def process_item(self, item, spider):

​        if item['id'] in self.ids_seen:

​            raise DropItem("Duplicate item found: %s" % item)

​        else:

​            self.ids_seen.add(item['id'])

​            return item



\# 激活管道

ITEM_PIPELINES = {

​    'myproject.pipelines.PricePipeline': 300,

​    'myproject.pipelines.JsonWriterPipeline': 800,

}





五，settings文件



\# -*- coding: utf-8 -*-



\# Scrapy settings for downloadertest project

\#

\# For simplicity, this file contains only settings considered important or

\# commonly used. You can find more settings consulting the documentation:

\#

\#     https://doc.scrapy.org/en/latest/topics/settings.html

\#     https://doc.scrapy.org/en/latest/topics/downloader-middleware.html

\#     https://doc.scrapy.org/en/latest/topics/spider-middleware.html



BOT_NAME = 'downloadertest'



SPIDER_MODULES = ['downloadertest.spiders']

NEWSPIDER_MODULE = 'downloadertest.spiders'





\# Crawl responsibly by identifying yourself (and your website) on the user-agent

\#USER_AGENT = 'downloadertest (+http://www.yourdomain.com)'

\#USER_AGENT = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_1) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/12.0.1 Safari/605.1.15'



\# Obey robots.txt rules

ROBOTSTXT_OBEY = True



\# Configure maximum concurrent requests performed by Scrapy (default: 16)

\#CONCURRENT_REQUESTS = 32



\# Configure a delay for requests for the same website (default: 0)

\# See https://doc.scrapy.org/en/latest/topics/settings.html#download-delay

\# See also autothrottle settings and docs

\#DOWNLOAD_DELAY = 3

\# The download delay setting will honor only one of:

\#CONCURRENT_REQUESTS_PER_DOMAIN = 16

\#CONCURRENT_REQUESTS_PER_IP = 16



\# Disable cookies (enabled by default)

\#COOKIES_ENABLED = False



\# Disable Telnet Console (enabled by default)

\#TELNETCONSOLE_ENABLED = False



\# Override the default request headers:

\#DEFAULT_REQUEST_HEADERS = {

\#   'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',

\#   'Accept-Language': 'en',

\#}



\# Enable or disable spider middlewares

\# See https://doc.scrapy.org/en/latest/topics/spider-middleware.html

\#SPIDER_MIDDLEWARES = {

\#    'downloadertest.middlewares.DownloadertestSpiderMiddleware': 543,

\#}



\# Enable or disable downloader middlewares

\# See https://doc.scrapy.org/en/latest/topics/downloader-middleware.html

\#DOWNLOADER_MIDDLEWARES = {

\#    'downloadertest.middlewares.DownloadertestDownloaderMiddleware': 543,

\#}

DOWNLOADER_MIDDLEWARES = {

   'downloadertest.middlewares.RandomUA': 543,

   'downloadertest.middlewares.ProxyMiddleware': 542,

}



\# Enable or disable extensions

\# See https://doc.scrapy.org/en/latest/topics/extensions.html

\#EXTENSIONS = {

\#    'scrapy.extensions.telnet.TelnetConsole': None,

\#}



\# Configure item pipelines

\# See https://doc.scrapy.org/en/latest/topics/item-pipeline.html

\#ITEM_PIPELINES = {

\#    'downloadertest.pipelines.DownloadertestPipeline': 300,

\#}



\# Enable and configure the AutoThrottle extension (disabled by default)

\# See https://doc.scrapy.org/en/latest/topics/autothrottle.html

\#AUTOTHROTTLE_ENABLED = True

\# The initial download delay

\#AUTOTHROTTLE_START_DELAY = 5

\# The maximum download delay to be set in case of high latencies

\#AUTOTHROTTLE_MAX_DELAY = 60

\# The average number of requests Scrapy should be sending in parallel to

\# each remote server

\#AUTOTHROTTLE_TARGET_CONCURRENCY = 1.0

\# Enable showing throttling stats for every response received:

\#AUTOTHROTTLE_DEBUG = False



\# Enable and configure HTTP caching (disabled by default)

\# See https://doc.scrapy.org/en/latest/topics/downloader-middleware.html#httpcache-middleware-settings

\#HTTPCACHE_ENABLED = True

\#HTTPCACHE_EXPIRATION_SECS = 0

\#HTTPCACHE_DIR = 'httpcache'

\#HTTPCACHE_IGNORE_HTTP_CODES = []

\#HTTPCACHE_STORAGE = 'scrapy.extensions.httpcache.FilesystemCacheStorage'