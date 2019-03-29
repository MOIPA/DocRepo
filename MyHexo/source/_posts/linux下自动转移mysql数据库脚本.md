title: linux下自动转移mysql数据库脚本
author: tr
tags:
  - linux
  - mysql
categories:
  - linux
date: 2019-03-29 14:05:00
---
使用此脚本将远程服务器的数据库导入，且显示sql文件在 back_dir="/root/databases_save

<!--more-->

1. 这是本地mysql的情况

```shell
#!/bin/bash

#实现将远程数据库sql导入到本地数据库且将sql文件存放本地目录用于以后传输

#target mysql info
mysql_host_remote="39.108.159.175"
mysql_user_remote="root"
mysql_password_remote="0800"

#local mysql info
mysql_host_local="localhost"
mysql_user_local="root"
mysql_password_local="0800"

#sql back directory
root_dir="/root"
back_dir="/root/databases_save"
data_dir="databases"
store_dir="databases"
if [ ! -d $back_dir ]; then
	mkdir -p $back_dir
fi

#备份的数据库数组
db_arr=$(echo "show databases;" | mysql -u$mysql_user_remote -p$mysql_password_remote -h$mysql_host_remote)
#不需要备份的单例数据库
nodeldb="information_schema"
nodeldb2="performance_schema"
nodeldb3="Database"


#当前日期
#date = $(date '+%Y%m%d')

date=$(date -d '+0 days' +%Y%m%d)

#zip 打包
#zipname="sql_save_"$date".zip"

#cd 到备份目录
cd $back_dir

#循环备份并且导入到本地数据库
for dbname in ${db_arr}
do
	#if [[ $dbname != $nodeldb ]] && [[ $dbname != $nodeldb2 ]] && [[ $dbname != $nodeldb3 ]]; then
	if [ $dbname == "test" ]; then
		sqlfile=$dbname-$date".sql"
		### -B 命令可以直接创建带建库语句的sql文件，但是导入必须得生命数据库 是否有不带数据库名的导入方法？
		mysqldump  --add-drop-database -u$mysql_user_remote -p$mysql_password_remote -h$mysql_host_remote -B $dbname > $sqlfile
		### 导入之前先在本地建库
		#$(echo "create database "$dbname | mysql -u$mysql_user_local -p$mysql_password_local -h$mysql_host_local)
		#mysqldump -u$mysql_user_local -p$mysql_password_local -h$mysql_host_local $dbname < $sqlfile
		### 很奇怪 只有source有效 使用mysqldump回复表不会被恢复
		$(echo "source "$back_dir"/"$sqlfile | mysql -u$mysql_user_local -p$mysql_password_local -h$mysql_host_local)
	fi
done

# .tar打包所有sql可用于以后传输
#tar -zcPpf $back_dir/$zipname --directory / $back_dir

#打包后删除sql文件
#if [ $? = 0 ]; then
#	rm *.sql
#fi

#mysqldum 到本地数据库



```

2. 如果使用docker部署本地mysql 

```shell
#!/bin/bash

#实现将远程数据库sql导入到本地数据库且将sql文件存放本地目录用于以后传输

#target mysql info
mysql_host_remote="192.168.0.100"
mysql_user_remote="root"
mysql_password_remote="123"

#local mysql info
mysql_host_local="localhost"
mysql_user_local="root"
mysql_password_local="123"

#sql back directory
back_dir="/root/mysql/data/databases_save"
source_dir="/var/lib/mysql/databases_save"
if [ ! -d $back_dir ]; then
	mkdir -p $back_dir
fi

###太坑了 公司的mysql是docker里的容器，使用所有命令前添加：  docker exec -i mysql-server
### 但是会使用source会出错，因为mysql内部容器没有权限打开外部sql文件

#备份的数据库数组
db_arr=$(echo "show databases;" | mysql -u$mysql_user_remote -p$mysql_password_remote -h$mysql_host_remote)
#不需要备份的单例数据库
nodeldb="information_schema"
nodeldb2="performance_schema"
nodeldb3="mysql"
nodeldb4="sys"
nodeldb5="Database"


#当前日期
#date = $(date '+%Y%m%d')

date=$(date -d '+0 days' +%Y%m%d)

#zip 打包
#zipname="sql_save_"$date".zip"

#cd 到备份目录
cd $back_dir

#循环备份并且导入到本地数据库
for dbname in ${db_arr}
do
	if [[ $dbname != $nodeldb ]] && [[ $dbname != $nodeldb2 ]] && [[ $dbname != $nodeldb3 ]] && [[ $dbname != $nodeldb4 ]] && [[ $dbname != $nodeldb5 ]]; then
	#if [ $dbname == "test" ]; then
		sqlfile=$dbname-$date".sql"
		### -B 命令可以直接创建带建库语句的sql文件，但是导入必须得声明数据库 是否有不带数据库名的导入方法？
		mysqldump --add-drop-database -u$mysql_user_remote -p$mysql_password_remote -h$mysql_host_remote -B $dbname > $sqlfile
		### 导入之前先在本地建库
		#$(echo "create database "$dbname | mysql -u$mysql_user_local -p$mysql_password_local -h$mysql_host_local)
		#mysqldump -u$mysql_user_local -p$mysql_password_local -h$mysql_host_local $dbname < $sqlfile
		### 很奇怪 只有source有效 使用mysqldump回复表不会被恢复
		$(echo "source "$source_dir"/"$sqlfile | docker exec -i mysql-server mysql -u$mysql_user_local -p$mysql_password_local -h$mysql_host_local)
	fi
done

# .tar打包所有sql可用于以后传输
#tar -zcPpf $back_dir/$zipname --directory / $back_dir

#打包后删除sql文件
#if [ $? = 0 ]; then
#	rm *.sql
#fi

#mysqldum 到本地数据库





```