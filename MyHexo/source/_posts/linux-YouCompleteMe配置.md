title: linux-YouCompleteMe配置
author: tr
tags:
  - Linux
categories: []
date: 2019-06-25 17:54:00
---
### 折腾笔记

这是一款linux在vim下的插件，自动补全代码

#### 配置vim的插件管理器vundle

执行一下命令导入vundle包

	git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/Vundle.vim

配置~/.vimrc （最好别改/etc/vimrc) 配置后重新打开，以下配置里写了两个插件：auto-format和nerdTree,这两个差价可以稍后写

```vimrc

" vundle
set nocompatible              " required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'gmarik/Vundle.vim'

" Add all your plugins here (note older versions of Vundle used Bundle instead of Plugin)
Plugin 'Chiel92/vim-autoformat'
nnoremap <F6> :Autoformat<CR>
let g:autoformat_autoindent = 0 
let g:autoformat_retab = 0 
let g:autoformat_remove_trailing_spaces = 0 
                           
Plugin 'https://github.com/scrooloose/nerdtree'
nnoremap <F3> :NERDTreeToggle<CR>

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required

```

配置插件后重新打开vim，输入:PluginInstall开始安装那两个插件


#### 配置安装YCM

1. Ensure that your version of Vim is at least 7.4.1578 and that it has support for Python 2 or Python 3 scripting.确保安装的vim版本在7.4以上（vim --version)确保vim支持python2/3

2. 下载YCM本体，有两种安装方式：
	
    a. 使用vundle安装（推荐）：在vimrc插件配置里写入： Plugin 'Valloric/YouCompleteMe'，执行：PluginInstall。
    
    b. 或者在~/.vim/bundle/下使用
    
    git clone https://github.com/ycm-core/YouCompleteMe.git,
    
    在下载后需要执行git submodule update --init --recursive 

3. 安装YCM：（下载C语言家族支持的情况下）
	
   1. 载好cmake和python head文件类似：python-dev,python3-dev,但是archlinux好像也么得
    
    然后为了支持c语言，我们需要编译ycm_core，首先执行以下命令

          cd ~
          mkdir ycm_build
          cd ycm_build

 2. 然后下载最新的libclang或者clangd（编译的支持文件），由于archlinux下好像么得libclang，我从llvm.org [llvm.org](http://http://releases.llvm.org/download.html).下载了clangd的编译版本，[archlinux编译过版本](http://releases.llvm.org/8.0.0/clang+llvm-8.0.0-aarch64-linux-gnu.tar.xz)    
下载后将文件放入~/ycm_temp/llvm_root_dir（不存在请创建）

 3. 执行编译配置：（系统是linux，Generator选UnixMakefiles，需要cmake和make提前装好）

          cmake -G "Unix Makefiles" -DPATH_TO_LLVM_ROOT=~/ycm_temp/llvm_root_dir . ~/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp
          
     编译结束，当前目录生成配置文件

 4. 执行编译本体
 
			cmake --build . --target ycm_core --config Release

    官方文档中说--config Release是为了windows特有的，可以不加
    
    注意：在cmake编译好配置文件后不能更改当前文件所在的目录，否则编译本体会找不到目录
   
  5. 