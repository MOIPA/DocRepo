title: Python解析Json
author: tr
tags:
  - Python
  - Json
categories:
  - Python
date: 2019-04-23 10:41:00
---
### JSON问题

目标解析对象数组的对象如下：
```python
 24 class Question:
 25 
 26     def __init__(self, qid, item, options, answer, point, analysis):
 27         self.qid = qid
 28         self.item = item
 29         self.option = Option(options)
 30         self.answer = answer
 31         self.point = point
 32         self.analysis = analysis
 33 
 34     def display(self):
 35         print
 36         str(self.qid) + ":" + str(self.item) + str(self.answer) + str(self.point) + str(self.analysis)
 37         self.option.display()
 38 


```
<!--more-->

#### 对象转JSON

import json

```python
# 这是复杂对象的json转换，因为对象里面还有对象
# quesOut是Question的一个实例
json_out = json.dumps(quesOut, default=lambda o: o.__dict__, sort_keys=True, indent=4, ensure_ascii=False)
             print json_out

```

#### JSON转对象

本地打开Json文件

```python
 24 def main(fileName):
 25     with open(fileName,'r') as f:
 26         load_json = json.load(f)
 27         question_size = load_json['size']
 28         for i in range(question_size):
 29             #print load_json['res']['questions'][i]
 30             parseOneJson(load_json['res']['questions'][i])
 31 
 32         print load_json['res']['questions'][1]['questionType']


```
