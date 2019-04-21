title: python解析word文档
author: tr
tags:
  - python
categories:
  - python
date: 2019-04-15 17:19:00
---
### 今天公司新需求，解析word习题集

使用python-docx这个库解析

<!--more-->

#### dockx的官方API描述

官方的比那些blog好多了：https://python-docx.readthedocs.io/en/latest/dev/analysis/features/shapes/shapes-inline.html

#### 使用ElementTree解析word的树
代码：

```python

import docx
import json

import xml.etree.cElementTree as ET


def parsePic2(questionArray, bookNameWord):
    fs = docx.Document(bookNameWord)
    # parse pic
    proxy = []
    for p in fs.paragraphs:
        proxy.append(p._element.xml)
    rIds = []
    for p in proxy:
     # 一段一个根树
        root = ET.fromstring(p)
     # 获得<w:r>树，所有的<w:pict>树均是<w:r>树的子树
        pictr_str = "{http://schemas.openxmlformats.org/wordprocessingml/2006/main}r"
        pictrs = root.findall(pictr_str)
        image_str = "*/{urn:schemas-microsoft-com:vml}shape/{urn:schemas-microsoft-com:vml}imagedata"
        for pictr in pictrs:
            # 获得所有<v:imagedata>标签
            pict = pictr.findall(image_str)
            if len(pict) > 0:
                rIds.append(pict[0].attrib['{http://schemas.openxmlformats.org/officeDocument/2006/relationships}id'])

def parsePic(questionArray,bookNameWord):
    fs = docx.Document(bookNameWord)
    for shap in fs.inline_shapes:
        tree = shap._inline.xml
        root = ET.fromstring(tree)
        root.findall('')
        #print (json.dumps(shap.__dict__, default=lambda o: o.__dict__))
    #for line in fs.paragraphs:
    #    print(line.text)

questionArray = []
parsePic(questionArray, './test.docx')
```