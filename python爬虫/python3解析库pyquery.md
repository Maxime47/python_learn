# python3解析库pyquery



### 1、pyquery安装

 

pip方式安装：

```python
$pip install pyquery

#它依赖cssselect和lxml包
pyquery==1.4.0
  - cssselect [required: >0.7.9, installed: 1.0.3] #CSS选择器并将它转换为XPath表达式
  - lxml [required: >=2.1, installed: 4.2.2] #处理xml和html解析库
```

### 2、pyquery对象初始化

pyquery首先需要传入HTML文本来初始化一个pyquery对象，它的初始化方式有多种，如直接传入字符串，传入URL或者传入文件名

（1）字符串初始化

```
from pyquery import PyQuery as pq

html='''
<div id="wenzhangziti" class="article 389862">
<p>人生是一条没有尽头的路，不要留恋逝去的梦，把命运掌握在自己手中，让我们来掌握自己的命运，别让别人的干扰与诱惑，别让功名与利禄，来打翻我们这坛陈酿已久的命运之酒！</p>
</div>
'''
doc=pq(html)   #初始化并创建pyquery对象
print(type(doc))
print(doc('p').text())

#
<class 'pyquery.pyquery.PyQuery'>
```

 

URL初始化

```python
from pyquery import PyQuery as pq

doc=pq(url='https://www.cnblogs.com/zhangxinqi/p/9218395.html')
print(type(doc))
print(doc('title'))

#
<class 'pyquery.pyquery.PyQuery'>
<title>python3解析库BeautifulSoup4 - Py.qi - 博客园</title>&#13;
```



PyQuery能够从url加载一个html文档，之际上是默认情况下调用python的urllib库去请求响应，如果requests已安装的话它将使用requests来请求响应，那我们就可以使用request的请求参数来构造请求了，实际请求如下：

 

```python
from pyquery import PyQuery as pq
import requests

doc=pq(requests.get(url='https://www.cnblogs.com/zhangxinqi/p/9218395.html').text)
print(type(doc))
print(doc('title'))

#输出同上一样
<class 'pyquery.pyquery.PyQuery'>
```

（3）通过文件初始化

通过本地的HTML文件来构造PyQuery对象

```python
from pyquery import PyQuery as pq

doc=pq(filename='demo.html',parser='html')
#doc=pq(open('demo.html','r',encoding='utf-8').read(),parser='html')  #注意：在读取有中文的HTML文件时，请使用此方法，否则会报解码错误
print(type(doc))
print(doc('p'))
```

### 3、CSS选择器

 

在使用属性选择器中，使用属性选择特定的标签，标签和CSS标识必须引用为字符串，它会过滤筛选符合条件的节点打印输出，返回的是一个PyQuery类型对象

```python
from pyquery import PyQuery as pq
import requests
html='''
<div id="container">
    <ul class="list">
         <li class="item-0">first item</li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
         <li class="item-1 active"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a></li>
     </ul>
 </div>
'''
doc=pq(html,parser='html')
print(doc('#container .list .item-0 a'))
print(doc('.list .item-1'))

#
<a href="link3.html"><span class="bold">third item</span></a><a href="link5.html">fifth item</a>
<li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-1 active"><a href="link4.html">fourth item</a></li>
```

 

### 4、查找节点

PyQuery使用查询函数来查询节点，同jQuery中的函数用法完全相同

(1)查找子节点和子孙节点

使用find()方法获取子孙节点，children()获取子节点，使用以上的HTML代码测试

 

```python
from pyquery import PyQuery as pq
import requests

doc=pq(html,parser='html')
print('find:',doc.find('a'))
print('children:',doc('li').children('a'))
```

(2)获取父节点和祖先节点

parent()方法获取父节点，parents()获取祖先节点

 

```python
doc(.list).parent()
doc(.list).parents()
```

 

(3)获取兄弟节点

siblings()方法用来获取兄弟节点，可以嵌套使用，传入CSS选择器即可继续匹配

 

```python
doc('.list .item-0 .active').siblings('.active')
```

### 5、遍历

 

对于pyquery的选择结果可能是多个字节，也可能是单个节点，类型都是PyQuery类型，它没有返回列表等形式，对于当个节点我们可指直接打印输出或者直接转换成字符串，而对于多个节点的结果，我们需要遍历来获取所有节点可以使用items()方法，它会返回一个生成器，循环得到的每个节点类型依然是PyQuery类型，所以我们可以继续方法来选择节点或属性，内容等

 

```
lis=doc('li').items()
for i in lis:
    print(i('a'))   #继续获取节点下的子节点
```



### 6、获取信息

attr()方法用来获取属性，如返回的结果有多个时可以调用items()方法来遍历获取

 

```
doc('.item-0.active a').attr('href')  #多属性值中间不能有空格
```

 

text()方法用来获取文本内容，它只返回内部的文本信息不包括HTML文本内容，如果想返回包括HTML的文本内容可以使用html()方法，如果结果有多个，text()方法会方法所有节点的文本信息内容并将它们拼接用空格分开返回字符串内容，html()方法只会返回第一个节点的HTML文本，如果要获取所有就需要使用items()方法来遍历获取了

```python
from pyquery import PyQuery as pq
html='''
<div id="container">
    <ul class="list">
         <li class="item-0">first item</li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
         <li class="item-1 active"><a href="link4.html">fourth item</a></li>
         <li class="item-0"><a href="link5.html">fifth item</a></li>
     </ul>
 </div>
'''
doc=pq(html,parser='html')
print('text:',doc('li').text())  #获取li节点下的所有文本信息
lis=doc('li').items()
for i in lis:
    print('html:',i.html()) #获取所有li节点下的HTML文本

#
text: first item second item third item fourth item fifth item
html: first item
html: <a href="link2.html">second item</a>
html: <a href="link3.html"><span class="bold">third item</span></a>
html: <a href="link4.html">fourth item</a>
html: <a href="link5.html">fifth item</a>
```

### 7、节点操作

pyquery提供了一系列方法来对节点进行动态修改，如添加一个class，移除某个节点，修改某个属性的值

addClass()增加Class，removeClass()删除Class

attr()增加属性和值，text()增加文本内容，html()增加HTML文本，remove()移除

```python
from pyquery import PyQuery as pq
import requests
html='''
<div id="container">
    <ul class="list">
         <li id="1">first item</li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-2 active"><a href="link3.html"><span class="bold">third item</span></a></li>
         <li class="item-3 active"><a href="link4.html">fourth item</a></li>
         <li class="item-4"><a href="link5.html">fifth item</a></li>
     </ul>
 </div>
'''
doc=pq(html,parser='html')
print(doc('#1'))
print(doc('#1').add_class('myclass'))  #增加Class
print(doc('.item-1').remove_class('item-1'))  #删除Class
print(doc('#1').attr('name','link'))  #添加属性name=link
print(doc('#1').text('hello world'))   #添加文本
print(doc('#1').html('<span>changed item</span>')) #添加HTML文本
print(doc('.item-2.active a').remove('span'))  #删除节点

#
<li id="1">first item</li>
         
<li id="1" class="myclass">first item</li>
         
<li class=""><a href="link2.html">second item</a></li>
         
<li id="1" class="myclass" name="link">first item</li>
         
<li id="1" class="myclass" name="link">hello world</li>
         
<li id="1" class="myclass" name="link"><span>changed item</span></li>
<a href="link3.html"/>
```



after()在节点后添加值

before()在节点之前插入值

append()将值添加到每个节点

contents()返回文本节点内容

empty()删除节点内容

remove_attr()删除属性

val()设置或获取属性值

另外还有很多节点操作方法，它们和jQuery的用法完全一致，详细请参考：<http://pyquery.readthedocs.io/en/latest/api.html>

[回到顶部](#_labelTop)

### 8、伪类选择器

CSS选择器之所以强大，是因为它支持多种多样的伪类选择器，如：选择第一个节点，最后一个节点，奇偶数节点等。



```python
#!/usr/bin/env python
#coding:utf-8
from pyquery import PyQuery as pq

html='''
<div id="container">
    <ul class="list">
         <li id="1">first item</li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-2 active"><a href="link3.html"><span class="bold">third item</span></a></li>
         <li class="item-3 active"><a href="link4.html">fourth item</a></li>
         <li class="item-4"><a href="link5.html">fifth item</a></li>
     </ul>
     <div><input type="text" value="username"/></div> 
</div>
'''
doc=pq(html,parser='html')
print('第一个li节点:',doc('li:first-child')) #第一个li节点
print('最后一个li节点:',doc('li:last_child')) #最后一个li节点
print('第二个li节点:',doc('li:nth-child(2)'))  #第二个li节点
print('第三个之后的所有li节点:',doc('li:gt(2)'))  #第三个之后的所有li节点
print('偶数的所有li节点:',doc('li:nth-child(2n)'))  #偶数的所有li节点
print('包含文本内容的节点:',doc('li:contains(second)')) #包含文本内容的节点
print('索引第一个节点：',doc('li:eq(0)'))
print('奇数节点:',doc('li:even'))
print('偶数节点:',doc('li:odd'))

#
第一个li节点: <li id="1">first item</li>
         
最后一个li节点: <li class="item-4"><a href="link5.html">fifth item</a></li>
     
第二个li节点: <li class="item-1"><a href="link2.html">second item</a></li>
         
第三个之后的所有li节点: <li class="item-3 active"><a href="link4.html">fourth item</a></li>
         <li class="item-4"><a href="link5.html">fifth item</a></li>
     
偶数的所有li节点: <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-3 active"><a href="link4.html">fourth item</a></li>
         
包含文本内容的节点: <li class="item-1"><a href="link2.html">second item</a></li>
         
索引第一个节点： <li id="1">first item</li>
         
奇数节点: <li id="1">first item</li>
         <li class="item-2 active"><a href="link3.html"><span class="bold">third item</span></a></li>
         <li class="item-4"><a href="link5.html">fifth item</a></li>
     
偶数节点: <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-3 active"><a href="link4.html">fourth item</a></li>
```



更多伪类参考：<http://pyquery.readthedocs.io/en/latest/pseudo_classes.html>

更多css选择器参考：<http://www.w3school.com.cn/cssref/css_selectors.asp>

### 9、实例应用

 抓取http://www.mzitu.com网站美女图片12万张用时28分钟，总大小9G，主要受网络带宽影响，下载数据有点慢

```python
#!/usr/bin/env python
#coding:utf-8
import requests
from requests.exceptions import RequestException
from pyquery import PyQuery as pq
from PIL import Image
from PIL import ImageFile
from io import BytesIO
import time
from multiprocessing import Pool,freeze_support
ImageFile.LOAD_TRUNCATED_IMAGES = True

headers={
'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.62 Safari/537.36'
,'Referer':'http://www.mzitu.com'
}

img_headers={
'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.62 Safari/537.36'
,'Referer':'http://i.meizitu.net'
}
#保持会话请求
sesion=requests.Session()

#获取首页所有URL并返回列表
def get_url(url):
    list_url = []
    try:
        html=sesion.get(url,headers=headers).text
        doc=pq(html,parser='html')
        url_path=doc('#pins > li').children('a')
        for i in url_path.items():
            list_url.append(i.attr('href'))
    except RequestException as e:
        print('get_url_RequestException:',e)
    except Exception as e:
        print('get_url_Exception:',e)
    return list_url

#组合首页中每个地址的图片分页返回列表
def list_get_pages(list_url):
    list_url_fen=[]
    try:
        for i in list_url:
            doc_children = pq(sesion.get(i,headers=headers).text,parser='html')
            img_number = doc_children('body > div.main > div.content > div.pagenavi > a:nth-last-child(2) > span').text()
            number=int(img_number.strip())
            for j in range(1,number+1):
                list_url_fen.append(i+'/'+str(j))
    except ValueError as e:
        print('list_get_pages_ValueError:',e)
    except RequestException as e:
        print('list_get_pages_RequestException',e)
    except Exception as e:
        print('list_get_pages_Exception:',e)
    return list_url_fen

#获取image地址并下载图片
def get_image(url):
    im_path=''
    try:
        html=sesion.get(url, headers=headers).text
        doc=pq(html,parser='html')
        im_path=doc('.main-image a img').attr('src')
        image_names = ''.join(im_path.split('/')[-3:])
        image_path = 'D:\images\\' + image_names
        with open('img_url.txt','a') as f:
            f.write(im_path + '\n')
        r=requests.get(im_path,headers=img_headers)
        b=BytesIO(r.content)
        i=Image.open(b)
        i.save(image_path)
        b.close()
        i.close()
        #print('下载图片:{}成功！'.format(image_names))
    except RequestException as e:
        print('RequestException:',e)
    except OSError as e:
        print('OSError:',e)
    except Exception as e:   #必须捕获所有异常，运行中有一些链接地址不符合抓取规律，需要捕获异常使程序正常运行
        print('Exception:',e)
    return im_path


#主调用函数
def main(item):
    url1='http://www.mzitu.com/page/{}'.format(item) #分页地址
    print('开始下载地址：{}'.format(url1))
    获取首页链接地址
    html=get_url(url1)
    #获取分页链接地址
    list_fenurl = list_get_pages(html)
    #根据分页链接地址获取图片并下载
    for i in list_fenurl:
        get_image(i)
    return len(list_fenurl)  #统计下载数

if __name__ == '__main__':
    freeze_support()  #windows下进程调用时必须添加
    pool=Pool()    #创建进程池
    start=time.time()
    count=pool.map(main,[i for i in range(1,185)])  #多进程运行翻页主页
    print(sum(count),count)  #获取总的下载数
    end=time.time()
    data=time.strftime('%M:%S',time.localtime(end-start))  #获取程序运行时间
    print('程序运行时间:{}分{}秒'.format(*data.split(':')))

#学习阶段，代码写得通用性很差，以后改进！
#运行结果
#注意：如果是固定IP网需要使用代理

开始下载地址：http://www.mzitu.com/page/137
OSError: image file is truncated (22 bytes not processed)
开始下载地址：http://www.mzitu.com/page/138
125571 
程序运行时间:28分27秒

进程完成，退出码 0
```