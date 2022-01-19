title: jupyter_notebook
author: tr
tags:
  - DataAnalysis
categories:
  - DataAnalysis
date: 2019-06-12 11:04:00
---
### jupyter_notebook

说明：将代码文档集中的工具

<!--more-->

#### 安装

1. 首先安装anaconda，因为它自带
2. 在终端下输入：conda install jupyter notebook


#### 使用

1. 如果使用环境，在环境中使用notebook。可以在外部环境下看到多出的带环境名的notebook，可以点击启动

![upload successful](/images/pasted-8.png)

2. 可以在终端启动（推荐），终端启动那么notebook的目录就是当前目录：jupyter notebook

3. 打开浏览器后可以在浏览器中新建env然后编写代码

##### jupyter_notebook结合环境env

目的：能够自动切换环境，不同环境不同目录

1. 安装环境结合库:conda install nb_conda

##### 代码自动补全

1. 包：conda install pyreadline
2. 在notebook中写时直接按下tab可以提示代码