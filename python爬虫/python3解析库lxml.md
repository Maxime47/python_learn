# python3解析库lxml

lxml是python的一个解析库，支持HTML和XML的解析，支持XPath解析方式，而且解析效率非常高

 

XPath，全称XML Path Language，即XML路径语言，它是一门在XML文档中查找信息的语言，它最初是用来搜寻XML文档的，但是它同样适用于HTML文档的搜索



### XPath常用规则

 

| 表达式            | 描述                                       |
| ----------------- | ------------------------------------------ |
| nodename          | 选取此节点的所有子节点                     |
| /                 | 从当前节点选取直接子节点                   |
| //                | 从当前节点选取子孙节点                     |
| .                 | 选取当前节点                               |
| ..                | 选取当前节点的父节点                       |
| @                 | 选取属性                                   |
| *                 | 通配符，选择所有元素节点与元素名           |
| @*                | 选取所有属性                               |
| [@attrib]         | 选取具有给定属性的所有元素                 |
| [@attrib='value'] | 选取给定属性具有给定值的所有元素           |
| [tag]             | 选取所有具有指定元素的直接子节点           |
| [tag='text']      | 选取所有具有指定元素并且文本内容是text节点 |

**（1）读取文本解析节点**



```
from lxml import etree

text='''
<div>
    <ul>
         <li class="item-0"><a href="link1.html">第一个</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-0"><a href="link5.html">a属性</a>
     </ul>
 </div>
'''
html=etree.HTML(text) #初始化生成一个XPath解析对象
result=etree.tostring(html,encoding='utf-8')   #解析对象输出代码
print(type(html))
print(type(result))
print(result.decode('utf-8'))

#etree会修复HTML文本节点
<class 'lxml.etree._Element'>
<class 'bytes'>
<html><body><div>
    <ul>
         <li class="item-0"><a href="link1.html">第一个</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
         <li class="item-0"><a href="link5.html">a属性</a>
     </li></ul>
 </div>
</body></html>
```



### （2）读取HTML文件进行解析



```
from lxml import etree

html=etree.parse('test.html',etree.HTMLParser()) #指定解析器HTMLParser会根据文件修复HTML文件中缺失的如声明信息
result=etree.tostring(html)   #解析成字节
#result=etree.tostringlist(html) #解析成列表
print(type(html))
print(type(result))
print(result)

#
<class 'lxml.etree._ElementTree'>
<class 'bytes'>
b'<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN" "http://www.w3.org/TR/REC-html40/loose.dtd">\n<html><body><div>&#13;\n    <ul>&#13;\n         <li class="item-0"><a href="link1.html">first item</a></li>&#13;\n         <li class="item-1"><a href="link2.html">second item</a></li>&#13;\n         <li class="item-inactive"><a href="link3.html">third item</a></li>&#13;\n         <li class="item-1"><a href="link4.html">fourth item</a></li>&#13;\n         <li class="item-0"><a href="link5.html">fifth item</a>&#13;\n     </li></ul>&#13;\n </div>&#13;\n</body></html>'
```





#### （3）获取所有节点

返回一个列表每个元素都是Element类型，所有节点都包含在其中



```
from lxml import etree

html=etree.parse('test',etree.HTMLParser())
result=html.xpath('//*')  #//代表获取子孙节点，*代表获取所有

print(type(html))
print(type(result))
print(result)

#
<class 'lxml.etree._ElementTree'>
<class 'list'>
[<Element html at 0x754b210048>, <Element body at 0x754b210108>, <Element div at 0x754b210148>, <Element ul at 0x754b210188>, <Element li at 0x754b2101c8>, <Element a at 0x754b210248>, <Element li at 0x754b210288>, <Element a at 0x754b2102c8>, <Element li at 0x754b210308>, <Element a at 0x754b210208>, <Element li at 0x754b210348>, <Element a at 0x754b210388>, <Element li at 0x754b2103c8>, <Element a at 0x754b210408>]
```



如要获取li节点，可以使用//后面加上节点名称，然后调用xpath()方法

```
html.xpath('//li')   #获取所有子孙节点的li节点
```



#### （4）获取子节点

通过/或者//即可查找元素的子节点或者子孙节点，如果想选择li节点的所有直接a节点，可以这样使用

```
result=html.xpath('//li/a')  #通过追加/a选择所有li节点的所有直接a节点，因为//li用于选中所有li节点，/a用于选中li节点的所有直接子节点a
```



#### （5）获取父节点

我们知道通过连续的/或者//可以查找子节点或子孙节点，那么要查找父节点可以使用..来实现也可以使用parent::来获取父节点

```
from lxml import etree
from lxml.etree import HTMLParser
text='''
<div>
    <ul>
         <li class="item-0"><a href="link1.html">第一个</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
     </ul>
 </div>
'''

html=etree.HTML(text,etree.HTMLParser())
result=html.xpath('//a[@href="link2.html"]/../@class')
result1=html.xpath('//a[@href="link2.html"]/parent::*/@class')
print(result)
print(result1)


#
['item-1']
['item-1']
```



#### （6）属性匹配

在选取的时候，我们还可以用`@`符号进行属性过滤。比如，这里如果要选取`class`为`item-1`的`li`节点，可以这样实现:



```
from lxml import etree
from lxml.etree import HTMLParser
text='''
<div>
    <ul>
         <li class="item-0"><a href="link1.html">第一个</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
     </ul>
 </div>
'''

html=etree.HTML(text,etree.HTMLParser())
result=html.xpath('//li[@class="item-1"]')
print(result)
```



#### （7）文本获取

我们用XPath中的text()方法获取节点中的文本



```
from lxml import etree

text='''
<div>
    <ul>
         <li class="item-0"><a href="link1.html">第一个</a></li>
         <li class="item-1"><a href="link2.html">second item</a></li>
     </ul>
 </div>
'''

html=etree.HTML(text,etree.HTMLParser())
result=html.xpath('//li[@class="item-1"]/a/text()') #获取a节点下的内容
result1=html.xpath('//li[@class="item-1"]//text()') #获取li下所有子孙节点的内容

print(result)
print(result1)
```



#### （8）属性获取

使用@符号即可获取节点的属性，如下：获取所有li节点下所有a节点的href属性

```
result=html.xpath('//li/a/@href')  #获取a的href属性
result=html.xpath('//li//@href')   #获取所有li子孙节点的href属性
```

#### （9）属性多值匹配

如果某个属性的值有多个时，我们可以使用contains()函数来获取



```
from lxml import etree

text1='''
<div>
    <ul>
         <li class="aaa item-0"><a href="link1.html">第一个</a></li>
         <li class="bbb item-1"><a href="link2.html">second item</a></li>
     </ul>
 </div>
'''

html=etree.HTML(text1,etree.HTMLParser())
result=html.xpath('//li[@class="aaa"]/a/text()')
result1=html.xpath('//li[contains(@class,"aaa")]/a/text()')

print(result)
print(result1)

#通过第一种方法没有取到值，通过contains（）就能精确匹配到节点了
[]
['第一个']
```



#### （10）多属性匹配

另外我们还可能遇到一种情况，那就是根据多个属性确定一个节点，这时就需要同时匹配多个属性，此时可用运用and运算符来连接使用：

[![复制代码](https:////common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
from lxml import etree

text1='''
<div>
    <ul>
         <li class="aaa" name="item"><a href="link1.html">第一个</a></li>
         <li class="aaa" name="fore"><a href="link2.html">second item</a></li>
     </ul>
 </div>
'''

html=etree.HTML(text1,etree.HTMLParser())
result=html.xpath('//li[@class="aaa" and @name="fore"]/a/text()')
result1=html.xpath('//li[contains(@class,"aaa") and @name="fore"]/a/text()')


print(result)
print(result1)


#
['second item']
['second item']
```

#### （11）XPath中的运算符

| 运算符 | 描述             | 实例              | 返回值                                         |
| ------ | ---------------- | ----------------- | ---------------------------------------------- |
| or     | 或               | age=19 or age=20  | 如果age等于19或者等于20则返回true反正返回false |
| and    | 与               | age>19 and age<21 | 如果age等于20则返回true，否则返回false         |
| mod    | 取余             | 5 mod 2           | 1                                              |
| \|     | 取两个节点的集合 | //book \| //cd    | 返回所有拥有book和cd元素的节点集合             |
| +      | 加               | 6+4               | 10                                             |
| -      | 减               | 6-4               | 2                                              |
| *      | 乘               | 6*4               | 24                                             |
| div    | 除法             | 8 div 4           | 2                                              |
| =      | 等于             | age=19            | true                                           |
| !=     | 不等于           | age!=19           | true                                           |
| <      | 小于             | age<19            | true                                           |
| <=     | 小于或等于       | age<=19           | true                                           |
| >      | 大于             | age>19            | true                                           |
| >=     | 大于或等于       | age>=19           | true                                           |

此表参考来源:<http://www.w3school.com.cn/xpath/xpath_operators.asp>

#### （12）按序选择

有时候，我们在选择的时候某些属性可能同时匹配多个节点，但我们只想要其中的某个节点，如第二个节点或者最后一个节点，这时可以利用中括号引入索引的方法获取特定次序的节点：

[![复制代码](https:////common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
from lxml import etree

text1='''
<div>
    <ul>
         <li class="aaa" name="item"><a href="link1.html">第一个</a></li>
         <li class="aaa" name="item"><a href="link1.html">第二个</a></li>
         <li class="aaa" name="item"><a href="link1.html">第三个</a></li>
         <li class="aaa" name="item"><a href="link1.html">第四个</a></li> 
     </ul>
 </div>
'''

html=etree.HTML(text1,etree.HTMLParser())

result=html.xpath('//li[contains(@class,"aaa")]/a/text()') #获取所有li节点下a节点的内容
result1=html.xpath('//li[1][contains(@class,"aaa")]/a/text()') #获取第一个
result2=html.xpath('//li[last()][contains(@class,"aaa")]/a/text()') #获取最后一个
result3=html.xpath('//li[position()>2 and position()<4][contains(@class,"aaa")]/a/text()') #获取第一个
result4=html.xpath('//li[last()-2][contains(@class,"aaa")]/a/text()') #获取倒数第三个


print(result)
print(result1)
print(result2)
print(result3)
print(result4)


#
['第一个', '第二个', '第三个', '第四个']
['第一个']
['第四个']
['第三个']
['第二个']
```

这里使用了last()、position()函数，在XPath中，提供了100多个函数，包括存取、数值、字符串、逻辑、节点、序列等处理功能，它们的具体作用可参考：<http://www.w3school.com.cn/xpath/xpath_functions.asp>

#### （13）节点轴选择

XPath提供了很多节点选择方法，包括获取子元素、兄弟元素、父元素、祖先元素等，示例如下：



```
from lxml import etree

text1='''
<div>
    <ul>
         <li class="aaa" name="item"><a href="link1.html">第一个</a></li>
         <li class="aaa" name="item"><a href="link1.html">第二个</a></li>
         <li class="aaa" name="item"><a href="link1.html">第三个</a></li>
         <li class="aaa" name="item"><a href="link1.html">第四个</a></li> 
     </ul>
 </div>
'''

html=etree.HTML(text1,etree.HTMLParser())
result=html.xpath('//li[1]/ancestor::*')  #获取所有祖先节点
result1=html.xpath('//li[1]/ancestor::div')  #获取div祖先节点
result2=html.xpath('//li[1]/attribute::*')  #获取所有属性值
result3=html.xpath('//li[1]/child::*')  #获取所有直接子节点
result4=html.xpath('//li[1]/descendant::a')  #获取所有子孙节点的a节点
result5=html.xpath('//li[1]/following::*')  #获取当前子节之后的所有节点
result6=html.xpath('//li[1]/following-sibling::*')  #获取当前节点的所有同级节点


#
[<Element html at 0x3ca6b960c8>, <Element body at 0x3ca6b96088>, <Element div at 0x3ca6b96188>, <Element ul at 0x3ca6b961c8>]
[<Element div at 0x3ca6b96188>]
['aaa', 'item']
[<Element a at 0x3ca6b96248>]
[<Element a at 0x3ca6b96248>]
[<Element li at 0x3ca6b96308>, <Element a at 0x3ca6b96348>, <Element li at 0x3ca6b96388>, <Element a at 0x3ca6b963c8>, <Element li at 0x3ca6b96408>, <Element a at 0x3ca6b96488>]
[<Element li at 0x3ca6b96308>, <Element li at 0x3ca6b96388>, <Element li at 0x3ca6b96408>]
```

[![复制代码](https:////common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

以上使用的是XPath轴的用法，更多轴的用法可参考：<http://www.w3school.com.cn/xpath/xpath_axes.asp>

#### （14）案例应用：抓取TIOBE指数前20名排行开发语言

```
#!/usr/bin/env python
#coding:utf-8
import requests
from requests.exceptions import RequestException
from lxml import etree
from lxml.etree import ParseError
import json

def one_to_page(html):
    headers={
        'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.62 Safari/537.36'
    }
    try:
        response=requests.get(html,headers=headers)
        body=response.text  #获取网页内容
    except RequestException as e:
        print('request is error!',e)
    try:
        html=etree.HTML(body,etree.HTMLParser())  #解析HTML文本内容
        result=html.xpath('//table[contains(@class,"table-top20")]/tbody/tr//text()') #获取列表数据
        pos = 0
        for i in range(20):
            if i == 0:
                yield result[i:5]
            else:
                yield result[pos:pos+5]  #返回排名生成器数据
            pos+=5
    except ParseError as e:
         print(e.position)


def write_file(data):   #将数据重新组合成字典写入文件并输出
    for i in data:
        sul={
            '2018年6月排行':i[0],
            '2017年6排行':i[1],
            '开发语言':i[2],
            '评级':i[3],
            '变化率':i[4]
        }
        with open('test.txt','a',encoding='utf-8') as f:
            f.write(json.dumps(sul,ensure_ascii=False) + '\n') #必须格式化数据
            f.close()
        print(sul)
    return None


def main():
    url='https://www.tiobe.com/tiobe-index/'
    data=one_to_page(url)
    revaule=write_file(data)
    if revaule == None:
        print('ok')
        
 
 
        
if __name__ == '__main__':
    main()
```