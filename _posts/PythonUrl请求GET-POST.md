title: PythonUrl请求GET+POST
author: tr
tags:
  - Python
categories:
  - Python
date: 2019-04-23 10:18:00
---
### python中的GET+POST

公司要求调用后台api，不能直接插入数据库。

首先import包
import urlib，urllib2
import requests

<!--more-->

#### GET

```python
  1 import urllib,urllib2
  2 
  3 def getToken(url,imgName):
  4    textmod={'name':imgName}
  5    textmod = urllib.urlencode(textmod)
  6    # start get request
  7    req = urllib2.Request(url='%s%s%s' % (url,'?',textmod))
  8    print(req.__dict__)
  9    res = urllib2.urlopen(req)
 10    res=res.read()
 11    print(res)
 12 
 13 if __name__=='__main__':
 14     imgName = 'test'
 15     url='http://192.168.0.100:8066/api/v1/oss/token/file/question'
 16     getToken(url,imgName)
~                               

```

#### Post请求

```python

 def pushData(json_data, url):
      headers = {'content-type': 'application/json', 'X-UserId': '2'}
      response = requests.post(url, data=json_data, headers=headers)
      print
      json_data


```