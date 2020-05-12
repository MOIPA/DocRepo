title: 'git clone need sudo: socks4错误'
author: tr
tags:
  - Linux
categories:
  - Linux
date: 2019-06-25 11:04:00
---
### git clone在开了代理的情况下无法使用

原因：git 默认https使用socks4作为代理协议，shadowsock使用的是socks5协议

方法： 											
	
    git config --global http.proxy 'socks5://127.0.0.1:1080' 
