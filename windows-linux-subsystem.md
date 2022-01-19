title: windows+linux subsystem
author: tr
date: 2020-01-31 13:33:59
tags:
---
### windows下的linux子系统

surface book的manjaro双系统体验糟糕，退而求其次使用子系统

<!--more-->
#### 安装linux

1. 控制面板中的：启用功能中，启动linux子系统
2. 自带市场安装一个linux版本
3. 启动安装的linux，进入换源，/etc/apt/source.list 切换对应版本的国内源

	sudo apt update
    
#### windows下gui准备
    
1. 安装vcxrv（远程软件）https://sourceforge.net/projects/vcxsrv/

2. 启动xlaunch，displaynumber设置0 ，windows给的子系统不支持gui，只能通过远程方式c/s模式了。

![upload successful](/images/pasted-13.png)
#### linux下准备

1. 选择一个桌面环境，gnome或者xfce，装哪个都可以

2. 设置输出桌面 `echo "export DISPLAY=:0.0" >> ~/.bashrc`

3. 修改中文环境语言 `echo "LANG=zh_CN.UTF-8" >> ~/.profile`

4. 环境启动配置，这里使用了xfce4

![upload successful](/images/pasted-14.png)


5. `xfce4-session` 启动界面，输出到windows下的xlaunch


![upload successful](/images/pasted-15.png)

