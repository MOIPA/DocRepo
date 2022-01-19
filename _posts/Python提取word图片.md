title: Python提取word图片
author: tr
tags:
  - Python
categories:
  - Python
date: 2019-04-24 17:29:00
---
### Python提取word图片

<!--more-->

#### 代码：

```python

import zipfile
import os
import shutil

def word2pic(path):
    zip_path = path.replace('docx','zip')
    shutil.copy(path,zip_path)
    f =  zipfile.ZipFile(zip_path,'r')
    if os.path.exists('./word_pic_save') == True:
        shutil.rmtree('./word_pic_save')
    os.mkdir('./word_pic_save')
    os.mkdir('./word_pic_save/tmp')
    #os.chdir('./word_pic_save')
    #print os.getcwd()

    # do extract
    for efile in f.namelist():
        f.extract(efile,'./word_pic_save/tmp')
    f.close()
    pic = os.listdir('./word_pic_save/tmp/word/media')
    for i in pic:
        shutil.copy(os.path.join('./word_pic_save/tmp/word/media',i),'./word_pic_save')
    shutil.rmtree('./word_pic_save/tmp')

if __name__=='__main__':
    word2pic('cp1.docx')

```