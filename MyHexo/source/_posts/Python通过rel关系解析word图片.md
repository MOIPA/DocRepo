title: Python通过rel关系解析word图片
author: tr
tags:
  - Python
categories:
  - Python
date: 2019-05-16 15:29:00
---
### python解析图片

选用工具：python-docx，elementtree

最终目标：读取word文档，把文档里的图片和wmf转为```<img src=""/>```或者```<word-pic>url</word-pic>```

###### 思路：

首先把所有的图片一次性提取出来，具体做法为将word名换为zip，解压缩将media包导出，但是这样存在一个问题：如何知道每个图片对应在word中的哪个位置。

第一种解决办法：由于图片是根据word中出现顺序排序的，所以扫描word文件，从0计数，将每出现的一个图片替换为计数。这样也存在问题：重复图片导致计数多与实际图片。wmf文件，就是word中每行中出现的矢量图，比如n不是n，而是wmf图片n。

<!--more-->

第二种思路：将每段的word解析为xml文档，使用docx，其实这个底层原理也是通过映射每段与document.xml实现的。

解压缩word为zip时，可以看到document.xml文件，这才是word真正的样子，docx可以把每段的xml展示出来，再通过elementtree解析xml树，把每段变为：文本+图片文件名字

解析xml树代码如下：

```python

import docx
import xml.etree.cElementTree as ET
import sys
import io
import re
sys.stdout=io.TextIOWrapper(sys.stdout.buffer,encoding='utf8')

fs = docx.Document('computer_science.docx')

lineCounts = 100

for line in fs.paragraphs:
   if lineCounts <= 0:
      exit()
   root = ET.fromstring(line.paragraph_format.element.xml)
   outStr = ''
   for ele in root.iter():
      # print(ele.tag)
      # continue
      if(ele.tag=="{http://schemas.openxmlformats.org/wordprocessingml/2006/main}t"):
         outStr += ele.text
      if(ele.tag == "{urn:schemas-microsoft-com:vml}shape"):
         outStr += "<word-pic>"+ele.get('id')+"</word-pic>"
   print(outStr)
   outStr = ''

   print("*****")
   lineCounts -= 1


```

解析的xml文档里放的是shapeid，就是直接对应media文件夹的图片，但是！！，shapeid有的时候不是“图片 2” 而是”_x0000...“，这就可能导致错误。

所以新的思路：在xml树里，每个图片都有一个r:id属性，值like：rid39。这个rid是word的映射对象，在解压缩后的document.xml.rel中可以找到对应的映射图片，比如rid39对应的target是图片12。

所以先变为rid,然后通过document.xml.rel文件将rid变为图片地址

```python
import docx
import xml.etree.cElementTree as ET
import sys
import io
import re
sys.stdout=io.TextIOWrapper(sys.stdout.buffer,encoding='utf8')

fs = docx.Document('computer_science.docx')

lineCounts = 100

def parseRidByRel(rid):
   root = ET.ElementTree(file='document.xml.rels')
   for ele in root.iter():
      if ele.tag == "{http://schemas.openxmlformats.org/package/2006/relationships}Relationship":
         if ele.get('Id') == rid:
            return ele.get('Target')
   return "Null"

def parseLinePic(line):
   root = ET.fromstring(line.paragraph_format.element.xml)
   outStr = ''
   for ele in root.iter():
      # print(ele.tag)
      # continue
      if(ele.tag=="{http://schemas.openxmlformats.org/wordprocessingml/2006/main}t"):
         outStr += ele.text
      if(ele.tag == "{urn:schemas-microsoft-com:vml}imagedata"):
         rid = ele.get('{http://schemas.openxmlformats.org/officeDocument/2006/relationships}id')
         outStr += "<word-pic>"+parseRidByRel(rid)+"</word-pic>"
   return outStr
      

for line in fs.paragraphs:
   if lineCounts <= 0:
      exit()
   outStr = parseLinePic(line)
   print(outStr)
   

   lineCounts -= 1

```

```text
输出
[Running] python -u "c:\Users\tassa\Downloads\test.py"
第一章 计算机网络体系结构
知识导图
<word-pic>media/image1.png</word-pic><word-pic>media/image2.png</word-pic>
历年真题
单项选择题
1.在OSI参考模型中，直接为会话层提供服务的是，插入测试图片如下图（   ）。             [全国联考2014年]
<word-pic>media/image2.png</word-pic>
A.应用层                             B.表示层
C.运输层                             D.网络层
2.在OSI参考模型中，<word-pic>media/image3.wmf</word-pic>下列功能需由应用层的相邻层实现的是（   ）。   [全国联考2013年]
A.对话管理                           B.数据格式转换
C.路由选择                           D.可靠数据传输
3.TCP/IP参考模型的网络层提供的是（   ）。                        [全国联考2011年]
A.无连接不可靠的数据报服务           B.无连接可靠的数据报服务
C.有连接不可靠的虚电路服务           D.有连接可靠的虚电路服务
4.下列选项中，不属于网络体系结构所描述的内容是（   ）。          [全国联考2010年]
A.网络的层次                         B.每一层使用的协议
C.协议的内部实现细节                 D.每一层必须完成的功能
5.在OSI参考模型中，自下而上第一个提供端到端服务的层次是（   ）。 [全国联考2009年]
A.数据链路层                         B.运输层
C.会话层                             D.应用层
6.有关交换技术的论述，以下哪个是正确的？（   ）          [杭州电子科技大学2017年]
A.电路交换要求在通信的双方之间建立起一条实际的物理通路，但通信过程中，这条通路可以与别的通信方共享
B.现有的公用数据网都采用报文交换技术
C.报文交换可以满足实时或交互式的通信要求
D.分组交换将一个大报文分割成分组，并以分组为单位进行存储转发在接收端再将各分组重新装成一个完整的报文
7.下列关于网络体系结构的描述中正确的是（   ）。          [杭州电子科技大学2017年]
A.网络协议中的语法涉及的是用于协调与差错处理有关的控制信息
B.在网络分层体系结构中，n层是<word-pic>media/image4.wmf</word-pic>层的用户，又是<word-pic>media/image5.wmf</word-pic>层的服务提供者
C.OSI参考模型包括了体系结构、服务定义和协议规范三级抽象
D.OSI和TCP/IP模型的网络层同时支持面向连接的通信和无连接通信
8.下列关于虚电路方式中路由选择的正确说法是（   ）。      [杭州电子科技大学2017年]
A.建立连接和发送数据都不进行路由选择
B.只在建立虚电路时进行路由选择
C.传送数据时进行路由选择
D.建立连接和发送数据都进行路由选择
9.网络协议的主要要素为（   ）。                          [桂林电子科技大学2016年]
A.数据格式、编码、信号电平
B.数据格式、控制信息、时序
C.语法、语义、时序
D.编码、控制信息、数据格式
10.下述对广域网的作用范围叙述最准确的是（   ）。         [桂林电子科技大学2016年]
A.几千米到十几千米
B.几十千米到几百千米
C.几十千米到几千千米
D.几千千米以上
11.如果比特率为<word-pic>media/image6.wmf</word-pic>，发送3000位需要多长时间？（   ）  [桂林电子科技大学2016年]
A.<word-pic>media/image7.wmf</word-pic>                              B.<word-pic>media/image8.wmf</word-pic>
C.<word-pic>media/image9.wmf</word-pic>                            D.<word-pic>media/image10.wmf</word-pic>
[Done] exited with code=0 in 1.571 seconds


```

其次在完成了映射后还有一点需要做的就是（html是无法解析wmf文件的）把wmf转为pic。

思路是把media文件夹下的所有文件转为png结尾，文档解析的<word-pic>内的wmf后缀转为png。pip install Pillow后
使用 Pillow库转换代码：

```python
def wmf2Png(fileName):
   Image.open(fileName).save(re.sub('wmf','png',fileName))

wmf2Png("image5.wmf")

```