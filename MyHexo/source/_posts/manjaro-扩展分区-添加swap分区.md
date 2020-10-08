title: manjaro 扩展分区+添加swap分区
author: tr
tags:
  - linux
categories:
  - linux
date: 2019-04-25 09:22:00
---
### archlinux--》manjaro 分区问题

由于linux系统的空间不足，去windows系统下面压缩了8个g，感觉windows下面的软件估计都不怎么用得到了，linux太方便了，如果用一个月，windows用不到就删了买个ipad写office

<!--more-->

1. 分区问题：使用kde自带的kde分区管理器，或者使用GParted，如果没有ide，使用fdisk来创建交换分区（假设 /dev/sdb2 是创建的交换分区）

2. 建立分区，右键建立，记住建立的设备名：sda8这样的

3. 建立交换分区：使用 mkswap 命令来设置交换分区：
　　
  		# mkswap /dev/sdb2
    
    启用交换分区：
    
		 # swapon /dev/sdb2


最后去检查/etc/fstab内是否写入了(这是boot时启动的信息):

/dev/sda8 swap swap defaults 0 0

