title: Docker 迁移一些自己的服务
author: tr
tags:
  - Docker
categories:
  - Docker
date: 2022-01-18 11:07:00
---
# 迁移自己的服务
> 用docker把以前服务器上的一些服务打包推送到docker hub
<!--more-->

## 博客 hexo

> hexo是基于nodejs的，所以镜像基于node，以下是dockerfile

```shell
FROM node

MAINTAINER tangrui<1540525748@qq.com>

WORKDIR /home/hexo

RUN npm install -g hexo-cli && hexo init /home/hexo && cd /home/hexo && npm install && npm install hexo-server --save
RUN apt update && apt install -y vim && apt install -y unzip
RUN npm install --save hexo-admin 

#ADD https://codeload.github.com/MOIPA/MOIPA.github.io/zip/refs/heads/master /home/hexo/source/public
RUN ["wget","https://codeload.github.com/MOIPA/MOIPA.github.io/zip/refs/heads/master","-O","./master"]

RUN unzip -oq ./master && mkdir public
RUN mv  MOIPA.github.io-master/* ./public
RUN rm -rf MOIPA.github.io-master

RUN git clone https://github.com/theme-next/hexo-theme-next themes/next

RUN sed -i 's/landscape/next/g' _config.yml

RUN npm install hexo-migrator-rss --save

# 配置RSS订阅，提供订阅链接
RUN ["npm","install","hexo-generator-feed","--save"]
 
RUN echo "\n\
Plugins:\n\
- hexo-generate-feed\n"\
>> _config.yml 

RUN sed -i 's/#RSS/RSS/g' ./themes/next/_config.yml \
&& sed -i 's/Muse/Gemini/g' ./themes/next/_config.yml &&\
sed -i 's/url: #/url: /g' ./themes/next/_config.yml 

RUN hexo g


EXPOSE 4000 

VOLUME ["/home/hexo/"]

CMD ["hexo","server","-p","4000"]
```

> 这个版本的hexo配置了next主题，我的git上的public的gitio博客内容，和rss迁移内容

```shell
# 启动博客容器
docker run -d --name trhexo -p 5555:4000 moipa/hexo:1.4
# 迁移所有博客
docker exec -it trhexo /bin/bash

hexo migrate rss http://39.108.159.175:4000/atom.xml
```

# 配置个人的专属hexo镜像

1. 将linux的公钥私钥复制到镜像中

2. 将source/_posts文件夹改为git目录，文档从github上同步，在deploy脚本内写好git push的命令

3. 每次新电脑只需要`docker pull moipa/hexo:2.x`到本地运行即可 
