title: Python+oss+json+request
author: tr
tags:
  - Python
categories:
  - Python
  - ''
date: 2019-05-12 22:29:00
---
#### python 调用oss实例

code:
<!--more-->
```python

# -*- coding: utf-8 -*-
# parse json and use api to upload all question
import json
import urllib
import urllib2
import requests
import re
import url_utils  
from urllib import urlretrieve
import oss2

QUESTION_BANK_ID=88
QUESTION_ADD_URL='http://47.99.50.105:8882/api/v1/questions/add'
X_USERID='72'
X_ORGANIZATION_ID = '21'

rootURL = 'http://miscrofile.oss-cn-hangzhou.aliyuncs.com/'
targetFold = 'root/question/yuanketang/'

access_key_id = 'LTAILQg6DvGdPrFN'
access_key_secret = 'pN6ScRagLy6k2dehw8BheFmNR7WDtZ'
bucket_name = 'miscrofile'
oss_url = 'http://oss-cn-hangzhou.aliyuncs.com'

def uploadFileViaSDK(imgName,targetFold):
    #(signature,dir,host,accessId,policy)=getToken(url,imgName)
    bucket = oss2.Bucket(oss2.Auth(access_key_id,access_key_secret), oss_url, bucket_name)
    # do upload file
    key = targetFold+imgName
    bucket.put_object_from_file(key,imgName)
    result = bucket.get_object(key)

def getAndUploadPic(localUrl):
    img = re.match(r'.*/(.*png)',localUrl).group(1)
    #get pic from this url
    urlretrieve(localUrl, './'+img)         
    #upload this picture and return url
    uploadFileViaSDK(img,targetFold)
    return rootURL+targetFold+img

def doReplace(line):
    #picm = re.match(r'.*?(http.*png)',line)
    #if picm:
    #    localUrl = picm.group(1)
    #    remoteUrl = getAndUploadPic(localUrl)
    #    line = re.sub(r'http.*png',remoteUrl,line)
    #return line
    pics = re.findall(r'http.*?png',line)
    for localUrl in pics:
        print 'do replace pic'
        line = re.sub(localUrl,getAndUploadPic(localUrl),line)
    print line
    return line


# replace picures' local url with oss-url
def replacePicUrl(question):
    questionJson = json.dumps(question)
    question = question.loads 
    #question['analysis'] = doReplace(str(question['analysis']))
    #question['answer'] = doReplace(str(question['answer']))
    
# pre-operation of every question
def commonChange(question,categoryId):
    #replacePicUrl(question)

    del question['updateTime']
    del question['id']
    del question['version']
    del question['disabled']
    del question['author']
    del question['label']
    del question['reviewed']
    del question['createTime']
    question['questionBankId'] = QUESTION_BANK_ID
    question['knowledgePointIds']=[question['knowledgePoints'][0]['id']]
    del question['knowledgePoints']
    # del tags's id
    for i in range(len(question['tags'])):
        del question['tags'][i]['id']
    # convert difficulty
    if question['difficulty'] == 'SIMPLE' :
        question['difficulty']=0
    elif question['difficulty'] == 'NORMAL' :
        question['difficulty']=1
    elif question['difficulty'] == 'HARD' :
        question['difficulty']=2
    else:
        print 'error , unknown difficulty type'
    
    # add categoryId
    question['categoryId'] = categoryId

    # convert questionType
    if question['questionType']=='TRUE_OR_FALSE':
        question['questionType']=0
    elif question['questionType']=='SINGLE_CHOICE':
        question['questionType']=1
    elif question['questionType']=='MULTIPLE_CHOICE':
        question['questionType']=2
    elif question['questionType']=='FILL_IN_BLANK':
        question['questionType']=3
    elif question['questionType']=='SUBJECTIVE':
        question['questionType']=4
    else:
        print 'error , unknown question type'

# format all types of questions 
def formatQuestion(question,categoryId):
    commonChange(question,categoryId)

    question = json.loads(doReplace(json.dumps(question)))
    # 题型  0：判断题checked，1：单选题checked，2：多选题checked，3：填空题checked，4：简答题，5：代码题
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
    del question['answer']
    for i in range(len(question['options'])):
        del question['options'][i]['id']

    #push data 
    url_utils.pushData(json.dumps(question),QUESTION_ADD_URL,X_USERID,X_ORGANIZATION_ID)
    #exit('test formatSingleChoiceQuestion')

def formatJudgeQuestion(question):
    print 'format 判断题'
    # get answer
    question['answer'] = question['answer']['value']
    #push data 
    url_utils.pushData(json.dumps(question),QUESTION_ADD_URL,X_USERID,X_ORGANIZATION_ID)
    
def formatMuitlChoiceQuestion(question):
    print 'format 多选题'
    del question['answer']
    for i in range(len(question['options'])):
        del question['options'][i]['id']
    #push data 
    url_utils.pushData(json.dumps(question),QUESTION_ADD_URL,X_USERID,X_ORGANIZATION_ID)

def formatFillQuestion(question):
    print 'format 填空题'
    # format question blank
    parseStr = question['stem']
    blankID=''
    blanks=[]
    while re.match(r'.*(__.{1}__).*',parseStr):
        blankID=re.match(r'.*(__.{1}__).*',parseStr).group(1)         
        parseStr = re.sub(blankID,'',parseStr)
        blanks.append({
            "blankAnswer":[question['blankItems'][blankID]['answer']['value']],
            "blankId":blankID    
        })

    question['blanks'] = json.dumps(blanks)
    #push data 
    url_utils.pushData(json.dumps(question),QUESTION_ADD_URL,X_USERID,X_ORGANIZATION_ID)
    #exit('test formatSingleChoiceQuestion')

def formatAnswerQuestion(question):
    print 'format 简答题'
    # get answer
    try:
        question['answer']=question['answer']['value']
    except Exception as e:
        print e
        print 'error  no answer,and stem is: '+question['stem']
    #push data 
    url_utils.pushData(json.dumps(question),QUESTION_ADD_URL,X_USERID,X_ORGANIZATION_ID)
    #exit('test formatSingleChoiceQuestion')

def formatCodeQuestion(question):
    print 'format 代码题'

# parse every question in the list
def parseQuestions(questionArray,categoryId):
    for i in range(len(questionArray)):
        load_json = json.loads(questionArray[i])
        formatQuestion(load_json['res'],categoryId)

# create and parse questions
def doAction(getAllQuestionUrl,getSingleQuestionUrl,token,requirementBasicBody,categoryName):
    # create category
    categoryId = url_utils.createCategory(categoryName,'http://47.99.50.105:8882/api/v1/category/add',X_USERID,X_ORGANIZATION_ID)
    # get and parse '需求分析' data
    questionArray = url_utils.getAndParseData(getAllQuestionUrl,getSingleQuestionUrl,token,requirementBasicBody)
    parseQuestions(questionArray,categoryId)

def main():
    # define environment
    token = 'bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJtc2ciOiJzdWNjZXNzIiwiY29kZSI6MCwidXNlcl9uYW1lIjoicm9vdCIsInNjb3BlIjpbImFsbCIsInJlYWQiLCJ3cml0ZSJdLCJleHAiOjE1NTc3MjA1OTgsInVzZXJJZCI6MSwiYXV0aG9yaXRpZXMiOlsiUk9MRV9VU0VSIiwiUk9MRV9URUFDSEVSIl0sImp0aSI6ImEwY2MwYjkzLTZlNjUtNDBiNy05YjY3LWQ3OGI1NmQ3MGM0OCIsImNsaWVudF9pZCI6Inl1YW50dSJ9.adVGNhqKZm6rIeLnmmL3a1Jx8A5oDbf31Q5LNkI0Z-rIzsmYIJ15FNXKLmPO2_H2izr9VQ5vDirHoP9YY_jrPpZvtB57hTYP4n116yIjAtlScVQzVlHi_bwtZH7OSdbwcJ76nJzluvbD-xIk177i9tmMI3KjKrToPNHdIlLUGBMBo25RjplFig1MRrfvv-DAZnH7ymxoCGbNnnaGtKmYJIgCIk5IAAhekFaF-iKIyQT7bFMN5KYAM02YwvbWjQYs0C4l_nACZ-yG9KW24s5iepna44E_34UW4rsNzAdGods1P039CXrkVH83M14hxpI2qClmKZASmvWqAQizM1tF6Q'
    getAllQuestionUrl = 'http://192.168.0.100:18080/api/v1/questions/search/conditions'
    getSingleQuestionUrl = 'http://192.168.0.100:18080/api/v1/question/'

    body = []
    category = []
    # define '需求基础' finished
    body.append('{"knowledgePointIds":[2415],"difficulties":null,"questionTypes":null,"knowledgeState":null,"pageSize":1000,"page":0,"keywords":""}')
    category.append('需求基础')
    # define 'oop'
    body.append('{"knowledgePointIds":[3004,3006,3007,3008,3009,3010,3011,3020,3021,3035,3523],"difficulties":null,"questionTypes":null,"knowledgeState":null,"pageSize":1000,"page":0,"keywords":""}')
    category.append('OOP')
    # '职责分配'
    body.append('{"knowledgePointIds":[3379],"difficulties":null,"questionTypes":null,"knowledgeState":null,"pageSize":1000,"page":0,"keywords":""}')
    category.append('职责分配')
    # '模块化'
    body.append('{"knowledgePointIds":[2749],"difficulties":null,"questionTypes":null,"knowledgeState":null,"pageSize":1000,"page":0,"keywords":""}')
    category.append('模块化')
    # '设计模式'
    body.append('{"knowledgePointIds":[2457],"difficulties":null,"questionTypes":null,"knowledgeState":null,"pageSize":1000,"page":0,"keywords":""}')
    category.append('设计模式')
    # '起始体系结构构建'
    body.append('{"knowledgePointIds":[2982],"difficulties":null,"questionTypes":null,"knowledgeState":null,"pageSize":1000,"page":0,"keywords":""}')
    category.append('起始体系结构构建')
    # 'OO分析'
    body.append('{"knowledgePointIds":[2673,2708,2715,2717,3054,3055,3056,2716],"difficulties":null,"questionTypes":null,"knowledgeState":null,"pageSize":1000,"page":0,"keywords":""}')
    category.append('OO分析')
    # '用例描述'
    body.append('{"knowledgePointIds":[2673],"difficulties":null,"questionTypes":null,"knowledgeState":null,"pageSize":1000,"page":0,"keywords":""}')
    category.append('用例描述')

    # do 
    #doAction(getAllQuestionUrl,getSingleQuestionUrl,token,bodyA,"需求基础")
    for i in range(len(body)):
        doAction(getAllQuestionUrl,getSingleQuestionUrl,token,body[i],category[i])
            

if __name__=="__main__":
    main()


```