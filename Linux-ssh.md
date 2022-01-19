title: Linux ssh scp
author: tr
tags:
  - Linux
categories:
  - Linux
date: 2020-09-04 09:54:00
---
#### linux ssh

1. 普通远程登录：`ssh -p [port] [user@ip]` eg:`ssh -p 22 root@39.108.xx.xx`

2. 免密登录： `ssh-copy-id -p [port] [user@ip] ` eg:`ssh-copy-id -p 22 root@39.108.159.175 `

	拷贝本机的ssh-key-gen生成的key即可

写了一个简单的脚本:

```shell
 1 #!/bin/bash
  2 #--------------------------------------------
  3 # 连接远程服务器脚本
  4 # author：tr
  5 # date：2020-09-04
  6 #--------------------------------------------
  7 ##### 服务器配置 开始#####
  8 remote=(
  9     root@39.108.xx.xx                                                                         
 10 )
 11 
 12 #### 服务器配置结束 #####
 13 echo '***********select ecs**************'
 14 num=1
 15 for ip in ${remote[*]}
 16 do
 17     echo 'ip select '$num':' ${ip}
 18     num=`expr $num + 1`
 19 done
 20 read -p 'Enter remote machine number : ' num
 21 select_ip=${remote[`expr $num - 1`]}
 22 echo 'connecting to : '$select_ip '...'
 23 ssh -p 22 $select_ip

```

#### scp

 下载文件:`scp user@host:<文件路径> <本地路径>` eg:`scp root@39.108.xxx.xxx:~/test ~/home/tr`
 
 下载文件夹`scp -r user@host:<文件夹路径> <本地路径>` 