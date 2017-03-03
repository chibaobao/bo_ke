---
title: python爬虫基础
date: 2017-03-03 15:54:18
tags: 
    - python
    - 爬虫
categories: 编程学习
---
# python爬虫基础

[参考文档:Python爬虫学习系列教程](http://cuiqingcai.com/1052.html)

要注意的问题：
- 注意请求是http还是https

## post请求与get请求

### get请求

```python
import urllib2
 
values={}
values['username'] = "xxxqq.com"
values['password']="XXXX"
data = urllib.urlencode(values) 
url = "http://passport.csdn.net/account/login"
geturl = url + "?"+data
request = urllib2.Request(geturl)
response = urllib2.urlopen(request)
print response.read()
```

### post请求
```python
port urllib
import urllib2
 
values = {"username":"1016903103@qq.com","password":"XXXX"}
data = urllib.urlencode(values) 
url = "https://passport.csdn.net/account/login?from=http://my.csdn.net/my/mycsdn"
request = urllib2.Request(url,data)
response = urllib2.urlopen(request)
print response.read()
```

## 抓取网页并保存网页中的图片

- urllib2.Request(url,headers = ``headers``)
- urllib2.urlopen(requset)
```python
#coding=utf-8
import urllib
import urllib2
import  re
import  requests

def gethtml(url):
#    page = urllib.urlopen(url)
    user_agent = "Mozilla / 5.0(Macintosh; Intel Mac OS X 10_12_2) AppleWebKit/537.36(KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36"
    headers = {'User-Agent': user_agent}
    requset = urllib2.Request(url,headers = headers)
    try:
        html = urllib2.urlopen(requset)
    except urllib2.URLError, e:
        print "error"
        #e.reason错误原因e.code错误码（eg：403）
        #return e.reason
        e.code
#html是个对象，html.geturl()真实url若被重定向显示重定向之后的，html.info()显示网页信息
    return html.read()

def getimage(html):
    reg = r'src="(.+?\.jpg)" pic_ext'
    imgre = re.compile(reg)
    imglist = re.findall(imgre,html)
    x=1
    for the_url in imglist:
        #保存图片
        urllib.urlretrieve(the_url,'%s.jpg' % x)
        x = x + 1

html = gethtml("https://www.zhih.com/question/51235971/answer/139605122")
print html
```

## cookie的使用

### 获取Cookie

```python
import urllib2
import cookielib
#声明一个CookieJar对象实例来保存cookie
cookie = cookielib.CookieJar()
#利用urllib2库的HTTPCookieProcessor对象来创建cookie处理器
handler=urllib2.HTTPCookieProcessor(cookie)
#通过handler来构建opener
opener = urllib2.build_opener(handler)
#此处的open方法同urllib2的urlopen方法，也可以传入request
response = opener.open('http://www.baidu.com')
for item in cookie:
    print 'Name = '+item.name
    print 'Value = '+item.value
```

### 保存Cookie到文件

```python
import cookielib
import urllib2

#设置保存cookie的文件，同级目录下的cookie.txt
filename = 'cookie.txt'
#声明一个MozillaCookieJar对象实例来保存cookie，之后写入文件
cookie = cookielib.MozillaCookieJar(filename)
#利用urllib2库的HTTPCookieProcessor对象来创建cookie处理器
handler = urllib2.HTTPCookieProcessor(cookie)
#通过handler来构建opener
opener = urllib2.build_opener(handler)
#创建一个请求，原理同urllib2的urlopen
response = opener.open("http://www.baidu.com")
#保存cookie到文件
cookie.save(ignore_discard=True, ignore_expires=True)
```

### 从文件中获取Cookie并访问

```python
import cookielib
import urllib2

#创建MozillaCookieJar实例对象
cookie = cookielib.MozillaCookieJar()
#从文件中读取cookie内容到变量
cookie.load('cookie.txt', ignore_discard=True, ignore_expires=True)
#创建请求的request
req = urllib2.Request("http://www.baidu.com")
#利用urllib2的build_opener方法创建一个opener
opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cookie))
response = opener.open(req)
print response.read()
```

### 利用cookie模拟网站登录(学校教务系统)
```python
#coding=utf-8
import urllib
import urllib2
import cookielib
import gzip
import StringIO


def decode_gzip(f):
    """
    如果网页抓取乱码，并采取gzip格式压缩，进行解压
    f = urllib2.urlopen(requset)
    """
    headers = f.info()
    html = f.read()
    if ('Content-Encoding' in headers and headers['Content-Encoding']) or (
            'content-encoding' in headers and headers['content-encoding']):
        data = StringIO.StringIO(html)
        gz = gzip.GzipFile(fileobj=data)
        html = gz.read()
        gz.close()
        return html

filename = 'cookie.txt'
#声明一个MozillaCookieJar对象实例来保存cookie，之后写入文件
cookie = cookielib.MozillaCookieJar(filename)
opener = urllib2.build_opener(urllib2.HTTPCookieProcessor(cookie))
postdata = urllib.urlencode({
            'yhm':'13122134xxxx',
            'mm':'xxxx',
            'yzm':''
		})
#登录教务系统的URL
loginUrl = 'http://jwxt.tstc.edu.cn/xtgl/login_login.html'
#模拟登录，并把cookie保存到变量
f = opener.open(loginUrl,postdata)
print decode_gzip(f)

#保存cookie到cookie.txt中
cookie.save(ignore_discard=True, ignore_expires=True)
#利用cookie请求访问另一个网址，此网址是成绩查询网址
gradeUrl = 'http://jwxt.tstc.edu.cn/xtgl/index_initMenu.html?t='
#请求访问成绩查询网址
f = opener.open(gradeUrl)
print decode_gzip(f)
```

## 处理乱码

### 处理gzip压缩导致的乱码

```python
import gzip
import StringIO
def decode_gzip(f):
    """
    如果网页抓取乱码，并采取gzip格式压缩，进行解压
    f = urllib2.urlopen(requset)
    """
    headers = f.info()
    html = f.read()
    if ('Content-Encoding' in headers and headers['Content-Encoding']) or (
            'content-encoding' in headers and headers['content-encoding']):
        data = StringIO.StringIO(html)
        gz = gzip.GzipFile(fileobj=data)
        html = gz.read()
        gz.close()
        return html
```

## 实战：抓取糗事百科第一页

```python
#coding=utf-8
import urllib
import urllib2
import re
def gethtml(url):
    user_agent = "Mozilla / 5.0(Macintosh; Intel Mac OS X 10_12_2) AppleWebKit/537.36(KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36"
    headers = {'User-Agent': user_agent}
    requset = urllib2.Request(url,headers = headers)
    try:
        html = urllib2.urlopen(requset)
    except urllib2.HTTPError, e:
        return e.code
    except urllib2.URLError, e:
        return e.reason
    return html.read()

def gettext(html):
    reg = r'<div class="content">.*?<span>(.*?)</span>.*?</div>'
    textre = re.compile(reg,re.S)
    textlist = re.findall(textre,html)
    i = 0
    for text in textlist:
        textlist[i] = text.replace('<br/>','\n')
        #print text+'\n'
        i= i+1
    return textlist

url = "http://www.qiushibaike.com/"
page = gethtml(url)
textlist = gettext(page)
for text in textlist:
    print text+'\n'
```

# requests的使用

[参考文档](http://cuiqingcai.com/2556.html)
## get请求

```python
import requests

payload = {'key1': 'value1', 'key2': 'value2'}
r = requests.get("http://httpbin.org/get", params=payload)
print r.url
```

## 添加headers

```python
import requests

payload = {'key1': 'value1', 'key2': 'value2'}
headers = {'content-type': 'application/json'}
r = requests.get("http://httpbin.org/get", params=payload, headers=headers)
print r.url
```

## post请求

普通post请求
```python
import json
import requests

url = 'http://httpbin.org/post'
payload = {'some': 'data'}
r = requests.post(url, data=json.dumps(payload))
print r.text
```

post上传文件
```python
import requests

url = 'http://httpbin.org/post'
files = {'file': open('test.txt', 'rb')}
r = requests.post(url, files=files)
print r.text
```

## cookies

获取和使用
```python
import requests

url = 'http://example.com'
r = requests.get(url)
cookies =  r.cookies
r = requests.get(url, cookies=cookies)
print r.text
```

## https

https访问github

```python
import requests

r = requests.get('https://github.com', verify=True)
print r.text
```

12306证书不合法，设置verify=False
```python
import requests

r = requests.get('https://kyfw.12306.cn/otn/', verify=False)
print r.text
```

##  Beautiful Soup

[参考链接](http://cuiqingcai.com/1319.html)
- 安装
	- pip install beautifulsoup4
	- pip install lxml
	- pip install html5lib

- soup.title获取title标签
- print soup.p.name获取p标签的标签名：p
- soup.p.attrs是p标签的属性
- soup.p.string是p标签里面的内容
- type(soup2.a.string)!=bs4.element.Comment判断a表片内容是否为注释
```python
#coding=utf-8
from bs4 import BeautifulSoup
import bs4
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>
"""

html2="""
<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>
"""

#正式代码部分
soup = BeautifulSoup(html,"html5lib")
soup2 = BeautifulSoup(html2,"html5lib")

#print soup.prettify()
#格式化输出

print soup.title
#<title>The Dormouse's story</title>

print soup.head
#<head><title>The Dormouse's story</title></head>

print soup.a
#<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>

print soup.p
#<p class="title" name="dromouse"><b>The Dormouse's story</b></p>

#name与attrs属性
print soup.name
print soup.p.name
print soup.p.attrs #得到的是一个字典{u'class': [u'title'], u'name': u'dromouse'}
print soup.p.string #获取标签里面的内容

#<!-- Elsie -->是注释，.string 类型为bs4.element.Comment
if type(soup2.a.string)!=bs4.element.Comment:
    print soup2.a.string
```

###  .string 和 .strings 以及 .stripped_strings 属性

- .string输出标签内容
- .string所有标签内容(list类型)
- .stripped_strings与.strings功能类似但是去除换行符tab键等

### 搜索功能

#### find_all

1. 查找标签内容eg:. ``find_all('a')`` 寻找所有被```<a></a>```包裹的内容
2. 用正则表达式查找eg: ``find_all(re.compile("^b"))`` 查找以b开头的
3. 传入列表 eg: ``soup.find_all(["a", "b"])`` 查找被a或b标签包裹的内容
4. soup.find_all(id='link2')查找id属性等于link2的
	```# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]```

5. soup.find_all(href=re.compile("elsie"))查找href属性包括elsie的
	```# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]```

6. soup.find_all("a", class_="sister")查找a标签且class属性是sister的
	```# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]```

#### select

1. 通过标签名查找：```soup.select('title') ``` 查找title标签
2. 通过类名查找：soup.select('.sister')查找包含 class等于``"sister"`` 的标签
	```#[<a class="sister" href="http://example.com/elsie" id="link1"><!-- Elsie --></a>]```
3. 通过id查找：``` soup.select('#link1')``` 查找id等于link1的
4. 通过标签名+id查找：```soup.select('p #link1')``` p标签且id等于link1
5. 查找子标签```soup.select("head > title")``` 父标签head子标签title
6. 通过属性查找 ```soup.select('a[class="sister"]')``` a标签且class=sister
 - 组合查找：```soup.select('p a[class="sister"]')``` 父标签为p的a标签，且class=sister


## Scrapy使用

[爬虫框架Scrapy示例入门教程](http://www.cnblogs.com/wuxl360/p/5567631.html)
[scrapy抓取糗事百科](http://blog.csdn.net/wangande105108/article/details/50792286)

1. 安装
[安装参考文档]（http://cuiqingcai.com/912.html）
```
pip install lxml
pip install Scrapy
```

2. 使用
	1. 新建一个工程 ：```scrapy startproject projectname ```
	
	- scrapy.cfg：项目的配置文件
	- projectname/：项目的Python模块，将会从这里引用代码
	- projectname/items.py：项目的items文件
	- projectname/pipelines.py：项目的pipelines文件
	- projectname/settings.py：项目的设置文件(设置heade等信息)
	- projectname/spiders/：存储爬虫的目录

	2. 编码

	>response.url获取当前网页的url
	>response.body获取当前的网页（文本格式）

	为了结构清晰已下示例将网页处理的函数定义到了类外

	3. 运行```scrapy crawl 代码中的爬虫名```
	

```python
#coding=utf-8
from scrapy.spider import Spider
from scrapy.http import Request
from p1.items import QiubaiItem
import re
import time

#一下函数为了结构清晰定义到了类外

#从网页中分析出需要的部分
def gettext(html):
    reg = r'<div class="content">.*?<span>(.*?)</span>.*?</div>'
    textre = re.compile(reg,re.S)
    textlist = re.findall(textre,html)
    for text in textlist:
        ret = text.replace('<br/>','\n')
     #   print type(ret)
     #   ret =ret.encode('utf-8')
        yield ret+"\n"
    #
def geturl(html):
    res = html.xpath('//ul[@class="pagination"]/li')
    textlist = res[-1].xpath('a/@href').extract()
    url= textlist[0].encode("utf-8")
    url = url.split("/", 2)[2]#.split("?",1)[0]
    return url

class DmozSpider(Spider):#类名自定义
    name = "dmoz" #爬虫名字
    allowed_domains = ["qiushibaike.com"] #搜索的域名范围
    start_urls = ["http://www.qiushibaike.com/hot/"]#起始url


    def parse(self, response):
        item = QiubaiItem()
        urllist = response.url.split('/')
        if len(urllist) == 7:
            file_name = urllist[4]+'_'+urllist[5]
        else:
            file_name = "page_1"
        print "file name:："+file_name
        output = open(file_name,"w")
        url2 = geturl(response)
        newurl = self.start_urls[0]+url2
        print "new url"+newurl
        for text in gettext(response.body):
            output.write(text+"\n")
        output.close()
        print "**"*30
        time.sleep(1)
        if url2 == '':
            print "-------------over--------------"
            return
        else:
            yield Request(newurl,callback=self.parse)
```