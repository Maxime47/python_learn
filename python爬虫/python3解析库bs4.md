# python3解析库bs4

bs4 全名 BeautifulSoup，是编写 python 爬虫常用库之一，主要用来解析 html 标签。



# 一、初始化

```
from bs4 import BeautifulSoup

soup = BeautifulSoup("<html>A Html Text</html>", "html.parser")
```

　　两个参数：第一个参数是要解析的html文本，第二个参数是使用那种解析器，对于HTML来讲就是html.parser，这个是bs4自带的解析器。

　　如果一段HTML或XML文档格式不正确的话，那么在不同的解析器中返回的结果可能是不一样的。

| **解析器**   | **使用方法**                                                 | **优势**                                                     |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Python标准库 | BeautifulSoup(html, "html.parser")                           | 1、Python的内置标准库2、执行速度适中3、文档容错能力强        |
| lxml HTML    | BeautifulSoup(html, "lxml")                                  | 1、速度快2、文档容错能力强                                   |
| lxml XML     | BeautifulSoup(html, ["lxml", "xml"])BeautifulSoup(html, "xml") | 1、速度快2、唯一支持XML的解析器                              |
| html5lib     | BeautifulSoup(html, "html5lib")                              | 1、最好的容错性2、以浏览器的方式解析文档3、生成HTML5格式的文档 |

**格式化输出**

```
soup.prettify()  # prettify 有括号和没括号都可以
```

# 二、对象

　　Beautfiful Soup将复杂HTML文档转换成一个复杂的树形结构，每个节点都是Python对象，所有对象可以归纳为4种：tag，NavigableString，BeautifulSoup，Comment。

## 1、tag

　　Tag对象与 xml 或 html 原生文档中的 tag 相同。

```
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')

tag = soup.b

type(tag)

# <class 'bs4.element.Tag'>
```

　　如果不存在，则返回 None，如果存在多个，则返回第一个。

**Name**

　　每个 tag 都有自己的名字

```
tag.name
# 'b'
```

**Attributes**

　　tag 的属性是一个字典

```
tag['class']
# 'boldest'

tag.attrs
# {'class': 'boldest'}

type(tag.attrs)
# <class 'dict'>
```



**多值属性**

　　最常见的多值属性是class，多值属性的返回 list。

```
soup = BeautifulSoup('<p class="body strikeout"></p>')

print(soup.p['class'])  # ['body', 'strikeout']

print(soup.p.attrs)     # {'class': ['body', 'strikeout']}
```

　　如果某个属性看起来好像有多个值,但在任何版本的HTML定义中都没有被定义为多值属性，那么Beautiful Soup会将这个属性作为字符串返回。

```
soup = BeautifulSoup('<p id="my id"></p>', 'html.parser')
print(soup.p['id'])    # 'my id'
```

**Text**

　　text 属性返回 tag 的所有字符串连成的字符串。

**其他方法**

　　tag.has_attr('id') # 返回 tag 是否包含 id 属性

　　当然，以上代码还可以写成 'id' in tag.attrs，之前说过，tag 的属性是一个字典。顺便提一下，has_key是老旧遗留的api，为了支持2.2之前的代码留下的。Python3已经删除了该函数。

## 2、NavigableString

　　字符串常被包含在 tag 内，Beautiful Soup 用 NavigableString 类来包装 tag 中的字符串。但是字符串中不能包含其他 tag。



```py
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')

s = soup.b.string

print(s)        # Extremely bold

print(type(s))  # <class 'bs4.element.NavigableString'>
```

## **3****、BeautifulSoup**

　　BeautifulSoup 对象表示的是一个文档的全部内容。大部分时候，可以把它当作 Tag 对象。但是 BeautifulSoup 对象并不是真正的 HTM L或 XML 的 tag，它没有attribute属性，name 属性是一个值为“[document]”的特殊属性。

## 4、Comment

　　Comment 一般表示文档的注释部分。

```
soup = BeautifulSoup("<b><!--This is a comment--></b>")

comment = soup.b.string

print(comment)          # This is a comment

print(type(comment))    # <class 'bs4.element.Comment'>
```

# 三、遍历 

## 1、子节点

**contents** **属性**

　　contents 属性返回所有子节点的列表，包括 NavigableString 类型节点。如果节点当中有换行符，会被当做是 NavigableString 类型节点而作为一个子节点。

　　NavigableString 类型节点没有 contents 属性，因为没有子节点。



```
soup = BeautifulSoup("""<div>
<span>test</span>
</div>
""")

element = soup.div.contents

print(element)          # ['\n', <span>test</span>, '\n']
```



**children** **属性**

　　children 属性跟 contents 属性基本一样，只不过返回的不是子节点列表，而是子节点的可迭代对象。

**descendants** **属性**

　　descendants 属性返回 tag 的所有子孙节点。

**string** **属性**

　　如果 tag 只有一个 NavigableString 类型子节点，那么这个 tag 可以使用 .string 得到子节点。

　　如果一个 tag 仅有一个子节点，那么这个 tag 也可以使用 .string 方法，输出结果与当前唯一子节点的 .string 结果相同。

　　如果 tag 包含了多个子节点，tag 就无法确定 .string 方法应该调用哪个子节点的内容, .string 的输出结果是 None。



```
soup = BeautifulSoup("""<div>
    <p><span><b>test</b></span></p>
</div>
""")

element = soup.p.string

print(element)          # test

print(type(element))    # <class 'bs4.element.NavigableString'>
```

　　特别注意，为了清楚显示，一般我们会将 html 节点换行缩进显示，而在BeautifulSoup 中会被认为是一个 NavigableString 类型子节点，导致出错。上例中，如果改成 element = soup.div.string 就会出错。

**strings** **和 stripped_strings 属性**

　　如果 tag 中包含多个字符串，可以用 strings 属性来获取。如果返回结果中要去除空行，则可以用 stripped_strings 属性。

```
soup = BeautifulSoup("""<div>
    <p>      </p>
    <p>test 1</p>
    <p>test 2</p>
</div>
""", 'html.parser')

element = soup.div.stripped_strings

print(list(element))          # ['test 1', 'test 2']
```



## 2、父节点

**parent** **属性**

　　parent 属性返回某个元素（tag、NavigableString）的父节点，文档的顶层节点的父节点是 BeautifulSoup 对象，BeautifulSoup 对象的父节点是 None。

**parents** **属性**

　　parent 属性递归得到元素的所有父辈节点，包括 BeautifulSoup 对象。

## 3、兄弟节点

**next_sibling** **和 previous_sibling**

　　next_sibling 返回后一个兄弟节点，previous_sibling 返回前一个兄弟节点。直接看个例子，注意别被换行缩进搅了局。



```
soup = BeautifulSoup("""<div>
    <p>test 1</p><b>test 2</b><h>test 3</h></div>
""", 'html.parser')

print(soup.b.next_sibling)      # <h>test 3</h>

print(soup.b.previous_sibling)  # <p>test 1</p>

print(soup.h.next_sibling)      # None
```



**next_siblings** **和 previous_siblings**

　　next_siblings 返回后面的兄弟节点

　　previous_siblings　　返回前面的兄弟节点

## 4、回退和前进

　　把html解析看成依次解析标签的一连串事件，BeautifulSoup 提供了重现解析器初始化过程的方法。

　　next_element 属性指向解析过程中下一个被解析的对象（tag 或 NavigableString）。

　　previous_element 属性指向解析过程中前一个被解析的对象。

　　另外还有next_elements 和 previous_elements 属性，不赘述了。

# 四、搜索

## 1、过滤器

　　介绍 find_all() 方法前，先介绍一下过滤器的类型，这些过滤器贯穿整个搜索的API。过滤器可以被用在tag的name中，节点的属性中，字符串中或他们的混合中。

示例使用的 html 文档如下：



```
html = """
<div>
    <p class="title"><b>The Dormouse's story</b></p>
    <p class="story">Once upon a time there were three little sisters; and their names were
    <a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
    <a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
    <a href="http://example.com/tillie" class="sister" id="link3">Tillie</a></p>
</div>
"""

soup = BeautifulSoup(html, 'html.parser')
```

**字符串**

查找所有的<b>标签

```
soup.find_all('b')  # [<b>The Dormouse's story</b>]
```

**正则表达式**

传入正则表达式作为参数，返回满足正则表达式的标签。下面例子中找出所有以b开头的标签。

```
soup.find_all(re.compile("^b"))  # [<b>The Dormouse's story</b>]
```

**列表**

传入列表参数，将返回与列表中任一元素匹配的内容。下面例子中找出所有<a>标签和<b>标签。

```
soup.find_all(["a", "b"])
```

**True**

True可以匹配任何值，下面的代码查找到所有的tag，但是不会返回字符串节点。

```
soup.find_all(True)
```

**方法**

如果没有合适过滤器，那么还可以自定义一个方法，方法只接受一个元素参数，如果这个方法返回True表示当前元素匹配被找到。下面示例返回所有包含 class 属性但不包含 id 属性的标签。

```
def has_class_but_no_id(tag):
    return tag.has_attr('class') and not tag.has_attr('id')


print(soup.find_all(has_class_but_no_id))
```

返回结果：

```
[<p class="title"><b>The Dormouse's story</b></p>, <p class="story">Once upon a time there were three little sisters; and their names were
    <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
    <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a> and
    <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a></p>]
```

这个结果乍一看不对，<a>标签含有 id 属性，其实返回的 list 中只有2个元素，都是<p>标签，<a>标签是<p>标签的子节点。

## 2、find 和 find_all

　　搜索当前 tag 的所有 tag 子节点，并判断是否符合过滤器的条件

语法：

　　find(name=None, attrs={}, recursive=True, text=None, **kwargs)

　　find_all(name=None, attrs={}, recursive=True, text=None, limit=None, **kwargs)

参数：

　　name：查找所有名字为 name 的 tag，字符串对象会被自动忽略掉。上面过滤器示例中的参数都是 name 参数。当然，其他参数中也可以使用过滤器。

　　attrs：按属性名和值查找。传入字典，key 为属性名，value 为属性值。

　　recursive：是否递归遍历所有子孙节点，默认 True。

　　text：用于搜索字符串，会找到 .string 方法与 text 参数值相符的tag，通常配合正则表达式使用。也就是说，虽然参数名是 text，但实际上搜索的是 string 属性。

　　limit：限定返回列表的最大个数。

　　kwargs：如果一个指定名字的参数不是搜索内置的参数名，搜索时会把该参数当作 tag 的属性来搜索。这里注意，如果要按 class 属性搜索，因为 class 是 python 的保留字，需要写作 class_。

 

　　Tag 的有些属性在搜索中不能作为 kwargs 参数使用，比如 html5 中的 data-* 属性。

```
data_soup = BeautifulSoup('<div data-foo="value">foo!</div>')

print(data_soup.find_all(data-foo="value"))

# SyntaxError: keyword can't be an expression
```

　　但是可以通过 attrs 参数传递：

```
data_soup = BeautifulSoup('<div data-foo="value">foo!</div>')

print(data_soup.find_all(attrs={"data-foo": "value"}))

# [<div data-foo="value">foo!</div>]
```

　　而按 class_ 查找时，只要一个CSS类名满足即可，如果写了多个CSS名称，那么顺序必须一致，而且不能跳跃。以下示例中，前三个可以查找到元素，后两个不可以。



```
css_soup = BeautifulSoup('<p class="body bold strikeout"></p>')

print(css_soup.find_all("p", class_="strikeout"))

print(css_soup.find_all("p", class_="body"))

print(css_soup.find_all("p", class_="body bold strikeout"))

# [<p class="body strikeout"></p>]

print(css_soup.find_all("p", class_="body strikeout"))

print(css_soup.find_all("p", class_="strikeout body"))

# []
```



## 3、像调用find_all()一样调用tag

　　find_all() 几乎是 BeautifulSoup 中最常用的搜索方法，所以我们定义了它的简写方法。BeautifulSoup 对象和 tag 对象可以被当作一个方法来使用，这个方法的执行结果与调用这个对象的 find_all() 方法相同，下面两行代码是等价的:

```
soup.find_all('b')

soup('b')
```

## 4、其他搜索方法

find_parents()　　　　　 返回所有祖先节点

find_parent()　　　　　　返回直接父节点

find_next_siblings()　　 返回后面所有的兄弟节点

find_next_sibling()　　 返回后面的第一个兄弟节点

find_previous_siblings() 返回前面所有的兄弟节点

find_previous_sibling()　返回前面第一个兄弟节点

find_all_next()　　　　 返回节点后所有符合条件的节点

find_next()　　　　　　 返回节点后第一个符合条件的节点

find_all_previous()　　 返回节点前所有符合条件的节点

find_previous()　　　　 返回节点前所有符合条件的节点

# 五、CSS选择器

BeautifulSoup支持大部分的CSS选择器，这里直接用代码来演示。



```
from bs4 import BeautifulSoup

 
html = """
<html>
<head><title>标题</title></head>
<body>
 <p class="title" name="dromouse"><b>标题</b></p>
 <div name="divlink">
  <p>
   <a href="http://example.com/1" class="sister" id="link1">链接1</a>
   <a href="http://example.com/2" class="sister" id="link2">链接2</a>
   <a href="http://example.com/3" class="sister" id="link3">链接3</a>
  </p>
 </div>
 <p></p>
 <div name='dv2'></div>
</body>
</html>
"""

soup = BeautifulSoup(html, 'lxml')

# 通过tag查找
print(soup.select('title'))             # [<title>标题</title>]

# 通过tag逐层查找
print(soup.select("html head title"))   # [<title>标题</title>]

# 通过class查找
print(soup.select('.sister'))
# [<a class="sister" href="http://example.com/1" id="link1">链接1</a>,
# <a class="sister" href="http://example.com/2" id="link2">链接2</a>,
# <a class="sister" href="http://example.com/3" id="link3">链接3</a>]


# 通过id查找
print(soup.select('#link1, #link2'))
# [<a class="sister" href="http://example.com/1" id="link1">链接1</a>,
# <a class="sister" href="http://example.com/2" id="link2">链接2</a>]


# 组合查找
print(soup.select('p #link1'))　　　　# [<a class="sister" href="http://example.com/1" id="link1">链接1</a>]

 
# 查找直接子标签
print(soup.select("head > title"))　 # [<title>标题</title>]

print(soup.select("p > #link1"))　　 # [<a class="sister" href="http://example.com/1" id="link1">链接1</a>]

print(soup.select("p > a:nth-of-type(2)"))　　# [<a class="sister" href="http://example.com/2" id="link2">链接2</a>]
# nth-of-type 是CSS选择器

 

# 查找兄弟节点（向后查找）
print(soup.select("#link1 ~ .sister"))
# [<a class="sister" href="http://example.com/2" id="link2">链接2</a>,
# <a class="sister" href="http://example.com/3" id="link3">链接3</a>]

print(soup.select("#link1 + .sister"))
# [<a class="sister" href="http://example.com/2" id="link2">链接2</a>]

 

# 通过属性查找
print(soup.select('a[href="http://example.com/1"]'))

# ^ 以XX开头
print(soup.select('a[href^="http://example.com/"]'))

# * 包含
print(soup.select('a[href*=".com/"]'))

# 查找包含指定属性的标签
print(soup.select('[name]'))

 

# 查找第一个元素
print(soup.select_one(".sister"))
```