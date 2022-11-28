title: DNF docker部署
author: tr
tags:
  - Docker
categories:
  - Docker
date: 2022-09-28 01:17:00
---
# DNF的部署

和上一篇虚拟机部署一样，本质是把泄露的服务端文件部署到centos服务器，只不过这次的虚拟机换位了docker更加方便

<!--more-->

# Linux下的部署

> 可以直接使用`DockerHub`提供的镜像

```shell
docker pull 1995chen/dnf:stable
# 两种启动方式
# docker-compose方式，复制项目的docker配置
docker-compose up init-dnf
docker-compose up -d dnf
```

> 这里面的配置项目很多，但是主要也就改个数据库密码`volumes`下面映射三个目录，第一个data目录需要里面存放对应客户端的`Script.pvf`文件，一开始没有也没关系，镜像内携带了一个版本的pvf文件，到时候可以复制到客户端，或者客户端复制到data内覆盖，重启镜像

```shell
version: "2.3"

networks:
  local:
    ipam:
      config:
        - subnet: 172.20.0.0/16

services:
  init-dnf:
    image: 1995chen/dnf:stable
    environment:
      - TZ=Asia/Shanghai
      # 数据库root密码[首次初始化会修改root密码]
      - DNF_DB_ROOT_PASSWORD=88888888
    command: ['/bin/bash', '/home/template/init/init.sh']
    networks:
      - local
    restart: "no"
    volumes:
      - ./data:/data
      - ./mysql:/var/lib/mysql
  dnf:
    image: 1995chen/dnf:stable
    depends_on:
      - init-dnf
    environment:
      - TZ=Asia/Shanghai
      # 这里填写你的IP地址
      - PUBLIC_IP=127.0.0.1
      # 数据库root密码
      - DNF_DB_ROOT_PASSWORD=88888888
      - GM_ACCOUNT=gm_user
      - GM_PASSWORD=gm_pass
    shm_size: 8g
    memswap_limit: -1
    mem_limit: 1g
    cpu_count: 1
    networks:
      - local
    restart: always
    ports:
      - 3000:3306/tcp       # mysql 数据库
      - 7600:7600/tcp       # DnfGateServe 网关
      - 881:881/tcp         # DnfGateServe 网关
      - 20303:20303/tcp     # df_dbmw_r
      - 20303:20303/udp     # df_dbmw_r
      - 20403:20403/tcp     # df_dbmw_r
      - 20403:20403/udp     # df_dbmw_r
      - 40403:40403/tcp     # df_manager_r
      - 40403:40403/udp     # df_manager_r
      - 7000:7000/tcp       # df_bridge_r
      - 7000:7000/udp       # df_bridge_r
      - 7001:7001/tcp       # df_channel_r
      - 7001:7001/udp       # df_channel_r
      - 7200:7200/tcp       # df_relay_r
      - 7200:7200/udp       # df_relay_r
      - 10011:10011/tcp     # df_game_r
      - 31100:31100/tcp     # df_community
      - 30303:30303/tcp     # df_monitor_r
      - 30303:30303/udp     # df_monitor_r
      - 30403:30403/tcp     # df_guild_r
      - 30403:30403/udp     # df_guild_r
      - 10052:10052/tcp     # df_game_r
      - 20011:20011/tcp     # df_game_r
      - 20203:20203/tcp     # df_dbmw_r
      - 20203:20203/udp     # df_dbmw_r
      - 30703:30703/udp     # df_coserver_r
      - 11011:11011/udp     # df_game_r
      - 2311-2313:2311-2313/udp   # df_stun_r
      - 30503:30503/udp     # df_statics_r
      - 11052:11052/udp     # df_game_r
    volumes:
      - ./data:/data
      - ./mysql:/var/lib/mysql
      - ./log:/home/neople/game/log
```
# windows下部署（直接使用我的镜像）

> windows下无法使用其他人镜像，我自己已经构建好了一个，可以直接在windows下使用
>
> 1. `docker pull moipa/dnf` 获取镜像，可以修改docker-compose文件的配置，这里注意修改的volumes目录，会映射最重要的数据库和pvf文件目录
>
> 2. 去云盘内下载DNF的客户端，推荐龙鸣的60版本和75版本客户端
>
> 3. dockerCompose文件配置见linux下部署的那个文件，切换到这个目录后执行：`docker-compose up init-dnf` 执行初始化目录的工作，执行后挂载的几个目录会生成三个文件夹`data`,`log`,`mysql`，分别存放pfv和日志以及数据库文件，这里的数据库文件就相当于存档了，可以拷贝其他数据库文件覆盖
>
> 4. 执行`docker-compose up -d dnf` 启动服务器 
>
> 5. 如果日后需要保存存档，保存mysql文件即可，加载存档只需将mysql文件覆盖过去
>
> 6. 选择自己喜欢的客户端版本，将其中的script.pvf文件复制到data内覆盖，重启即可`docker restart 镜像名`
>
> 7. 生成游戏的登录器，下载云盘内的统一网关在线管理工具，编辑`线路名称：随意` `游戏地址：127.0.0.1 或者填服务器的地址` `登录器端口：7600` `网关地址：127.0.0.1 或者服务器` `通信秘钥：763WXRBW3PFTC3IXPFWH`，完成后点击生成登录器，将登录器复制到游戏目录下运行即可登录游玩
>
> 8. 如果发现频道是黑的，可能内存不足导致，重启镜像，或者进入镜像重启服务

```text
默认的网关信息
网关端口: 881
通讯密钥: 763WXRBW3PFTC3IXPFWH
登录器版本: 20180307
登录器端口: 7600
GM账户: gm_user
GM密码: gm_pass
```

# windows下的部署（需要自建镜像）

> windows下直接使用别人的镜像会导致一系列问题，本质在于windows使用的是wsl（内置的一个linux子系统），google了下由于docker设计问题，基于wsl的容器启动后无法使用`systemctl`指令
>
> 当然这时候可以选择不在linux下部署，自己安装mysql、php服务器也可以，但是更麻烦
>
> `stackOverFlow`提供了指令无法执行的解决方案

1. 下载 git上 `1995chen/dnf`的项目，这个项目比较全，提供了服务器文件，等级补丁文件，登录器文件以及`centos`基础镜像和环境包的安装配置。

2. 改docker镜像的构建思路是从构建centos环境的基础镜像到导入DNF服务器文件的DNF镜像，下面从基础重新构建

   > 1. 准备好构建一个国内源的centos镜像
   >
   >    ```shell
   >    # Base Image
   >    FROM centos:7.9.2009
   >
   >    MAINTAINER 1995chen
   >
   >    # 删除旧源
   >    RUN cd /etc/yum.repos.d/ && \
   >        rm -rf CentOS-Base.repo && \
   >        sed -i "s|enabled=1|enabled=0|g" /etc/yum/pluginconf.d/fastestmirror.conf
   >    # 添加源
   >    ADD build/Centos7/CentOS-Base.repo /etc/yum.repos.d/
   >    # 更新Repo
   >    RUN yum clean all && yum update -y && yum install -y psmisc openssl openssl-devel curl wget perl vim && \
   >        yum clean all
   >    # 切换工作目录
   >    WORKDIR /root
   >    ```
   >
   > 2. 这是镜像的国内源 CentOS-Base.repo：
   >
   >    ```shell
   >    # CentOS-Base.repo
   >    #
   >    # The mirror system uses the connecting IP address of the client and the
   >    # update status of each mirror to pick mirrors that are updated to and
   >    # geographically close to the client.  You should use this for CentOS updates
   >    # unless you are manually picking other mirrors.
   >    #
   >    # If the mirrorlist= does not work for you, as a fall back you can try the
   >    # remarked out baseurl= line instead.
   >    #
   >    #
   >    
   >    [base]
   >    name=CentOS-$releasever - Base - mirrors.aliyun.com
   >    failovermethod=priority
   >    baseurl=http://mirrors.aliyun.com/centos/$releasever/os/$basearch/
   >            http://mirrors.aliyuncs.com/centos/$releasever/os/$basearch/
   >            http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/
   >    gpgcheck=1
   >    gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
   >    
   >    #released updates
   >    [updates]
   >    name=CentOS-$releasever - Updates - mirrors.aliyun.com
   >    failovermethod=priority
   >    baseurl=http://mirrors.aliyun.com/centos/$releasever/updates/$basearch/
   >            http://mirrors.aliyuncs.com/centos/$releasever/updates/$basearch/
   >            http://mirrors.cloud.aliyuncs.com/centos/$releasever/updates/$basearch/
   >    gpgcheck=1
   >    gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
   >    
   >    #additional packages that may be useful
   >    [extras]
   >    name=CentOS-$releasever - Extras - mirrors.aliyun.com
   >    failovermethod=priority
   >    baseurl=http://mirrors.aliyun.com/centos/$releasever/extras/$basearch/
   >            http://mirrors.aliyuncs.com/centos/$releasever/extras/$basearch/
   >            http://mirrors.cloud.aliyuncs.com/centos/$releasever/extras/$basearch/
   >    gpgcheck=1
   >    gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
   >    
   >    #additional packages that extend functionality of existing packages
   >    [centosplus]
   >    name=CentOS-$releasever - Plus - mirrors.aliyun.com
   >    failovermethod=priority
   >    baseurl=http://mirrors.aliyun.com/centos/$releasever/centosplus/$basearch/
   >            http://mirrors.aliyuncs.com/centos/$releasever/centosplus/$basearch/
   >            http://mirrors.cloud.aliyuncs.com/centos/$releasever/centosplus/$basearch/
   >    gpgcheck=1
   >    enabled=0
   >    gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
   >    
   >    #contrib - packages by Centos Users
   >    [contrib]
   >    name=CentOS-$releasever - Contrib - mirrors.aliyun.com
   >    failovermethod=priority
   >    baseurl=http://mirrors.aliyun.com/centos/$releasever/contrib/$basearch/
   >            http://mirrors.aliyuncs.com/centos/$releasever/contrib/$basearch/
   >            http://mirrors.cloud.aliyuncs.com/centos/$releasever/contrib/$basearch/
   >    gpgcheck=1
   >    enabled=0
   >    gpgkey=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
   >    
   >    ```
   >
   > 3. 构建该镜像：`docker build -t tr/centos .`

3. 继续构建下一个带有环境的`centos`前需要先准备解决下`systemctl`无法执行的问题

   >1. 下载python编写的systemctl同功能脚本：`  curl https://raw.githubusercontent.com/gdraheim/docker-systemctl-replacement/master/files/docker/systemctl.py `
   >
   >2. 下载完成后放入`Centos7-DNF-Base`内，和其他数据库等软件一起
   >
   >3.  在构建的`Dockerfile`内准备好配置，改配置已经放在第四步的`Dockerfile`内了：
   >
   >   ```dockerfile
   >     # 适配windows的systemctl
   >     ADD ./systemctl.py /usr/bin
   >     RUN mv /usr/bin/systemctl /usr/bin/systemctl.old
   >     RUN cat /usr/bin/systemctl.py > /usr/bin/systemctl
   >     RUN chmod +x /usr/bin/systemctl
   >   ```
   >
   >   

4. 准备好`mysql`数据库，`GeoIP`和一些`lib`包，`python`包工具，构建一个带有环境的`centos`

   >1. 准备包可以直接使用项目提供的包在`Centos7-DNF-Base`下，在该目录执行构建
   >3. `Dockerfile`配置：
   >
   >  ```dockerfile
   >  # Base Image
   >  FROM tr/centos
   > 
   >   # 安装mysql依赖
   >   ADD ./MySQL-shared-compat-5.0.95-1.rhel5.x86_64.rpm /tmp
   >     ADD ./MySQL-devel-community-5.0.95-1.rhel5.x86_64.rpm /tmp
   >   ADD ./MySQL-client-community-5.0.95-1.rhel5.x86_64.rpm /tmp
   >   ADD ./MySQL-server-community-5.0.95-1.rhel5.x86_64.rpm /tmp
   >   # 编译GeoIP
   >   ADD ./GeoIP-1.4.8.tgz /home
   >   # 添加依赖库
   >   ADD ./lib.tgz /lib
   > 
   >   # 更新Repo
   >   RUN yum update -y && yum install -y gcc gcc-c++ make zlib-devel libc.so.6 libstdc++ libstdc++.so.6 glibc.i686 libssl.so.6 xulrunner.x86_64 libXtst.x86_64 initscripts && \
   >         ln -sf /usr/lib64/libssl.so.10 /usr/lib64/libssl.so.6 && ln -sf /usr/lib64/libcrypto.so.10 /usr/lib64/libcrypto.so.6 && \
   >       rpm -ivh /tmp/MySQL-shared-compat-5.0.95-1.rhel5.x86_64.rpm && \
   >       rpm -ivh /tmp/MySQL-devel-community-5.0.95-1.rhel5.x86_64.rpm && \
   >       rpm -ivh /tmp/MySQL-client-community-5.0.95-1.rhel5.x86_64.rpm && \
   >       rpm -ivh /tmp/MySQL-server-community-5.0.95-1.rhel5.x86_64.rpm && service mysql stop && \
   >       rm -rf /var/lib/mysql/* && cd /home/GeoIP-1.4.8/ && chmod 777 configure && ./configure && make && make install && yum clean all
   > 
   >   # 适配windows的systemctl
   >   ADD ./systemctl.py /usr/bin
   >     RUN mv /usr/bin/systemctl /usr/bin/systemctl.old
   >   RUN cat /usr/bin/systemctl.py > /usr/bin/systemctl
   >   RUN chmod +x /usr/bin/systemctl
   > 
   >   # 切换工作目录
   >   WORKDIR /root
   >  ```
   > 
   > 3. 执行构建：`docker build -t tr/centos7`

5. 最后构建DNF镜像，即把DNF服务器文件加载执行

   >1. 进入项目提供的`Centos7-DNF`文件夹内，可以看到`DNF`的配置文件和服务文件，不知道是不是原始流出文件，这里主要是两个文件可以看看是怎么编写的`docker-entrypoint.sh` `init.sh`
   >
   >2. 编写`Dockerfile`
   >
   >  ```dockerfile
   >  # Base Image
   >  FROM tr/centos7
   >  
   >  MAINTAINER tr
   >  
   >  # 定义默认环境变量
   >  ENV AUTO_PUBLIC_IP=false
   >  ENV PRELOAD_LD=true
   >  ENV PUBLIC_IP=127.0.0.1
   >  ENV GM_ACCOUNT=gm_user
   >  ENV GM_PASSWORD=gm_pass
   >  ENV GM_CONNECT_KEY=763WXRBW3PFTC3IXPFWH
   >  ENV GM_LANDER_VERSION=20180307
   >  ENV DNF_DB_ROOT_PASSWORD=88888888
   >  ENV DNF_DB_GAME_PASSWORD=uu5!^%jg
   >  
   >  # 将模板添加到模版目录下[后续容器启动需要先将环境变量替换,再将文件移动到正确位置]
   >  ADD ./neople /home/template/neople
   >  ADD ./root /home/template/root
   >  ADD ./privatekey.pem /home/template/init/
   >  ADD ./publickey.pem /home/template/init/
   >  # 初始化sql脚本
   >  COPY ./init_sql.tgz /home/template/init/
   >  # 初始化版本文件
   >  COPY ./Script.tgz /home/template/init/
   >  # 初始化等级补丁
   >  ADD ./df_game_r /home/template/init/
   >  # 初始化脚本
   >  ADD ./init.sh /home/template/init/
   >  # 网关配置文件
   >  ADD ./Config.ini /home/template/init/
   >  # 启动脚本
   >  ADD ./docker-entrypoint.sh /
   >  # TEA算法
   >  ADD ./TeaEncrypt /
   >  # 优化CPU
   >  ADD ./libhook.so /lib/
   >  # 放开tmp目录权限
   >  WORKDIR /tmp
   >  RUN chmod -R 777 /tmp
   >  # 该目录用于存放版本文件
   >  RUN mkdir /data
   >  # 切换工作目录
   >  WORKDIR /root
   >  CMD ["/bin/bash", "/docker-entrypoint.sh"]
   >  ```
   >
   >
   >
   >3. 执行构建：`docker build -t tr/dnf .`

## 踩坑的点

下载`systemctl.py`的时候，最好用curl直接下载，如果点击页面直接复制代码的话，也别在windows的编辑器弄，上传到linux的时候用`head systemctl.py`打开看开头总有两个未知字符导致文件执行失败，可以用xshell等软件在vim中打开在下载到windows。