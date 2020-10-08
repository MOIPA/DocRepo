title: AWK
author: tr
tags:
  - AWK
categories:
  - Linux
date: 2019-05-30 10:57:00
---
### AWK

非常好用的命令行工具，可以替代grep，支持正则

#### 常用

通过 pattern+action方式运行

输出第一个和第二个域，只存在action。每个域默认是通过空格分离的。

	ll |awk '{print $3}'

添加了pattern -F ':' 表示使用冒号分割，输出第一个和第二个域。

	cat /etc/passwd|awk -F ':'  '{print $1 "\t" $2}'
    
直接查文件：

	awk -F ':'  '{print $1 " " $2}' /etc/passwd
    
所以可以直接作为搜索使用 -F: ''表示搜索pattern,注意格式:'\内容\'

	awk -F: ’/root/'
    
支持正则
	
	awk -F: ’/^root/'
    
 支持变量：
 
 	统计某个文件夹下的文件占用的字节数
    ls -l |awk 'BEGIN {size=0;} {size=size+$5;} END{print "[end]size is ", size}'
    
高级用法以后用到再说…………


	