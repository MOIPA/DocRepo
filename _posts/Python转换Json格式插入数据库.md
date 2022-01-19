title: Python转换Json格式插入数据库
author: tr
tags:
  - Python
categories:
  - Python
date: 2019-04-24 17:26:00
---
### 公司任务

是一个使用json和list的例子,所以还是记录下

<!--more-->

#### 代码：

```python
# -*- coding: utf-8 -*-
# parse json and use api to upload all question
import json
import urllib
import urllib2
import requests
import re

QUESTION_BANK_ID=77
KNOWLEDGE_POINT_IDS=['1']
QUESTION_ADD_URL='http://192.168.0.100:8882/api/v1/questions/add'
X_USERID='2'

def commonChange(question):
    del question['useCount']
    del question['id']
    question['questionBankId']= QUESTION_BANK_ID


def pushData(json_data,url):
    headers={'content-type': 'application/json', 'X-UserId': X_USERID}
    response = requests.post(url, data=json_data, headers=headers)
    if response.status_code:
        print 'insert succeed'

def formatQuestion(question):
    commonChange(question)
    # 题型  0：判断题，1：单选题，2：多选题，3：填空题，4：简答题，5：代码题
    if question['questionType'] == 0:
        formatJudgeQuestion(question)
    elif question['questionType']==1:
        formatSingleChoiceQuestion(question)
    elif question['questionType']==2:
        formatMuitlChoiceQuestion(question)
    elif question['questionType']==3:
        formatFillQuestion(question)
    elif question['questionType']==4:
        formatAnswerQuestion(question)
    elif question['questionType']==5:
        formatCodeQuestion(question)
    else:
        print 'error can\'t parse'
        
def formatSingleChoiceQuestion(question):
    print 'format 单选题'
    question['knowledgePointIds']=KNOWLEDGE_POINT_IDS
    # del tags's id
    for i in range(len(question['tags'])):
        del question['tags'][i]['id']
    for i in range(len(question['optionDTOS'])):
        del question['optionDTOS'][i]['id']
    question['options'] = question['optionDTOS']
    del question['optionDTOS']

    #push data 
    pushData(json.dumps(question),QUESTION_ADD_URL)
    #exit('test formatSingleChoiceQuestion')


def formatJudgeQuestion(question):
    print 'format 判断题'
    # delete optionsDTOS
    del question['optionDTOS']
    # del tags's id
    for i in range(len(question['tags'])):
        del question['tags'][i]['id']
    #push data 
    pushData(json.dumps(question),QUESTION_ADD_URL)
    #exit('test formatSingleChoiceQuestion')
    
def formatMuitlChoiceQuestion(question):
    print 'format 多选题'


def formatFillQuestion(question):
    print 'format 填空题'
    # delete optionsDTOS
    del question['optionDTOS']
    question['knowledgePointIds']=KNOWLEDGE_POINT_IDS
    parseStr = question['stem']
    blankID=''
    blanks=[]
    while re.match(r'.*(__.{1}__).*',parseStr):
        blankID=re.match(r'.*(__.{1}__).*',parseStr).group(1)         
        parseStr = re.sub(blankID,'',parseStr)
        blanks.append({
            "blankAnswer":["sorry no answer avaiable for "+blankID],
            "blankId":blankID    
        })

    question['blanks'] = json.dumps(blanks)
    #push data 
    pushData(json.dumps(question),QUESTION_ADD_URL)
    #exit('test formatSingleChoiceQuestion')

def formatAnswerQuestion(question):
    print 'format 简答题'
    #push data 
    pushData(json.dumps(question),QUESTION_ADD_URL)
    #exit('test formatSingleChoiceQuestion')

def formatCodeQuestion(question):
    print 'format 代码题'

def main(fileName):
    with open(fileName,'r') as f:
        load_json = json.load(f)
        question_size = load_json['size']
        # start formating 
        for i in range(question_size):
            formatQuestion(load_json['res']['questions'][i])
            

if __name__=="__main__":
    main('./quiz.json')


```