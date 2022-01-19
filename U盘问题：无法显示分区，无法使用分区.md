title: U盘问题：无法显示分区，无法使用分区
author: tr
tags:
  - Usb
  - Log
categories:
  - Log
date: 2019-05-02 11:37:00
---
### U盘问题：清理分区

###### 在公司使用U盘+USBWriter安装centOS后导致U盘分区无法使用，显示只有安装大小

#### 解决方法：

1. 清理分区：

       .win+r，打开运行窗口，输入cmd，出现dos运行环境，输入diskpart，回车，弹出另一个命令窗口，然后将在上面（1）中的索引，输入select disk 1 进去，回车，输入clean，之后即可将空间内容清除（disk 编号可以再系统磁盘管理器中找到）

2. 创建分区：
	打开磁盘管理器，建立即可
