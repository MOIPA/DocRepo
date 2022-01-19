title: CompanyDailyProblem
author: tr
tags: []
categories: []
date: 2019-03-28 10:00:00
---
questions：
<!--more-->
	1. AssertUtil.assertNotNull 是公司的api，抛出的异常去哪里处理了？

	3. @Enumerated 和@Type 自定义type的区别
		为什么用来@Type可以让里面的value随自己的改变改动
		
	4. 坑爹的jpa： 不允许mysql使用驼峰，否则自己主动解析为下划线，所以推荐在mysql中使用下划线
	
	5.Pageable的pageRequest是从0开始的
	
	6.使用@query和pageable一起使用的时候，会在执行query结果后使用分页
	
	
	sonarytooken: 205dd880c9542453404554bbb38da09515980275
	
	
	构建到exam_system_front
	
	
	7.源图的项目分布：	nexus服务器 放置公司jar包
						docker部署测试环境jenkins
						docker配合jenkins自动部署每个模块的image
						gitlab服务器 放置公司代码 也是jenkins拉取数据的源
						jira服务器 文档等设定
						
	8.今日问题：公司准备把测试环境从100服务器转移到99服务器 步骤如下
				1.先在99服务器开始配置自动化部署和测试环境docker和jenkins，
					jenkins是基于docker安装的，其中一些代码审查工具，maven，jdk等要在jenkins容器配置好
					比如sonarQube得在容器安装后，配置jenkins插件。。。
				
				2.jenkins自动部署步骤：首先在gitlab中有一个账号，在jenkins中通过添加这个账号形成拉取凭据
					每次拉取后执行一下脚本或者maven命令，大部分自动部署项目都是maven，直接执行maven自己的命令即可
					比如maven clean package，会自动生成/target/*.war包
					下一步就是把这个生成的包推送到99号服务器的一个目录下，这个目录下有打包成镜像的文件。配置jenkins的publish ssh
					写好推送的文件和推送后执行的脚本。
					在推送过去后，jenkins执行这个脚本，这个脚本把推送过来的war包打成docker镜像文件。
					这样可以直接运行这些服务器了
				
				3.在安装了jenkins和一些环境后，出先了一个问题：jenkins中拉取的数据源的数据库配置是100号服务器，
					但是需要转到99号服务器的测试数据库，jenkins拉取的是master分支