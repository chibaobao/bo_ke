---
title: python总结
date: 2016-11-11 23:33:56
tags: python
categories: 编程学习
---

# python

## 函数

主函数经常这样写，这样imort该文件，主函数就不会被执行了
```python
if __name__=="__main__"：
	主函数
```
### 常用函数

```python
range(10)  #生成列表0到9
list.append("aa") #向list中尾插aa
dict.update("aa":"bb") #对dict进行更新key为aa的value，有更新，无则插入
str = "&".join(list)   #["11","22","33"]--->"11&22&33"
list2=str.split("&",3) #"11&22&33"--->["11","22","33"],切割成3个元素
fd = os.popen('ls') #执行ls命令并将结果返回文件描述符fd中
os.system('rm aa') #执行删除aa成功返回0
fd.readline() #从文件中读取一行
time.asctime(time.localtime(time.time()))  获取当前时间
```

### 示例

####  ["11","22","33"]--->"11&22&33"--->["11","22","33"]

```python
list = ["11","22","33"]

str = "&".join(list)
#str="11&22&33"

list2=str.split("&",3)
#list2={"11","22","33"}
```

#### ["aa=11","bb"=22,"cc"=33]---->取出aa的value：11
```
list = ["aa=11","bb"=22,"cc"=33]
for que in list:
	key_value = que.split("=",2)
	if(key_value[0] == "aa"):
		print key_value[1]
```



#### 执行ps并拿到相应id
```python
import os

#执行命令
output = os.popen('ps aux|grep ssh')
mylist =[]

while  True:
	#从结果中读一行内容
	line = output.readline()
	#如果不是空行
	if (line != ""):
		#将该行追加加入到list中
		mylist.append(line)
	else:
		break
#读取第三行，并切割为list
the_line = mylist[2].split()
#读取第三行第一列（pid）
the_word = the_line[1]
print the_word
comm = "kill "+first_word
#执行kill成功返回0
os.system(comm)
```

#### 生成随机数(激活码格式)

```
import string, random

#激活码中的字符和数字,letters:所有字母a-z和A-Z, digits:0-9
field = string.letters + string.digits

#获得四个字母和数字的随机组合
def getRandom():
    return "".join(random.sample(field,4))

#生成的每个激活码中有几组
def concatenate(group):
    return "-".join([getRandom() for i in range(group)]) 
#["dsadas", "eddasads", "ewqewwq"]
#dsadas-eddasads-ewqewwq"
'''
    for i in range(group):
            getRandom()
'''

#生成n组激活码
def generate(n): 
    return [concatenate(4) for i in range(n)]

if __name__ == '__main__':
    print generate(2)

```

## 值拷贝与地址拷贝
通过list实现

```
def func(mylist):
    print mylist[0]
    mylist[0] =333


if __name__ == "__main__":
    mylist=[100]
    print mylist
    func(mylist)
    print mylist
```

## 变参
```python
def fun1(a,*args):
    print a
    #out:1
    print args
    #out:(2,3,4)
def fun2(a,**args):
    print a
    #out:111
    print args
    #out:{"name"="cc","age"=18}

fun1(1,2,3,4)
fun2(111,name="cc",age=18)
```

## 闭包

- 语法：其中内部函数向使用外部函数的变量，外部函数要定义为list类型
- 分析：外部函数相当于类，内部函数相当于成员 函数，内部函数使用的外部函数的变量相当于类的成员变量

```python
#coding:utf-8


#外部函数
def counter():
    count = [0]
    #内部函数
    def incr():
        count[0] = count[0] + 1
        return count[0]

    return incr


if __name__=="__main__":
#生成一个内部函数，（拿到内部函数）
    c1 = counter()    
#调用外部函数中的内部函数
    print c1()
    print c1()

#生成一个内部函数
    c2 =counter()
    print c2()
    print c2()

```

## 装饰器

语法：装饰器必须用闭包实现，装饰器函数定义前先@装饰器
分析：调用装饰器函数，相当于先调用装饰器包裹函数函数体，再调用装饰器函数
```
#定义一个装饰器函数
def doc_func(func):

    #包裹函数(闭包)，固定写法
    def warpfunc():
        #做一些额外的事情
        print "%s called" %(func.__name__) #要完成的业务
        func()
        
    return warpfunc

@
doc_func
def foo():
    print "hello"

@doc_func  #定义其装饰器为doc_fun
def bar():
    print "nihao"


if __name__== "__main__":
    foo()

    bar()

```

## 类

语法：每个函数都要有self变量，相当于c++的this指针
```python
class Student:
    '''学生类''' 
    
	sex = "man"
	#相当于c++中类的静态成员变量

    def __init__(self, name, age):
        '''构造函数init'''
        print "__init__  ..."
        self.name = name
        #如果self.__name = name，则self.__name表示私有变量，外部不可访问
        self.age = age

        #python中的成员方法 一定要有个形参 self，表示调用该方法的对象本身

    def showMe(self):
        '''普通成员方法'''
        print "showMe()..."
        print self.name
        print self.age

	@classmethod   #定义为类方法（相当于c++中的静态成员方法）
    def showme2(cls):
        #类变量
        print cls.sex
    def __del__(self):
        '''析构函数'''
        print "__del__..."


if __name__  == "__main__":
    s1 = Student("zhang3", 18)
    s1.showMe()

    s2 = Student("li4", 20)
    s2.showMe()

	#python调用静态成员方法
    Student.showme2()
```

## 继承

```python
#coding:utf-8


class Person:
    '''人类'''
    
    def __init__(self, name):
        '''人类的构造函数'''
        print "Person __init___"
        self.name = name

    def show(self):
        '''人类的show()'''
        print "Person showme()"       
        print self.name


class Student(Person):
    '''学生类'''

    def __init__(self, name, age):
        #c++中 如果子类继承父类，调用子类的构造，会自动调用父类构造o
        #python中并不是这样，需要手动调用父类构造函数，来初始化继承过来的变量
        Person.__init__(self, name)
        print "Student __init___"
        self.age = age

    def show(self):
        print "Student showme()"       
        print self.name
        print self.age


if __name__=="__main__":
    s1 = Student("zhang3", 18)
    s1.show()

    p1 = Person("li4")
    p1.show()
```

## 文件操作

复制一个文件
```python
#coding:utf-8

fp = open("aa.txt","r")
ch = fp.read()
fp.close()

fp = open("bb.txt","w")
fp.write(ch)
fp.close()
```

## struct相关

```python
#coding:utf-8


import struct

'''写二进制
fp = open("./file/binary.dat", "w")
str = "itcast"
str_len = len(str)

cmd= str(str_len)+"sid"

fp.write(struct.pack(cmd, "itcast", 100, 3.14))

fp.close()
'''


'''读二进制
'''
fp = open("./file/binary.dat", "r")

#6s:表示一个字符串string长为6,i表示int,d表示double
(name, id, score) = struct.unpack("6sid" ,fp.read() )

print name
print id
print score

fp.close()
```

## 图片处理

向图片上写字
```
#coding: utf-8

import Image, ImageFont, ImageDraw

myPath = "/home/itcast/ace/media/"
fontPath = "/home/itcast/ace/media/"

inputFile = "ace.jpg"

outputFile = "output.jpg"

#打开图片
im = Image.open(myPath + inputFile)
draw = ImageDraw.Draw(im)

#根据图片大小确定字体大小
fontsize = min(im.size)/4

#加文字
myfont = ImageFont.truetype(fontPath + 'simhei.ttf', fontsize)

draw.text((im.size[0]-fontsize, 0), '8', font = myfont, fill = (256,0,0))

im.save(myPath + outputFile,"jpeg")

```

## 系统命令调用和文件处理（统计当前代码行数）

```
#coding: utf-8
import os, re

# 代码所在目录
FILE_PATH = './'

def analyze_code(codefilesource):
    '''
        打开一个py文件，统计其中的代码行数，包括空行和注释
        返回含该文件总行数，注释行数，空行数的列表
    '''
    total_line = 0
    comment_line = 0
    blank_line = 0

    with open(codefilesource) as f:
        lines = f.readlines()#  ['import aa', 'if main', '']
        total_line = len(lines)
        line_index = 0

        # 遍历每一行
        while line_index < total_line:
            line = lines[line_index]
            # 检查是否为注释
            if line.startswith("#"):
                comment_line += 1
            # 检查是否为空行
            elif line == "\n":
                blank_line += 1

            line_index += 1

    print "在%s中：" % codefilesource
    print "代码行数：", total_line
    print "注释行数：", comment_line, "占%0.2f%%" % (comment_line*100.0/total_line)
    print "空行数：  ", blank_line, "占%0.2f%%" % (blank_line*100.0/total_line)

    return [total_line, comment_line, blank_line]


def run(FILE_PATH):
    # 切换到code所在目录
    os.chdir(FILE_PATH)
    # 遍历该目录下的py文件
    total_lines = 0
    total_comment_lines = 0
    total_blank_lines = 0

    for i in os.listdir(os.getcwd()):
        if os.path.splitext(i)[1] == '.py':
            line = analyze_code(i)
            total_lines, total_comment_lines, total_blank_lines = total_lines + line[0], total_comment_lines + line[1], total_blank_lines + line[2]

    print "总代码行数：", total_lines
    print "总注释行数：", total_comment_lines, "占%0.2f%%" % (total_comment_lines*100.0/total_lines)
    print "总空行数：  ", total_blank_lines, "占%0.2f%%" % (total_blank_lines*100.0/total_lines)


if __name__ == '__main__':
    run(FILE_PATH)

```


## 生成验证码图片

```
#coding: utf-8

import Image, ImageDraw, ImageFont, ImageFilter
import string, random

fontPath = "/home/itcast/ace/media/"

# 获得随机四个字母
def getRandomChar():
    return [random.choice(string.letters) for _ in range(4)]

# 获得颜色
def getRandomColor():
    return (random.randint(30, 100), random.randint(30, 100), random.randint(30, 100))

# 获得验证码图片
def getCodePiture():
    width = 240
    height = 60

    # 创建画布
    image = Image.new('RGB', (width, height), (180,180,180))
    font = ImageFont.truetype(fontPath + 'simhei.ttf', 80)
    draw = ImageDraw.Draw(image)

    # 创建验证码对象
    code = getRandomChar()#code-> [x,A,y,U] 

    # 把验证码放到画布上
    for t in range(4):
        draw.text((60 * t + 10, 0), code[t], font=font, fill=getRandomColor())

    # 填充噪点
    for _ in range(random.randint(1500,3000)):
        draw.point((random.randint(0,width), random.randint(0,height)), fill=getRandomColor())

    # 模糊处理
#image = image.filter(ImageFilter.BLUR)

    # 保存名字为验证码的图片
    #code = [x,y, U,a] --> xyUa.jpg
    image.save("".join(code) + '.jpg', 'jpeg');


if __name__ == '__main__':
    getCodePiture()
```

## socket

```
#coding:utf-8


from socket import *

myhost="127.0.0.1"
myport= 8082

if __name__=="__main__":

    sock = socket(AF_INET, SOCK_STREAM)

    sock.bind((myhost, myport))

    sock.listen(128)

    while True:
        cfd, address=sock.accept()
        
        print "remote addr " + address[0]
        print "remote port " + str(address[1])

        while True:
            data = cfd.recv(1024)
            if not data:
                break 
            cfd.send("echo" + data)

        cfd.close()

```