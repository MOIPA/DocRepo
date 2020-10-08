title: Python报错问题解决：'ascii' codec can't encode character
author: tr
tags:
  - Python
categories:
  - 'Python '
date: 2019-04-25 10:48:00
---
### python报错问题解决：'ascii' codec can't encode character

vim /usr/lib/python2.7/site-packages/sitecustomize.py 

```python
#添加如下内容，设置编码为utf8

# encoding=utf8
import sys

reload(sys)
sys.setdefaultencoding('utf8')

```