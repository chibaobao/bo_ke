---
title: web.py总结
date: 2017-03-03 15:48:20
tags: python
categories: 编程学习
---
# web.py使用

## post和get请求

- 主要部分
	1. urls
	2. 与urls中映射相同的类名，定义GET或POST函数相应相关请求
	3. app = web.application(urls, globals())
	4. app.run()

- web.input()
	- 表示url中的参数(get,post)
	- eg:http://127.0.0.1:8080/?name=aa&id=123
- web.data() 
	- 表示上传的数据
	- eg:http://127.0.0.1:8080/?name=aa&id=哈哈
		- body内容：{ "name": "aa", "id": "bb"}
- json.dumps(dict)
	- 将字典转换为json

```python
#coding=utf-8
import web
import json
urls = (
    '/', 'index'
)

class index:
    def GET(self):
#	result = web.input()
#如果url中没有参数name，id下面代码不会报错，而是result.id为空
		result = web.input(name='',id='') 
		ret = {}
		ret["name"] = result.name
		ret["id"] = result["id"]
#        return json.dumps(ret)
		return json.dumps(ret , ensure_ascii = False) #参数二表示编码为utf-8
    def POST(self):
		result = web.input()
		print "input:",result
		print "data:",web.data()
        return result

if __name__ == "__main__":
    app = web.application(urls, globals())
    app.run()
```

## 连接mysql

- 搭建环境（mac）
	1.  安装mysql
	2.  brew install mysql-connector-c
	3.  sudo pip install MySQL-python

- 查询操作示例

```
import web
g_gdb = web.database(dbn='mysql', host="127.0.0.1" , port=3306, user="root", pw="passwd", db="test" , pooling = False)
results = g_gdb.query('select * from contactinfo where userid=$m_uid',vars={'m_uid':111});
for result in results :
	userid = result.userid
	name = result.name
	phone = result.phone
	address = result.address
dictInfo = {}
dictInfo['userid'] = str(userid)
dictInfo['name'] = str(name)
dictInfo['phone'] = str(phone)
dictInfo['address'] = str(address)
print dictInfo
```

