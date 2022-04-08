title: VSC Remote Container
author: tr
tags:
  - VSC
categories:
  - VSC
date: 2022-04-08 10:35:00
---
# Visual Studio Code : Remote Container


> 一款神级插件使用说明

<!--more-->

## 插件说明

> 这款插件：`Remote Containers` 可以将快速的将本地的工程目录放入独立容器中，所有的环境配置，运行，都在容器内执行，但是工程文件编辑就在vsc内，用户体验极好

> 应该更适用于开发环境版本较多的人，假如一个工程是用python2.7开发的，另一个是python3 两个同时安装在宿主机需要版本切换控制。类似于这种情况，多版本导致的冲突污染。使用该插件可以将两个工程都放入对应的容器内部避免了主机版本冲突。

> 更为强大的是很多配置都是自动化的，假如一个spring的maven项目在文件夹内，使用这款插件可以快速搭建maven环境，插件自动按照我们给的选项生成镜像容器，导入jdk，安装maven，并且自动启动java服务器，同步pom内的依赖，导入结束后只需 `F5` 启动即可。

> 一个很有特色的功能点是，假如团队合作中多人开发，某人提交了一个 `PR/MR` 不清楚这个功能点是否好用，可以将这个`PR/MR`地址复制下来，使用插件快速搭建，插件将自动拉去这个提交的代码构建环境快速启动测试

## 使用

### DEMO

> 使用官方DEMO感受插件带来的快速构建工作环境镜像，这里用的java maven工程

![upload successful](/images/pasted-159.png)

![upload successful](/images/pasted-160.png)

可以看到很快速的启动了一个镜像，可以在vsc内编辑任何代码，只要按下 `F5`就可以启动服务了。所有的环境配置包括maven仓库都存储在容器内，不会污染主机环境。

![upload successful](/images/pasted-161.png)

点击即可关闭和容器的链接


![upload successful](/images/pasted-163.png)

可以看到有很多选项，可以本地打开目录，可以打开远程仓库甚至是一个`PR/MR`请求。

### 正常使用

> 这里以java的springboot工程为例，先在本地文件夹内建立好项目文件，也可以git拉取

![upload successful](/images/pasted-162.png)

> 打开vsc的控制台，选择open folder

![upload successful](/images/pasted-166.png)

> 接下来会选择基础的jdk等开发环境设置，选好jdk，maven，os的版本即可

![upload successful](/images/pasted-164.png)

> 在构建完毕后最好安装这些官方推荐的插件，这些插件将会被安装在容器内部，用于容器内的项目开发，安装后才有不输给IntelliJIDea的开发体验

### 插件原理

> 实质上插件本身是通过我们选择时给的选项生成Dockerfile文件和一个 .devcontainer.json文件，这个文件是用于生成的镜像执行的文件包括一些环境配置。

> 在我们选择好选项后，插件生成Dockerfile并且build镜像启动容器，在这个过程中我们的工作目录会自动挂载到容器内的/vscode目录下

> 如果想先看看生成的配置文件，再执行构建，可以选择这条选项

![upload successful](/images/pasted-167.png)

> 选择目录，选择配置，生成文件，修改配置确认无误后，可以再次选择 open container 打开配置文件所在目录就会开始自动构建配置

### VSC的插件入门


![upload successful](/images/pasted-165.png)

每次打开都可以看到主页面的右下角这些start，随便点击即可看到插件的学习使用教程，推荐安装的插件每个都看一下知道怎么修改配置