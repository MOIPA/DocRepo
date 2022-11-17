title: Arduino开发-入门配置
author: tr
tags:
  - Arduino
categories:
  - Arduino
date: 2021-11-17 03:20:00
---
# Arduino开发 入门配置

> 单片机开发，得到了一块ESP8266芯片，可以选择micropython或者arduino的系统环境，虽然micropython使用python作为开发语言，简单很多，但是生态和库没有arduino丰富，这里选择了烧录arduino环境到芯片

<!--more-->

## 下载arduino的开发ide

> 下载地址：https://www.arduino.cc/
>
> 如果和我一样使用linux系统，可以通过版本自带管理器下载，以我的archlinux为例：`pacman -S arduino`

## 配置开发板

> 打开file -> preferences 
>
> 在开发板管理器网址填（Additional Boards Manager URLs）：`http://arduino.esp8266.com/stable/package_esp8266com_index.json`

> 重启IDE环境
>
> 找到tools下 开发板选项->开发板管理器 (Boards Manager)
>
> 搜索esp8266，点击下载即可，完成后可选择esp8266的板子了


![upload successful](/images/pasted-187.png)


![upload successful](/images/pasted-188.png) 

> 插入esp开发板后，这时候会多一个串行端口，点击tools下的port选项选择该端口即可使用实例了
>
> 关于端口，如果是在windows下可以直接用设备管理器查看端口，如果是linux下需要`sudo chmod +x /dev/ttyUSB0` 给新设备赋予权限

## VSC下开发

> 这里更推荐使用VSC替代arduino ide做开发，VSC插件丰富，代码校验等
>
> 在VSC下安装以下几个插件

![upload successful](/images/pasted-189.png)

> 安装完毕后可以打开`.ino`的arduino文件，看到如下所示的状态栏，可以快速选择板子类型，开发板端口和打开串口监听
>
> 也可以使用F1呼出命令栏，输入arduino，可以配置实例，管理库等

![upload successful](/images/pasted-190.png)