title: Intellij idea实用配置和快捷键
author: tr
tags:
  - IDE
categories:
  - IDE
date: 2020-05-12 09:39:00
---
### 配置

#### 配置文件

`IDE Features Trainer` ：官方推出的学习插件


配置文件地址，在idea目录下

+ 32位系统对应：idea.exe.vmoptions
+ 64位系统对应：idea64.exe.vmoptions

<!--more-->

#### JVM配置
推荐使用 IDEA 自带菜单中的 Help -> Edit Custom VM Options 进行JVM个性化配置，直接修改会在下次更新丢失

![upload successful](/images/pasted-24.png)



```
常用参数推荐

+ -Xms128m，16 G 内存的机器可尝试设置为 -Xms512m
+ -Xmx750m，16 G 内存的机器可尝试设置为 -Xmx1500m
+ -XX:MaxPermSize=350m，16G 内存的机器可尝试设置为 -XX:MaxPermSize=500m（P.S：2017 后的版本该参数被剔除）
+ -XX:ReservedCodeCacheSize=225m，16G 内存的机器可尝试设置为 -XX:ReservedCodeCacheSize=500m
```

#### IDEA配置

`Help -> Edit Custom Properties`

```
idea.config.path=${user.home}/.IntelliJIdea/config，该属性主要用于指向 IntelliJ IDEA 的个性化配置目录，默认是被注释，打开注释之后才算启用该属性，这里需要特别注意的是斜杠方向，这里用的是正斜杠。
idea.system.path=${user.home}/.IntelliJIdea/system，该属性主要用于指向 IntelliJ IDEA 的系统文件目录，默认是被注释，打开注释之后才算启用该属性，这里需要特别注意的是斜杠方向，这里用的是正斜杠。如果你的项目很多，则该目录会很大，如果你的 C 盘空间不够的时候，还是建议把该目录转移到其他盘符下。
idea.max.intellisense.filesize=2500，该属性主要用于提高在编辑大文件时候的代码帮助。IntelliJ IDEA 在编辑大文件的时候还是很容易卡顿的。
idea.cycle.buffer.size=1024，该属性主要用于控制控制台输出缓存。有遇到一些项目开启很多输出，控制台很快就被刷满了没办法再自动输出后面内容，这种项目建议增大该值或是直接禁用掉，禁用语句 idea.cycle.buffer.size=disabled。

```

### 快捷键

#### 常用必会快捷键
##### ctrl
```
Ctrl + W	递进式选择代码块。可选中光标所在的单词或段落，连续按会在原有选中的基础上再扩展选中范围 
Ctrl + E	显示最近打开的文件记录列表 
Ctrl + N	根据输入的 类名 查找类文件 
Ctrl + G	在当前文件跳转到指定行处
Ctrl + J	插入自定义动态代码模板 
Ctrl + P	方法参数提示显示 
Ctrl + U	前往当前光标所在的方法的父类的方法 / 接口定义 
Ctrl + B	进入光标所在的方法/变量的接口或是定义处，等效于 Ctrl + 左键单击 
Ctrl + K	版本控制提交项目，需要此项目有加入到版本控制才可用
Ctrl + T	版本控制更新项目，需要此项目有加入到版本控制才可用
Ctrl + F1	在光标所在的错误代码处显示错误信息 
Ctrl + F3	调转到所选中的词的下一个引用位置 
Ctrl + Tab	编辑窗口切换，如果在切换的过程又加按上delete，则是关闭对应选中的窗口,加shift是逆着选
```

##### alt

```
Alt + `  git操作必备
Alt + F1	弹出所有工具栏，可以快速选择db等栏目  超好用！
Alt + 1,2,3...9	显示对应数值的选项卡

```

##### 组合

```
Ctrl + Alt + L	格式化代码，可以对当前文件和整个包目录使用 
Ctrl + Alt + O	优化导入的类，可以对当前文件和整个包目录使用 
Ctrl + Alt + T	对选中的代码弹出环绕选项弹出层，可以很方便快捷的添加trycatch
Ctrl + Alt + 左方向键	退回到上一个操作的地方 
Ctrl + Alt + 右方向键	前进到上一个操作的地方 
Ctrl + Alt + Enter 光标所在行上空出一行
Shift + Enter 光标下空出一行
Ctrl + Alt + S 打开系统设置

Ctrl + Shift + N	通过文件名定位 / 打开文件 / 目录，打开目录需要在输入的内容后面多加一个正斜杠 
Ctrl + Shift + U	对选中的代码进行大 / 小写轮流转换 
Ctrl + Shift + T	对当前类生成单元测试类，如果已经存在的单元测试类则可以进行选择 （必备）
Ctrl + Shift + C	复制当前文件磁盘路径到剪贴板 
Ctrl + Shift + B	跳转到类型声明处
Ctrl + Shift + / 代码块注释
Ctrl + Shift + F7	高亮显示所有该选中文本，按Esc高亮消失 
Ctrl + Shift + F12	编辑器最大化
Ctrl + Shift + Enter	自动结束代码，行末自动添加分号
Ctrl + Shift + Backspace	退回到上次修改的地方 
Ctrl + Shift + 1,2,3...9	快速添加指定数值的书签
```

#### 第三方VIM快捷键

 常用的jkhl移动和p，y赋值粘贴即可，其他快捷键和idea冲突，建议在右下角设置全部还原为idea
 
![upload successful](/images/pasted-25.png)

![upload successful](/images/pasted-26.png)


#### debug快捷键


```
F7	F7	进入下一步，如果当前行断点是一个方法，则进入当前方法体内，如果该方法体还有方法，则不会进入该内嵌的方法中 
F8	F8	进入下一步，如果当前行断点是一个方法，则不进入当前方法体内 
F9	Command + Option + R	恢复程序运行，但是如果该断点下面代码还有断点则停在下一个断点上 
Alt + F8	Option + F8	选中对象，弹出可输入计算表达式调试框，查看该输入内容的调试结果 
Shift + F7 自动步入  特别好用
```
