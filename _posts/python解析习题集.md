title: python解析习题集
author: tr
tags:
  - Python
categories:
  - Python
date: 2019-04-11 09:52:00
---
### 使用python解析txt题目文本

 公司需求，解析一本习题集，将所有的单选题和答案解析放在一个对象，转成json输出
 
 <!--more-->
 
 ```python
 
 # -*- coding: utf-8 -*-
import json
import os
import re
import shutil
import sys
import xml.etree.cElementTree as ET
import docx
import requests
from post_question import uploadFileViaSDK
from extract_word_pic import word2pic
from format_json import formatJson

#reload(sys)
#sys.setdefaultencoding('utf8')
#TODO 优化图片，优化tag寻图算法（不同word版本导致的tag变动问题）

SINGLE_CHOICE = 'single_choice_question'
FILL_IN = 'fill_in_question'
ANSWER_QUESTION = 'answer_question'
OBJECTIVE_QUESTION = 'objective_question'

QUESTION_BANK_ID=153
QUESTION_ADD_URL='http://192.168.0.100:8882/api/v1/questions/add'
X_USERID='72'
X_ORGANIZATION_ID = '21'

UPLOAD_OSS = False
PUSH_DATA = False

def parseRidByRel(rid):
   root = ET.ElementTree(file='./word_pic_save/document.xml.rels')
   for ele in root.iter():
      if ele.tag == "{http://schemas.openxmlformats.org/package/2006/relationships}Relationship":
         if ele.get('Id') == rid:
            return re.sub('media/','',ele.get('Target'))
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
         if UPLOAD_OSS:
             outStr += uploadOSS(re.sub('wmf','png',parseRidByRel(rid)))
         else:
             outStr += "<word-pic>"+re.sub('wmf','png',parseRidByRel(rid))+"</word-pic>"
   return outStr

def pushData(json_data, url):
    headers = {'content-type': 'application/json', 'X-UserId': X_USERID}
    response = requests.post(url, json=json.loads(json_data), headers=headers)
    if response.status_code == requests.codes.ok:
        print 'insert succeed '+response.text
    else:
        print 'failed'+str(response.status_code)


def uploadOSS(imageFile):
        # print picID+" is the id"
        rootURL = 'https://miscrofile.oss-cn-hangzhou.aliyuncs.com/'
        targetFold = 'root/question/computer_science/'
        targetFile = imageFile 
        try:
            uploadFileViaSDK('./word_pic_save/'+targetFile, targetFold)
        except:
            print str(picID)+' pic upload failed'
        return '<img src=\'' + rootURL+targetFold+targetFile+'\'/>'


# parse the line's pic,upload this picture and return the replaced line
def uploadOSSOld(picID):
        # print picID+" is the id"
        rootURL = 'https://miscrofile.oss-cn-hangzhou.aliyuncs.com/'
        targetFold = 'root/question/computer_science/'
        targetFile = 'image'+str(picID)+'.png'
        if UPLOAD_OSS == False:
            #return '<word-pic>' +targetFile+'<word-pic/>'
            return targetFile
        try:
            uploadFileViaSDK('./word_pic_save/'+targetFile, targetFold)
        except:
            print str(picID)+' pic upload failed'
        return '<img src=\'' + rootURL+targetFold+targetFile+'\'/>'

def parsePic(line, counter):
    # indentify pic
    root = ET.fromstring(line.paragraph_format.element.xml)
    for ele in root.iter():
        if(ele.tag == '{http://schemas.openxmlformats.org/wordprocessingml/2006/main}pict'):
            counter[0] += 1
            return uploadOSS(counter[0])+'\n'


class Question:
    type = ''
    options = ''
    stem = ''
    answer = ''
    id = ''
    analysis = ''
    point = ''
    short_question_answer = ''

    # format options
    def formatOptions(self):
        checkedOptions = []
        if self.options != '' and self.type == SINGLE_CHOICE and len(self.options) > 0:
            # print "****options***"+json.dumps(question, default=lambda o: o.__dict__,sort_keys=True, indent=4,ensure_ascii=False)
            #	seq =  re.match(r'^([A|B|C|D|a|b|c|d])[\.|•]?\s*(.*)',question.option[0].strip())
            #	dot =  re.match(r'^([A|B|C|D|a|b|c|d])[\.|•]\s*(.*)',question.option[0].strip())
            for op in self.options:
                # able to judge
                seq = re.match(
                    r'^([A|B|C|D|a|b|c|d])[\.|•]?\s*(.*)', op.strip())
                dot = re.match(
                    r'^([A|B|C|D|a|b|c|d])[\.|•]\s*(.*)', op.strip())
                if seq:
                    # sequence shuould be the same type
                    if seq.group(1) == 'A' or seq.group(1) == 'B' or seq.group(1) == 'C' or seq.group(1) == 'D':
                        srchType = "UPPERCASE"
                    if seq.group(1) == 'a' or seq.group(1) == 'b' or seq.group(1) == 'c' or seq.group(1) == 'd':
                        srchType = "LOWERCASE"
                    # proceed with seqType and dot
                    # a b c d
                    if seq.group(1) == 'A' or seq.group(1) == 'a':
                        if srchType == 'UPPERCASE' and dot:
                            rest1 = re.match(
                                r'(.*)B[\.|•]\s*(.*)', seq.group(2))
                        elif srchType == 'UPPERCASE' and dot == None:
                            rest1 = re.match(
                                r'(.*)B[\.|•]?\s*(.*)', seq.group(2))
                        elif srchType == 'LOWERCASE' and dot:
                            rest1 = re.match(
                                r'(.*)b[\.|•]\s*(.*)', seq.group(2))
                        elif srchType == 'LOWERCASE' and dot == None:
                            rest1 = re.match(
                                r'(.*)b[\.|•]?\s*(.*)', seq.group(2))
                        # B options exists
                        if rest1:
                            checkedOptions.append('A.' + rest1.group(1))

                            if srchType == 'UPPERCASE' and dot:
                                rest2 = re.match(
                                    r'(.*)C[\.|•]\s*(.*)', rest1.group(2))
                            elif srchType == 'UPPERCASE' and dot == None:
                                rest2 = re.match(
                                    r'(.*)C[\.|•]?\s*(.*)', rest1.group(2))
                            elif srchType == 'LOWERCASE' and dot:
                                rest2 = re.match(
                                    r'(.*)c[\.|•]\s*(.*)', rest1.group(2))
                            elif srchType == 'LOWERCASE' and dot == None:
                                rest2 = re.match(
                                    r'(.*)c[\.|•]?\s*(.*)', rest1.group(2))

                            # C options exists
                            if rest2:
                                checkedOptions.append(
                                    'B.' + rest2.group(1))

                                if srchType == 'UPPERCASE' and dot:
                                    rest3 = re.match(
                                        r'(.*)D[\.|•]\s*(.*)', rest2.group(2))
                                elif srchType == 'UPPERCASE' and dot == None:
                                    rest3 = re.match(
                                        r'(.*)D[\.|•]?\s*(.*)', rest2.group(2))
                                elif srchType == 'LOWERCASE' and dot:
                                    rest3 = re.match(
                                        r'(.*)d[\.|•]\s*(.*)', rest2.group(2))
                                elif srchType == 'LOWERCASE' and dot == None:
                                    rest3 = re.match(
                                        r'(.*)d[\.|•]?\s*(.*)', rest2.group(2))

                                # rest3 = re.match(r'(.*)[D|d][\.|•]?\s*(.*)',rest2.group(2))
                                # D options exists
                                if rest3:
                                    checkedOptions.append(
                                        'C.' + rest3.group(1))
                                    checkedOptions.append(
                                        'D.' + rest3.group(2))
                                else:
                                    checkedOptions.append(
                                        'C.' + rest2.group(2))
                            else:
                                checkedOptions.append(
                                    'B.' + rest1.group(2))

                        else:
                            checkedOptions.append('A.' + seq.group(2))
                    # b c d
                    elif seq.group(1) == 'B' or seq.group(1) == 'b':
                        if srchType == 'UPPERCASE' and dot:
                            rest1 = re.match(
                                r'(.*)C[\.|•]\s*(.*)', seq.group(2))
                        elif srchType == 'UPPERCASE' and dot == None:
                            rest1 = re.match(
                                r'(.*)C[\.|•]?\s*(.*)', seq.group(2))
                        elif srchType == 'LOWERCASE' and dot:
                            rest1 = re.match(
                                r'(.*)c[\.|•]\s*(.*)', seq.group(2))
                        elif srchType == 'LOWERCASE' and dot == None:
                            rest1 = re.match(
                                r'(.*)c[\.|•]?\s*(.*)', seq.group(2))
                        # rest1 = re.match(r'(.*)[C|c][\.|•]?\s*(.*)',seq.group(2))
                        # C options exists
                        if rest1:
                            checkedOptions.append('B.' + rest1.group(1))
                            if srchType == 'UPPERCASE' and dot:
                                rest2 = re.match(
                                    r'(.*)D[\.|•]\s*(.*)', rest1.group(2))
                            elif srchType == 'UPPERCASE' and dot == None:
                                rest2 = re.match(
                                    r'(.*)D[\.|•]?\s*(.*)', rest1.group(2))
                            elif srchType == 'LOWERCASE' and dot:
                                rest2 = re.match(
                                    r'(.*)d[\.|•]\s*(.*)', rest1.group(2))
                            elif srchType == 'LOWERCASE' and dot == None:
                                rest2 = re.match(
                                    r'(.*)d[\.|•]?\s*(.*)', rest1.group(2))
                            #	rest2 = re.match(r'(.*)[D|d][\.|•]?\s*(.*)',rest1.group(2))
                            # D options exists
                            if rest2:
                                checkedOptions.append(
                                    'C.' + rest2.group(1))
                                checkedOptions.append(
                                    'D.' + rest2.group(2))
                            else:
                                checkedOptions.append(
                                    'C.' + rest1.group(2))

                        else:
                            checkedOptions.append('B.' + seq.group(2))
                    # c d
                    elif seq.group(1) == 'C' or seq.group(1) == 'c':
                        if srchType == 'UPPERCASE' and dot:
                            rest1 = re.match(
                                r'(.*)D[\.|•]\s*(.*)', seq.group(2))
                        elif srchType == 'UPPERCASE' and dot == None:
                            rest1 = re.match(
                                r'(.*)D[\.|•]?\s*(.*)', seq.group(2))
                        elif srchType == 'LOWERCASE' and dot:
                            rest1 = re.match(
                                r'(.*)d[\.|•]\s*(.*)', seq.group(2))
                        elif srchType == 'LOWERCASE' and dot == None:
                            rest1 = re.match(
                                r'(.*)d[\.|•]?\s*(.*)', seq.group(2))
                        # rest1 = re.match(r'(.*)[D|d][\.|•]?\s*(.*)',seq.group(2))
                        # D options exists
                        if rest1:
                            checkedOptions.append('C.' + rest1.group(1))
                            checkedOptions.append('D.' + rest1.group(2))
                        else:
                            checkedOptions.append('C.' + seq.group(2))
                    # d
                    elif seq.group(1) == 'D' or seq.group(1) == 'd':
                        checkedOptions.append('D.' + seq.group(2))

            self.options = checkedOptions
            if len(checkedOptions) != 4:
                self.id = -1


    def formatQuestion(self):
        return formatJson(json.dumps(self, default=lambda o: o.__dict__, sort_keys=True, indent=4, ensure_ascii=False),QUESTION_BANK_ID,SINGLE_CHOICE,FILL_IN,ANSWER_QUESTION,OBJECTIVE_QUESTION)
        #print json.dumps(self, default=lambda o: o.__dict__, sort_keys=True, indent=4, ensure_ascii=False)


class Chapter:
    chapterText = ''

    questionArray = []

    def __init__(self, chapterText):
        self.chapterText = chapterText

    def parseQuestion(self, singleChoiceText, questionType):
        questionArray = []
        flag = 'unstart'
        tempString = ''
        options = []
        results = singleChoiceText.split('\n')
        for line in results:
            # remove chinese "quanjiao"
            line = line.replace('\xa1\xa1', '')
            # match empth line
            if re.match(r'^\s*$', line):
                continue

            # match item such as text begining with 1.or 12 .etc
            m = re.search(r'^[0-9]+[\.|•]\s*(.*)$', line.strip())
            # Match if it is an option
            m_op = re.search(r'^[A|B|C|D|a|b|c|d]\s*(.*)$', line.strip())
            if m != None or m_op == None:
                if m:
                    #if flag == 'option' or flag == 'stem':
                    if flag == 'option' or (questionType != SINGLE_CHOICE and flag=='stem'):
                        if flag == 'stem':
                            question.stem = re.sub(
                                '^[0-9]*\.?', '', tempString)
                        # save the last option
                        question.options = options
                        questionArray.append(question)
                        options = []
                        tempString = ""
                    # get question id
                    qid = re.search(
                        r'\s*([0-9]*)[\.|•]\s*(.*)$', line).group(1)
                    question = Question()
                    question.id = qid
                    question.type = questionType
                    # reset to 1 starting filling topic
                    flag = 'stem'

                if m_op == None and flag == 'stem':
                    tempString += line

            else:
                # fill question topic
                if flag == 'stem':
                    question.stem = re.sub('^[0-9]*\.?', '', tempString)
                    tempString = ""
                    flag = 'option'
                res = re.split(
                    r'\s*[A|B|C|D|a|b|c|d]\s*\.{3}\s*\s*', line.strip())

                # match options
                for elem in res:
                    if elem:
                        options.append(elem)
        # end finish appending
        question.options = options
        questionArray.append(question)
        if question.stem == '':
            question.stem = re.sub('^[0-9]*\.?', '', tempString)

        return questionArray

    # used to merge data to topic
    def appendAnswer2Topic(self, questionArray, id, answer, point, analysis, short_question_answer):
        for elem in (question for question in questionArray if question.id == id):
            elem.short_question_answer = short_question_answer
            elem.answer = answer
            elem.point = point
            elem.analysis = analysis

    # append answer
    def appendAnswer(self, answerText):
        #print answerText
         # status to identify the operation status,-1 no, 1 answer, 2 point, 3 analysis , 4 nomal
        status = -1
        result = answerText.split('\n')
        answer_question_answer = ''
        for line in result:
            # remove chinese "quanjiao"
            line = line.replace('\xa1\xa1', '')

            # match empth line
            if re.match(r'^\s*$', line):
                continue
            # match answer id line
            aidm = re.match(r'\s*([0-9]*)[\.|•]', line)
            # print '** aidm is:'+str(aidm)
            # match answer
            answerm = re.match(r'\s*[0-9]*\s*\.?\s*【答案】(.*)', line)
            # match point
            pointm = re.match(r'\s*【考点】(.*)', line)
            # match analysis
            analysism = re.match(r'\s*【解析】(.*)', line)
            # match answer of answer-question
            answer_question_answerm = re.match(r'\s*【参考答案】(.*)', line)


            # judge status
            if aidm:
                # encounter init part, handle result if it exists
                if status != -1:
                    # append operation
                    self.appendAnswer2Topic(self.questionArray,
                                            aid, answer, point, analysis, answer_question_answer)

                aid = aidm.group(1)
                
            if answer_question_answerm:
                answer_question_answer = '[answer_question_answer]' + answer_question_answerm.group(1)
                status = 4    
            elif answerm:
                answer = '[answer]' + answerm.group(1)
                status = 1
            elif pointm:
                point = '[point]' + pointm.group(1)
                status = 2
            elif analysism:
                analysis = '[analysis]' + analysism.group(1)
                status = 3
            else:
                if status == 3:
                    analysis += line
                if status == 2:
                    point += line
                if status == 4:
                    answer_question_answer += line
                if status == 1:
                    answer += line
                # if status != -1:
                #    status = 4
        # end finish appending
        self.appendAnswer2Topic(self.questionArray, aid,
                                answer, point, analysis,answer_question_answer)

    def devideAnswerAndQuestion(self, chapterText):
        #print '*****************'
        #print chapterText
        result = re.split(r'\s*参考答案与解析\s*', chapterText)
        return (result[0], result[1])

    def parseDiffQuestion(self, questionText):
         # define question array
        singleQuestionText = ''
        fillQuestionText = ''
        answerQuestionText = ''
        objectiveQuestionText = ''

        questions = re.split(r'(单项选择题|填空题|简答题|综合应用题)', questionText)
        for i in range(len(questions)):
            if re.match(r'单项选择题', questions[i]) and re.match(r'.*[单项选择题,填空题,简答题,综合应用题]', questions[i+1]) == None:
                singleQuestionText = questions[i+1]
            if re.match(r'填空题', questions[i]) and re.match(r'.*[单项选择题,填空题,简答题,综合应用题]', questions[i+1]) == None:
                fillQuestionText = questions[i+1]
            if re.match(r'简答题', questions[i]) and re.match(r'.*[单项选择题,填空题,简答题,综合应用题]', questions[i+1]) == None:
                answerQuestionText = questions[i+1]
            if re.match(r'综合应用题', questions[i]) and re.match(r'.*[单项选择题,填空题,简答题,综合应用题]', questions[i+1]) == None:
                objectiveQuestionText = questions[i+1]

        # convert stem
        if singleQuestionText != '':
            self.questionArray += self.parseQuestion(
                singleQuestionText, SINGLE_CHOICE)
        if fillQuestionText != '':
            self.questionArray += self.parseQuestion(fillQuestionText, FILL_IN)
        if answerQuestionText != '':
            self.questionArray += self.parseQuestion(
                answerQuestionText, ANSWER_QUESTION)
        if objectiveQuestionText != '':
            self.questionArray += self.parseQuestion(
                objectiveQuestionText, OBJECTIVE_QUESTION)

    def parseChapter(self):

        # split question and answer
        (questionText, answerText) = self.devideAnswerAndQuestion(self.chapterText)
        # split question and parse them into questionArray
        self.parseDiffQuestion(questionText)

        # format single_choice_question's option
        for single_choice in (question for question in self.questionArray if question.type == SINGLE_CHOICE):
            single_choice.formatOptions()

        # append answer
        self.appendAnswer(answerText)

        #print json.dumps(self.questionArray, default=lambda o: o.__dict__, sort_keys=True, indent=4, ensure_ascii=False)

    def getQuestionArray(self):
        return self.questionArray


class Book:
    chapters = []

    def showChapter(self):
        for i in range(len(self.chapters)):
            print (self.chapters[i].chapterText)

    # skip category and replace every line's picture to picture number
    def parseChapter(self, wordFile):
        # define parse status
        parseStatus = 'unstart'
        # define plain text as the content of chapter
        chapterText = ''

        # open the book file
        fs = docx.Document(wordFile)
        for line in fs.paragraphs:

            line.text = parseLinePic(line)
            # remove chinese "quanjiao"
            line = line.text.decode().encode('utf-8') + '\n'
            line = line.replace('\xa1\xa1', '')

            # match empth line
            if re.match(r'^\s*$', line):
                continue
            # match chapter
            chapterIdm = re.match(r'\s*第(.*)章\s*', line)

            # chapter title
            if chapterIdm:
                # if the parsing progress is started record the text
                if parseStatus == 'start':
                    self.chapters.append(Chapter(chapterText))
                    chapterText = ''
                parseStatus = 'start'
            # normal line
            else:
                if parseStatus == 'start':
                    chapterText += line

        # finish the last chapter
        self.chapters.append(Chapter(chapterText))
        #print (chapterText)
        chapterText = ''

    def getChapters(self):
        return self.chapters


def main(path):
    book = Book()
    book.parseChapter(path)
    # book.showChapter()
    chapters = book.getChapters()
    for i in range(len(chapters)):
        chapters[i].parseChapter()
        for question in chapters[i].getQuestionArray():
            jsonObj  = question.formatQuestion()
            questionJson =  json.dumps(jsonObj, default=lambda o: o.__dict__, sort_keys=True, indent=4, ensure_ascii=False)
            print questionJson
            if PUSH_DATA:
                pushData(questionJson,QUESTION_ADD_URL)
        return


if __name__ == "__main__":
    # main("./", "http://192.168.0.99:8882/api/v1/questions/add")
    main('./computer_science.docx')
    # main('./cp1.docx')

 
 ```