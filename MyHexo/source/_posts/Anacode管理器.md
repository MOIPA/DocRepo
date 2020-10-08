title: Anaconda管理器
author: tr
tags:
  - DataAnalysis
categories:
  - DataAnalysis
  - ''
date: 2019-06-12 09:51:00
---
### Anaconda管理器

说明：使用python做大数据时需要的工具,配置后可以使用conda
<!--more-->

#### 安装

win10环境

1. 打开官网下载安装包
2. 使用时右击管理员下打开提供的终端


#### 使用conda

##### 简单命令
1. 管理员下打开anaconda终端
2. conda list 查看已安装包
3. conda upgrade    --all  更新所有包
4. conda install package_name 安装包
5. conda remove package_names 卸载包
6. conda update package_name 更新包
7. conda search keywork 搜索包

##### 使用环境

不同的项目可以建立不同的环境，并且可以将环境导出为配置文件，在部署到其他服务器时读取配置文件即可自动安装依赖

1. conda create -n env_name package_names ：创建env_name环境且安装 package_names包

2. conda create -n env_name python=3 : 创建env_name环境且指定环境为python3，会自动为环境安装

3. activate my_env 进入环境（windows）

4. deactivate 离开环境

5. conda env export > environment.yaml  导出环境配置

6. conda env update -f=/path/to/environment.yml 导入配置，自动安装配置

7. conda env list 列出环境

8. conda env remove -n env_name 删除环境

