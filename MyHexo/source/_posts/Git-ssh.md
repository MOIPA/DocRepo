title: Git ssh
author: tr
tags:
  - Git
categories:
  - Git
date: 2020-01-31 12:45:00
---
### git ssh

 迁移东西到新电脑了，好多指令和工具忘记了，记录……
 
 <!--more-->
 
 #### windows
 
 1. 下载 git for windows
 2. 默认的vim即可，不过也支持vsc，也不错
 3. 生成sshkey，ssh-keygen -t rsa -C "email"
 4. 默认c/../usr/.ssh/...pub 导入github控制
 
![upload successful](/images/pasted-12.png)

5. 测试：ssh -T git@github.com

6. 配置全局：

	``` git
    git config --global user.name "Firstname Lastname" 
    git config --global user.email "your_email@youremail.com"

   ```