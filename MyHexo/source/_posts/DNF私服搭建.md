title: DNF私服搭建
author: tr
tags:
  - DNF
categories: []
date: 2021-09-11 17:36:00
---
# 毒奶粉私服配置

>> 偶然刷b站发现了毒奶粉的单机教学视频，突然儿时日日通宵打dnf的回忆涌上脑海，搜了下网上的配置，找了个比较纯净的版本

## 服务端文件

>> 我原以为是dnf的源代码被泄露了，其实只是服务端代码被泄露了，代码是台服的70版本，网上的魔改也是照这个版本改的

>> 在贴吧的资源里可以找到配置教程，所使用的环境是centos 5-7。打包好了资源文件和安装脚本，比较遗憾的是我不太想用centos，还是那么老的版本，如果能把服务docker化方便迁移是最好的

![upload successful](/images/pasted-82.png)

### 服务端配置

1. 按照百度贴吧的教程，上传到root目录下，`dof.tar.gz`和脚本文件`dof`，dof是个安装环境的shell脚本，看了下脚本没有装什么奇怪的东西。

2. `chmod +x dof`,`./dof`执行完毕脚本即可

3. `dof.tar.gz` 这里面是服务端php文件，会被解压到`/home/neople`目录下，大部分的ip配置都在里面如果需要替换ip 如下指令即可
```
将这个目录下所有cfg结尾的，里面含有ip的内容的文件列出，替换里面内容
sed -i "s/stun_ip=/stun_ip=172.17.0.1/g" `grep ip -rl /home/neople/game/cfg/*.cfg`
```

### 上云的问题

>> 上面人家给的脚本和配置只是针对单个虚拟机或者centos云的情况，如果是云的话还需要开放对应端口，可以下周`nmap`扫描工具，`nmap 127.0.0.1` 把里面的端口开放下即可

![upload successful](/images/pasted-83.png)

>> 更换ip的问题，这个脚本里面实际会自动检测服务器当前的ip，让tcp绑定这个ip，不过如果是虚拟机没有配置好nat或者上云要手动输入公网ip的话，可以将dof解压的目录的ip替换下

![upload successful](/images/pasted-84.png)

### docker的探索 （linux下，windows一样只不过映射目录换下：/d/...)

1. `docker pull centos:7`:下载centos7的话用这个，记住跑这个镜像的时候最好开启真正的虚拟机环境，因为docker的虚拟实质上是单个独立应用线程

```
docker run -it --net=host -p 3360:3360 -v /root/dnfServer:/root centos:7 /usr/sbin/init --privileged=true

```
`--privileged=true`:真正开启root，否则docker里面的root只是个普通用户
`-v /root/dnfServer:/root`:映射下文件，不然没法传安装文件和脚本
`/usr/sbin/init`: 启动容器之后可以使用systemctl方法
`-p 3360:3360`: 映射下端口，先映射个数据库的，实际上如果host模式不需要映射了

2. 如果你要使用centos5，那么命令需要改下

`docker run -itd --privileged=true --net=host -p 3360:3360 -v /root/dnfServer:/root centos:5 /sbin/init
`
这是因为centos5的init不在usr目录下，至于为什么需要加上`/sbin/init` 这是为了使用systemctl命令，不然 1号进程不是 init的话系统会缺失很多信息

3. 生成并且运行容器后可以查看下：`docker ps -a`

4. 将dof和dof.tar.gz存入/root/dnfServer中，然后进入容器`docker exec -it <你的容器id> /bin/bash`

5. 进入后记得cp命令将文件拷贝到/目录下，然后执行dof脚本即可

### docker的时候遇到的问题

>> 很奇怪的事是告诉我确实类库，libstd，查了下这个是gcc的动态链接库，没办法，我用的还是centos5 最老的一个版本，已经不再支持更新，yum源换了也没啥用，直接去官网下载离线包，离线安装了，centos5的包地址是`ult.centos.org/5.11/os/x86_64/` 我的版本是5.11 其他版本自行替换，ctrl+f找到gcc，glibc相关的包，下载下来上传到服务器，然后安装：`rpm -Uvh xxxx.rpm --nodeps`

>> 去容器的home目录下执行run脚本即可，然后启动游戏享受吧

## docker 的网络模式

本来在docker安装前我担心这个ip映射的问题，我在想是不是要用bridge模式，制造虚拟网卡，让宿主机和容器互通，这部分有点麻烦，配置完后游戏的客户端需要连接云的公网ip，然后再做个映射。后来发现大可不必，直接host模式就好了，所有服务端口就开在宿主机上o了。

# 总结

docker真是个好东东，这下以后服务器没了，直接把容器拷贝过去start一下就好了，多舒服，docker仓库+1。

dnf是个好游戏，但是已经不再是儿时的act动作游戏了，她已经变质了，不再可能有白板装备技术流单刷老牛的情况了，从70版本男法出现后dnf就是个二次元动画rpg快餐游戏了，纪念逝去的青春。


![upload successful](/images/pasted-85.png)