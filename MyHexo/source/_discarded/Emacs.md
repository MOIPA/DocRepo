title: Emacs
author: tr
tags:
  - Emacs
categories:
  - Emacs
date: 2020-08-19 10:35:00
---
### emacs for windows配置


下载的emacs版本在24以上，内置了插件管理软件

<!--more-->

#### 基础配置

1. 官网下载emacs

2. 由于在windows下使用，需要配置home目录，启动emacs后可以打开手册看到windows推荐配置

	第一种：windows+R 输入regedit打开注册表 建立如下目录`\HKEY_CURRENT_USER\Software\GNU`，新建字符串`HOME`,值为自己的目录
    
   第二种：直接添加`HOME`环境变量，不推荐，用这个方法会导致很多其他依赖home的软件配置地址失效，如git，本来地址在c:/users/.ssd下，修改后到了emacs的home目录
   
3. 基础学习：ctrl+h t 打开menu手册，简单学习emacs操作方式先

4. M+x : alt + x   

5. C+x: ctrl + x

6. RET: enter

#### 配置国内源

1. 默认的网速比较慢，直接打开home目录下的.emacs文件，在文件头添加如下内容

  ```
  ; set china source
  (setq package-archives '(("tgnu"   . "http://mirrors.tuna.tsinghua.edu.cn/elpa/gnu/")
                           ("melpa" . "http://mirrors.tuna.tsinghua.edu.cn/elpa/melpa/")))
  (package-initialize)

  ```
  
  `package-initialize` 只需添加一次
  
#### 配置插件

所有插件推荐去github或者官网查readme文档

##### evil

这个插件为`vim`快捷键插件，和emacs冲突的键位不多，不用太多配置即可直接上手

1. `M+x package-refresh-contents` 刷新来自melpa的内容，evil默认不在elpa内

2. `M+x package-install <RET> evil <RET>` 直接安装，或者 `M+x list-packages` => `C+s evil`寻找，找到后键入 `I` 标识即将安装 键入`x` 执行安装

3. 安装后编辑`.emacs`,输入如下内容

  ```
  (require 'evil)
  (evil-mode 1)
  ```
4. 重启即可生效，简要说明下，下方状态栏内`<N>`标识vim模式，`ctrl+z`即可切换到emacs模式 此时标识为`<E>` ,其他标识如`<V>`是vim的其他模式，视图模式啊啥的。evil可以添加其他扩展支持，具体wiki


#### org

必装的东西，我用emacs就是为了这个和eww

1. `M+x package-install <RET> org <RET>` 直接安装，或者 `M+x list-packages` => `C+s org`寻找，找到后键入 `I` 标识即将安装 键入`x` 执行安装

2. 无需配置，直接使用，新建文本:`C+x f`在文本内:`M+x org-mode`进入org-mode
	
    `ctrl+shift+<RET>` 快速创建一个todo
    
    `ctrl+c ctrl+t`快速切换todo状态
    
    `C-c C-s` 选择时间
    
3. 其余看官方文档


#### smex

快速命令插件，命令代替各种奇葩快捷键

1. `M+x package-install <RET> smex <RET>` 直接安装，或者 `M+x list-packages` => `C+s smex`寻找，找到后键入 `I` 标识即将安装 键入`x` 执行安装

2. smex可以改变`M+x`快捷键，需要添加配置

  ```
  (require 'smex)   ; Not needed if you use package.el
  (smex-initialize) ; Can be omitted. This might cause a (minimal) delay
                    ; when Smex is auto-initialized on its first run.
  ; bind keys
  (global-set-key (kbd "M-x") 'smex)
  (global-set-key (kbd "M-X") 'smex-major-mode-commands)
  ;; This is your old M-x.
  (global-set-key (kbd "C-c C-c M-x") 'execute-extended-command)

  ```
3. 使用时`M+x`呼出的是smex，此模式下输入命令后会在ido交互内出现`{...|..}`命令提示，默认是7个推荐，按照使用频率排序，在输入提示模式下`C+s`/`C+r`切换命令，`SPACE`展示所有相关命令并切换

	如果想使用原来的`M+x` 键入`C-c C-c M-x`即可
   
   
#### helm

只用helm-buffers-list代替`C-x b`，其他和smex有点覆盖，smex够用了暂时不折腾


