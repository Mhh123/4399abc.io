## 1 BeautifulSoup4   CSS选择器

Beautiful Soup支持大部分的CSS选择器 [[6\]](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#id87) ,在 `Tag` 或 `BeautifulSoup` 对象的 `.select()` 方法中传入字符串参数,即可使用CSS选择器的语法找到tag:

```
soup.select("title")
# [<title>The Dormouse's story</title>]

soup.select("p nth-of-type(3)")
# [<p class="story">...</p>]
```

通过tag标签逐层查找:

```
soup.select("body a")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie"  id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("html head title")
# [<title>The Dormouse's story</title>]

```

找到某个tag标签下的直接子标签 [[6\]](https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#id87) :

```
soup.select("head > title")
# [<title>The Dormouse's story</title>]

soup.select("p > a")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie"  id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("p > a:nth-of-type(2)")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

soup.select("p > #link1")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select("body > a")
# []

```

找到兄弟节点标签:

```
soup.select("#link1 ~ .sister")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie"  id="link3">Tillie</a>]

soup.select("#link1 + .sister")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

```

通过CSS的类名查找:

```
soup.select(".sister")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("[class~=sister]")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

```

通过tag的id查找:

```
soup.select("#link1")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select("a#link2")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

```

通过是否存在某个属性来查找:

```
soup.select('a[href]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

```

通过属性的值来查找:

```
soup.select('a[href="http://example.com/elsie"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select('a[href^="http://example.com/"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select('a[href$="tillie"]')
# [<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select('a[href*=".com/el"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

```

通过语言设置来查找:

```
multilingual_markup = """
 <p lang="en">Hello</p>
 <p lang="en-us">Howdy, y'all</p>
 <p lang="en-gb">Pip-pip, old fruit</p>
 <p lang="fr">Bonjour mes amis</p>
"""
multilingual_soup = BeautifulSoup(multilingual_markup)
multilingual_soup.select('p[lang|=en]')
# [<p lang="en">Hello</p>,
#  <p lang="en-us">Howdy, y'all</p>,
#  <p lang="en-gb">Pip-pip, old fruit</p>]

```

对于熟悉CSS选择器语法的人来说这是个非常方便的方法.Beautiful Soup也支持CSS选择器API,如果你仅仅需要CSS选择器的功能,那么直接使用 `lxml` 也可以,而且速度更快,支持更多的CSS选择器语法,但Beautiful Soup整合了CSS选择器的语法和自身方便使用API.





## 2  lxml  第三方插件，可以解析 xml 和 HTML

### 2.1  安装:

    pip install lxml


### 2.2  使用：




    from lxml import etree -> 导入模块，该库常用的XML处理功能都在lxml.etree中
    # 生成tree对象
    tree = etree.HTML('xxx.html')
    print(tree)
    #<lxml.etree._ElementTree object at 0x000000000252B388>
    
    ret = tree.xpath('//div[@class="hero"]/text()')



###### 扩展1:

```python
def xpath(self, _path, namespaces=None, extensions=None, smart_strings=True, **_variables): # real signature unknown; restored from __doc__

        xpath(self, _path, namespaces=None, extensions=None, smart_strings=True, **_variables)

                XPath evaluate in context of document.

                ``namespaces`` is an optional dictionary with prefix to namespace URI
                mappings, used by XPath.  ``extensions`` defines additional extension
                functions.

                Returns a list (nodeset), or bool, float or string.

                In case of a list result, return Element for element nodes,
                string for text and attribute values.

                Note: if you are going to apply multiple XPath expressions
                against the same document, it is more efficient to use
                XPathEvaluator directly.

        pass
```


### 2.3  xpath语法:

​	xpath : 使用路径表达式来选取 XML 文档中的节点或节点集。

    安装一个谷歌浏览器xpath插件，在这里吗学习xpath解析html语法
    
    右边三个点==》更多工具==》扩展程序==》将xpath.crx拖进来安装即可
    
    启动和关闭插件：ctrl+shift+x


### 	2.3-1 常用的xpath

        《1》通过属性筛选
         input[@id="kw"]
         //div[@class="bdbri bdbriimg"]
         【注】如果这个标签的class有多个，再通过class进行筛选的时候要将所有的class全部写进来
      《2》索引和层级定位
         //div[@id="head"]/div/div[3]/a[2]
         //div[@id="head"]/div/div[@id="u1"]/a[2]
      《3》获取属性和内容
         //div[@id="u1"]/a[3]/text()   获取标签内容
         //div[@id="u1"]/a[3]/@href    获取属性
      《4》xpath函数
         contains
            属性包含  //div[@id="u1"]/a[contains(@href,"www")]
            内容包含  //div[@id="u1"]/a[contains(text(),"闻")]
         starts-with
            属性开头  //div[@id="u1"]/a[starts-with(@class,"mn")]
            内容开头  //div[@id="u1"]/a[starts-with(text(),"地")]
      谷歌浏览器自动生成xpath和选择器
         //*[@id="u1"]/a[2]
         #u1 > a:nth-child(2)
    (2)在代码中通过xpath获取到想要的节点
      tree = etree.HTML(content)
      步骤：读取文件内容，生成一个对象，然后通过对象的xpath方法去获得想要的节点
      ，文件可以是本地的，也可以是网络的，学习使用本地的
    【注】xpath返回的是列表
      【注】如果标签里还有还有标签，分情况:
        /text()     :获取本标签这一层里面所有内容
        //text()    :获取本标签向下递归所有层里面所有内容

######   扩展2:


    
          etree.HTML()
            """
            HTML(text, parser=None, base_url=None)
    
                    Parses an HTML document from a string constant.  Returns the root
                    node (or the result returned by a parser target).  This function
                    can be used to embed "HTML literals" in Python code.
            """
            etree.parse()
            """
                parse(source, parser=None, base_url=None)
    
                    Return an ElementTree object loaded with source elements.  If no parser
                    is provided as second argument, the default parser is used.
                 The ``source`` can be any of the following:
    
                        - a file name/path
                        - a file object
                        - a file-like object
                        - a URL using the HTTP or FTP protocol
            """
    
      

### 2.3-2 xpath实例

    图片下载
        懒加载技术:
            一个图片网页，上面是100个图片<img src=xxx>，
            只有呈现在可视区内的图片需要加载过来，其他的图片等滚动滚动条，
            出现在可视区的时候再去动态加载
      如何实现：
         图片里面写的都是src2，等到出现在可视区内，通过js动态将src2修改为src即可
        <img src2=xxx>
        呈现形式:
            src2  data-src  data-original  class="lazy"
## 3  json数据解析

    json数据是什么？
    (1)  数据在键值对中
    (2)  数据以逗号分隔
    (3)  {}  保存对象
    (4)  []  保存数组
    爬取数据，ajax接口，返回时json格式数据
    《1》  使用python自带的模块
            import json
            json.dumps()    将python对象转化为json格式字符串
            json.loads()    将json格式字符串-->python对象
    
            string = json.dumps(lt, ensure_ascii=False)
    
            [{"name": "\u738b\u9759",...}]
            #加一个ensure_ascii=False可以将乱码转换可读
            [{"name": "王静", ...}]
    《2》jsonpath(了解)
        pip install jsonpath
        xpath jsonpath
        /   $ 根元素
        .   @  当前元素
        /   .[]  层级分隔符
        //  ..   任意位置的查找
        【注】下标xpath中从1开始。jsonpath中从0开始

Jsonpath博客:    http://blog.csdn.net/luxideyao/article/details/77802389

