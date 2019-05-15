# Newspaper框架

Newspaper框架是Python爬虫框架中在GitHub上点赞排名第三的爬虫框架，适合抓取新闻网页。

推荐安装Python3版本：`pip3 install newspaper3k` （pip install newspaper是Python2版本）



1. 基本使用方法

```python
url = 'https://www.washingtonpost.com/powerpost/trump-to-make-new-offer-to-democrats-as-government-shutdown-drags-on/2019/01/19/2cde029e-1bf3-11e9-9ebf-c5fed1b7a081_story.html?utm_term=.4db5c2055c6d'

# 创建文章对象
article = Article(url)

# 下载网页
article.download()

# 打印html文档
print(article.html)

# 网页解析
article.parse()

# 标题
print(article.title)

# # 作者
print(article.authors)

# 发布日期
print(article.publish_date)

# 正文
print(article.text)

# 配图
print(article.top_image)

# 视频
print(article.movies)


# 自然语言处理
article.nlp()

# 关键词
print(article.keywords)

# 文章摘要
print(article.summary)
```

1. 整体抓取首页

```python
import newspaper

# 构建新闻源
washingtonpost_paper = newspaper.build('https://www.washingtonpost.com')

# 所有文章的url
for article in washingtonpost_paper.articles:
    print(article.url)

# 文章分裂
for category in washingtonpost_paper.category_urls():
    print(category)
```

1. Requests和Newspaper结合解析正文

```python
import requests
from newspaper import fulltext

html = requests.get('https://www.washingtonpost.com/business/economy/2019/01/17/19662748-1a84-11e9-9ebf-c5fed1b7a081_story.html?utm_term=.26198c91916f').text
text = fulltext(html)

print(text)
```

1. Google Trends信息

```python
import newspaper

# Google的新闻热点
print(newspaper.hot())

# 流行网站
print(newspaper.popular_urls())
```

1. 多任务

```python
import newspaper
from newspaper import news_pool

# 创建并行任务
slate_paper = newspaper.build('http://slate.com')
tc_paper = newspaper.build('http://techcrunch.com')
espn_paper = newspaper.build('http://espn.com')

papers = [slate_paper, tc_paper, espn_paper]
news_pool.set(papers, threads_per_source=2) # (3*2) = 6 共6个线程

news_pool.join()

print(slate_paper.articles[10].html)
```

作者：SeanCheney

链接：https://www.jianshu.com/p/1c12e0cc9a2e

来源：简书