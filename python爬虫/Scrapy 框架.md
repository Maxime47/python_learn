# Scrapy 框架

 

Scrapy是用纯Python实现一个为了爬取网站数据、提取结构性数据而编写的应用框架，用途非常广泛。

 

框架的力量，用户只需要定制开发几个模块就可以轻松的实现一个爬虫，用来抓取网页内容以及各种图片，非常之方便。

 

Scrapy 使用了 Twisted['twɪstɪd](https://segmentfault.com/a/1190000013178839#)异步网络框架来处理网络通讯，可以加快我们的下载速度，不用自己去实现异步框架，并且包含了各种中间件接口，可以灵活的完成各种需求。

 

## **一、****框架示意图**![Scrapy架构图](C:\Users\ZK\Pictures\Saved Pictures\Scrapy架构图.png)

**Scrapy Engine(引擎)**: 负责Spider、ItemPipeline、Downloader、Scheduler中间的通讯，信号、数据传递等。 

**Scheduler(调度器)**: 它负责接受引擎发送过来的Request请求，并按照一定的方式进行整理排列，入队，当引擎需要时，交还给引擎。

**Downloader（下载器）**：负责下载Scrapy Engine(引擎)发送的所有Requests请求，并将其获取到的Responses交还给Scrapy Engine(引擎)，由引擎交给Spider来处理，

**Spider（爬虫）**：它负责处理所有Responses,从中分析提取数据，获取Item字段需要的数据，并将需要跟进的URL提交给引擎，再次进入Scheduler(调度器)，


**Downloader Middlewares（下载中间件）**：你可以当作是一个可以自定义扩展下载功能的组件。

**Spider Middlewares（Spider中间件）**：你可以理解为是一个可以自定扩展和操作引擎和Spider中间通信的功能组件（比如进入Spider的Responses;和从Spider出去的Requests）

## **二、爬虫人员的主要工作**

1、创建一个Scrapy项目 

2、定义提取的Item

3、编写爬取网站的 spider 并提取 Item

4、编写 Item Pipeline 来存储提取到的Item(即数据)

## 三、scrapy基本流程

![1400528-20180611223618766-1192344882[1]](C:\Users\ZK\AppData\Local\Packages\microsoft.microsoftedge_8wekyb3d8bbwe\AC\#!001\MicrosoftEdge\Cache\OFHZX1FO\1400528-20180611223618766-1192344882[1].png)

## **四、scrapy 框架各部分详解**

> ## **1、** [**Scrapy Items**](#topics-items) **：****定义您想抓取的数据**

import scrapy

class TorrentItem(scrapy.Item):

##     name = scrapy.Field()

> ## **2、spiders：****编写提取数据的Spider**

1）：定义初始URL根网址、 针对后续链接的规则以及从页面中提取数据的规则（即写正则或xpath等等）。

2）执行spider，获取数据

注：运行spider来获取网站的数据，并以JSON格式存入到scraped_data.json 文件中:

terminal：scrapy crawl mininova -o scraped_data.json

> ## **3、编写 item** **pipeline** **将item存储到数据库中**

注：

1）、Item在Spider中被收集之后，它将会被传递到Item Pipeline，一些组件会按照一定的顺序执行对Item的处理；

2）每个item pipeline组件(有时称之为“Item Pipeline”)是实现了简单方法的Python类。他们接收到Item并通过它执行一些行为，同时也决定此Item是否继续通过pipeline，或是被丢弃而不再进行处理。

3）item pipeline的一些典型应用：

a）清理HTML数据

b）验证爬取的数据(检查item包含某些字段)

c）查重(并丢弃)

4）将爬取结果保存到数据库中

> ## **4、编写自己的item pipeline**

注：每个item pipiline组件是一个独立的Python类，同时必须实现以下方法:

> #### **1）process_item(item, spider)**

每个item pipeline组件都需要调用该方法，这个方法必须返回一个 Item (或任何继承类)对象， 或是抛出 DropItem 异常，被丢弃的item将不会被之后的pipeline组件所处理。

参数:

item (Item 对象) – 被爬取的item

spider (Spider 对象) – 爬取该item的spider

> #### **2）open_spider(spider)**

当spider被开启时，这个方法被调用。

参数:spider (Spider 对象) – 被开启的spider

> #### **3）close_spider(spider)**

当spider被关闭时，这个方法被调用

参数:spider (Spider 对象) – 被关闭的spider

> ## **5、查看提取到的数据**

执行结束后，查看 scraped_data.json , 将看到提取到的item:

注 ：1）由于 selectors 返回list, 所以值都是以list存储的(除了 url 是直接赋值之外)。

2） [Item Loaders](#topics-loaders) ：可以保存单个数据或者对数据执行额外的处理