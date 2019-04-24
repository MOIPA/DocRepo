title: python解析习题集
author: tr
tags:
  - python
categories: []
date: 2019-04-11 09:52:00
---
### 使用python解析txt题目文本

 公司需求，解析一本习题集，将所有的单选题和答案解析放在一个对象，转成json输出
 
 <!--more-->
 
 ```python
 
 # -*- coding: utf-8 -*-
import re
import sys
import os,glob
import json
import shutil

class Option:

	def __init__(self,options):
		self.options=options
	def display(self):
		print self.options

class Question:

	def __init__(self, qid, item, options, answer, point, analysis):	
		self.qid=qid
		self.item=item
		self.option=Option(options)
		self.answer=answer
		self.point=point
		self.analysis=analysis

	def display(self):
		print str(self.qid)+":"+str(self.item)+str(self.answer)+str(self.point)+str(self.analysis)
		self.option.display()

# execute the parse program, transfer topic into question object,then put add it to the list
def parseQuestionFile(questionFileName,questionArray):
	# flag ,used to distinguish the parsing statment 0:means parsing options 1:means parsing topic
	flag=-1
	tempString="";
	fout = open('all_single_choice_quesion','a+')
	fs = open(questionFileName,'r+')
	# initialization
	options=[]
	for line in fs.readlines():
		# remove chinese "quanjiao"
		line = line.replace('\xa1\xa1', '')

		# match empth line
		if re.match(r'^\s*$', line):
			continue

		# match item such as text begining with 1.or 12 .etc
		m = re.search(r'\s*[0-9]+\.\s*(.*)$', line)
		# Match is it is an option
		m_op =  re.search(r'\s*[A|B|C|D]+\.\s*(.*)$', line)
		if m or m_op==None:
			if m:
				if flag==0:
					#db
					# encountered another topic time to finish this question filling
					question.option=options
					questionArray.append(question)
					options=[]
					tempString=""
				# get question id
				qid = re.search(r'\s*([0-9]*)\.\s*(.*)$', line).group(1)
				question=Question(qid,'def','','def','def','def')
				# reset to 1 starting filling topic
				flag=1
				#fout.write(qid+': item:\n'+m.group(1))
				tempString+='item:\n'+m.group(1)
			if m_op==None and flag==1:
				fout.write(line)
				tempString+=line
		else:
			# match options
			if flag==1:
				# db
				fout.write('\nOptions:\n')
				question.item=tempString
				tempString=""
				flag=0
			res = re.split(r'\s*[A|B|C|D]\s*\.\s*', line.strip())

			for elem in res:
				if elem:
				#	print "\nelem****"+elem+"***elem\n"
					options.append(elem)
					fout.write('\t' + elem + '\n')
	# end finish appending
	question.options=options
	questionArray.append(question)
	fout.write('\n')
	fout.close()

# used to merge data to tpoic
def appendAnswer2Topic(questionArray,qid,answer,point,analysis):
	i=0
	while i<len(questionArray):
		if questionArray[i].qid==qid:
			questionArray[i].answer=answer
			questionArray[i].point=point
			questionArray[i].analysis=analysis
		i+=1

# parse answer,the problem is answer and point have only one line except analysis which may occupy many lines,but still give all of them status flag for the further change
def parseAnswerFile(answerFileName,questionArray):
	fs = open(answerFileName,'r+')

	# status to identify the operation status,-1 no, 1 answer, 2 point, 3 analysis , 4 nomal
	status=-1;

	for line in fs.readlines():
		# remove chinese "quanjiao"
		line = line.replace('\xa1\xa1', '')
	
		# match empth line
		if re.match(r'^\s*$', line):
			continue
		
		# match answer id line
		aidm = re.match(r'\s*([0-9]*)[\.|•]',line)
		# match answer 	
		answerm = re.match(r'\s*[0-9]*\.【答案】([a-zA-Z]?)',line)
		# match point 
		pointm = re.match(r'\s*【考点】(.*)',line)	
		# match analysis
		analysism = re.match(r'\s*【解析】(.*)',line)	
	
		# judge status
		if aidm:
			# encounter the start, handle result if it is exist
			if status != -1:
				# append operation
				appendAnswer2Topic(questionArray,aid,answer,point,analysis)
				fout = open('all_single_choice_answer','a+')
				fout.write(aid)
				fout.write('\n'+answer)
				fout.write('\n'+point)
				fout.write('\n'+analysis+'\n')
				fout.close()
			aid=aidm.group(1)
			if answerm:
				answer = '[answer]'+answerm.group(1)
			status=1
		elif pointm:
			point='[point]'+pointm.group(1)
			status=2
		elif analysism:
			analysis='[analysis]'+analysism.group(1)
			status=3
		else :
			if status==3:
				analysis+=line
			if status!=-1:
				status=4
	# end finish appending
	appendAnswer2Topic(questionArray,aid,answer,point,analysis)
			
	
# parse topic and answer operation
def parseTopicAndAnswer(questionFileName, answerFileName, outFileName):
	# define data
	questionArray = []
	# start parse
	parseQuestionFile(questionFileName, questionArray)
	parseAnswerFile(answerFileName, questionArray)
	#show(questionArray)
	#fout=open(outFileName,'a')
	convert(questionArray)

# show result
def show(questionArray):
	for ques in questionArray:
	 	ques.display()
	 	for opt in ques.options:
	 		print opt
	 	print '\nquestion is over\n'
	
# convert to json
def convert(questionArray):
	fout=open('out_json','a+')	
	for ques in questionArray:
	 #	for opt in ques.options:
			#opt=opt[1:]
		#	print opt
		fout.write(json.dumps(ques, default=lambda o: o.__dict__,sort_keys=True, indent=4,ensure_ascii=False))
	fout.close()
	

# parse all full book into several files(single choice question)
def parseBookSingleQuestion(path,bookName):

	# define question and answer file name array
	questionFileNameArray=[]
	answerFileNameArray=[]

	fs=open(bookName,'r+')
	bm=re.match(r'\s*(.*)\..*','computer_science.txt').group(1)
	# delete directory
	if os.path.exists(bm+'_single_choice')==True:
		shutil.rmtree(bm+'_single_choice')
	print 'create directory'
	os.mkdir(bm+'_single_choice')
	os.chdir(bm+'_single_choice')
	# status to indetify parsing status, 0 enter in chapter ,1 enter chapter single choice question , 2 enter in chapter fill question , 3 enter in answer single choice, 4 enter in answer single fill
	writeStatus=-1
	# if it is paring into answer section
	answerStatus=-1
	i=1
	for line in fs.readlines():
		# remove chinese "quanjiao"
		line = line.replace('\xa1\xa1', '')
	
		# match empth line
		if re.match(r'^\s*$', line):
			continue
		# match chapter
		chapterIdm=re.match(r'\s*第(.*)章\s*',line)
		scqm=re.match(r'\s*单项选择题\s*',line)
		fqm=re.match(r'\s*填空题\s*',line)
		anm=re.match(r'\s*参考答案与解析\s*',line)
		
		# status judge
		if chapterIdm:
			answerStatus=0
			chapterId=chapterIdm.group(1)
		if scqm:
			writeStatus=1;
			if answerStatus!=1:
				fout=open('single_choice_question_'+chapterId+'.data','w')
				questionFileNameArray.append('single_choice_question_'+chapterId+'.data')
				
			elif answerStatus==1:
				fout=open('single_choice_answer_'+chapterId+'.data','w')
				answerFileNameArray.append('single_choice_answer_'+chapterId+'.data')

		# enter in fill question means the single question part is over
		if fqm:
			writeStatus=-1
			answerStatus=-1
			fout.write('\n')
			fout.close() 
		if anm:
			answerStatus=1
		# normal line write
		if writeStatus==1:
			fout.write(line)
		
	return (questionFileNameArray,answerFileNameArray)

# main first parse into several files  then ..
def mainParse():
	# path
	path = './'
	os.chdir(path)
	# declare the file 
	outFile='out.txt'
	bookName='computer_science.txt'

	
	# start parse 
	# parseBook into several files and get these filenames
	qfna,afna=parseBookSingleQuestion(path,bookName)
	while qfna:
		parseTopicAndAnswer(qfna.pop(),afna.pop(),outFile)


mainParse()

 
 ```
 