title: ArchLinux 安装 + 大部分的软件配置
author: tr
tags:
  - Linux
categories:
  - Linux
date: 2022-03-28 09:18:00
---
# ArchLinux安装配置

## 安装系统

### 安装准备

我的是双系统的配置，在安装之前划分50GB的磁盘空间

> 1. 去官网下载镜像，制作usb启动盘
> 2. 开机进入usb的livecd内
<!--more-->

### 联网设置

用的笔记本 所以使用wlan联网，arch的livecd环境里提供了iwctl命令
```shell
iwctl #进入
device list #查看所有网卡设备 这里我只有wlan0设备
station wlan0 get-networks #获取wifi链接
station wlan0 connect wifi名 # 会提示输入密码，输入即可
exit #退出
ping www.baidu.com
```


### 划分分区

> 划分扇区，按照arch的规定，至少要一个swap分区（否则无法休眠）和一个主分区以及boot分区（boot分区内存放了esp信息）

```shell
lsblk #查看分区信息
fdisk -l # 查看分区信息
fdisk -l /dev/nvme0n1 # 查看使用的这块硬盘的分区信息

# 如果在windows中压缩出了50GB，且建立了扇区的话这里要删除这个扇区
fdisk /dev/nvme0n1  # 进入管理扇区模式

p #打印所有分区信息
d #删除分区（我这里删除8号分区50GB的那个，之前在windows下建立了分区）
n #新建swap分区 （提示输入分区头地址信息，回车即可，提示分配大小，输入：+2GB，回车即可）
n #新建boot分区 （提示输入分区头地址信息，回车即可，提示分配大小，输入：+512MB，回车即可）
n #新建主分区 全部回车即可
t #改变两个分区类型（提示数字，选择刚刚建立的两个分区，修改EFI类型和swap类型）
w #写入分区信息 
```
> 新的分区会有新的名字，fdisk -l /dev/nvme0n1 会看到新的三个分区信息 可能是这样的命名：nvme0n1p6（512MB的efi分区） , nvme0n1p7（2GB的swap分区） , nvme0n1p8（47GB的主分区）

> 格式化分区：
> + mkfs.ext4 /dev/nvme0n1p8
> + mkswap /dev/nvme0n1p7
> + mkfs.fat -F 32 /dev/nvme0n1p6

> 启动交换区：swapon /dev/nvme0n1p7

> 挂载分区：
> + mount /dev/nvme0n1p8 /mnt # 这里挂载到livecd的mnt目录下，因为安装的资料会拷贝进去
> + mount /dev/nvme0n1p6 /mnt/boot #若无boot目录 mkdir /mnt/boot即可

### 执行安装

#### 1. 安装

pacstrap /mnt base linux linux-firmware  # 开始正式拷贝内核信息和启动条码到主分区和esp分区

#### 2. 自动挂载

安装完成后执行：`genfstab -U /mnt >> /mnt/etc/fstab # 生成自动挂载信息`，有了这个文件，每次系统启动就会读取并自动挂载

#### 3. 切换到安装好的系统：arch-chroot /mnt

#### 4. 配置时区

`ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`

`hwclock --systohc`

#### 5. 配置系统语言

编辑 `/etc/locale.gen` 取消这个注释 `en_US.UTF-8 UTF-8`

最后生成语言信息 执行： `locale-gen` 

然后编辑`/etc/locale.conf`文件 写入：`LANG=en_US.UTF-8`

#### 6. 写入hostname信息

`vim /etc/hostname`内容可随意

#### 7. 生成Initramfs（如果使用了refind需要生成）

`mkinitcpio -P`

```
#initramfs 原理：
Linux系统启动时使用initramfs (initram file system), initramfs可以在启动早期提供一个用户态环境，借助它可以完成一些内核在启动阶段不易完成的工作。当然initramfs是可选的，Linux中的内核编译选项默认开启initrd。在下面的示例情况中你可能要考虑用initramfs。

加载模块，比如第三方driver
定制化启动过程 (比如打印welcome message等)
制作一个非常小的rescue shell
任何kernel不能做的，但在用户态可以做的 (比如执行某些命令)
一个initramfs至少要包含一个文件，文件名为/init。内核将这个文件执行起来的进程作为main init进程(pid 1)。当内核挂载initramfs后，文件系统的根分区还没有被mount, 这意味着你不能访问文件系统中的任何文件。如果你需要一个shell，必须把shell打包到initramfs中，如果你需要一个简单的工具，比如ls, 你也必须把它和它依赖的库或者模块打包到initramfs中。总之，initramfas是一个完全独立运行的体系。
```

#### 8. 修改密码 添加用户

`passwd` # 修改root密码

`useradd -m -G wheel tr` # 创建wheel用户组下的用户tr

`passwd tr`#修改密码

修改文件：`/etc/sudoers`，取消注释：`%wheel ALl=(ALL:ALL) ALL`，这样wheel组下的除root用户的其他用户才能执行任何命令

```
-m 创建主目录
-G wheel 指定用户组为wheel（管理员组，如果不是这个组的用户，无法通过su命令成为管理员）
```

#### 9.1 配置网络源

镜像配置：`vim /etc/pacman.d/mirrorlist` 

添加内容`Server = https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
`

cn源配置：`vim /etc/pacman.conf `

添加内容：`[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
`

导入key：`pacman -S archlinuxcn-keyring`

更新仓库：`pacman -Syy`


#### 9.2 安装基本的工具包

安装一些常用的网络管理工具和编辑工具等，这里面还安装了grub + os-prober，如果不用grub启动可以换成其他boot loader 如rEFind

`pacman -S grub efibootmgr networkmanager network-manager-applet dialog wireless_tools wpa_supplicant os-prober mtools dosfstools ntfs-3g base-devel linux-headers reflector git sudo wget vim`

#### 10.安装配置 boot loader

如果你是intel的cpu，需要安装intel的微码文件：`pacman -S intel-ucode`
如果是amd `pacman -S amd-ucode`

推荐使用rEFind，美观多了，使用它之前确保生成了Initramfs,也就是第七步内容

执行`pacman -S refind` , `refind-install`

执行完后可以查看boot目录是否写入了配置

#### 11. rEFind配置

refind所有配置都在esp分区也就是boot目录：`ls /boot/`

可以看到refind_linux.conf配置文件，里面写了不同情况下的启动，里面的id是分区id，分区id有很多种，uuid，partuuid，id等，通过命令看到
```shell
ls -l /dev/disk/by-
by-id/        by-label/     by-partlabel/ by-partuuid/  by-path/      by-uuid/
```

rEFind真正的启动配置：`/boot/EFI/refind/refind.conf`，这个文件内描述了启动条目包括启动的图标，loader等

#### 12. 识别windows问题

不管是用grub还是rEFind，都不需要我们手动执行操作，系统应该可以自动识别，因为使用了UEFI的启动方式，当机器开机时，不再读取MBR内容，而是读取磁盘内的ESP分区，找到ESP的分区启动条目后显示在启动菜单内。

安装完rEFind或者grub后，会在UEFI的启动项内新增启动条目，开机进入UEFI可以看到（Acer是f2进入bios，f12进入UEFI启动序列）

如果使用rEFind启动但是没有windows的图标（不太可能），可以先进入linux系统，手动挂载windows的ESP分区（很小 只有100
，里面有个Microsoft目录），把目录的Microsoft目录拷贝到`/boot/EFI/`下

#### 13. 完成安装

```shell
exit #退出系统
reboot
```

### 安装后的配置

大体配置按照文档来：[安装后的推荐配置](https://wiki.archlinux.org/title/General_recommendations)

开机启动进入新系统

#### 时区问题

时间快了8个小时，这个是因为双系统下，linux认为电脑时钟就是UTC时间，时区是上海，所以+了8小时。但是windwos下认为电脑时钟（Bios时间）是本地时间。这个问题可以通过把时区设置为+0或者使用网络时间解决。

timedatectl #查看本机时间

#### 配置网络

按照archlinux官方文档来：[网络配置](https://wiki.archlinux.org/title/NetworkManager_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)

`systemctl enable --now NetworkManager` 自动启动网络管理工具

`nmtui` 设置wifi

#### 显卡启动（集显就别操心了）

NVIDIA独显驱动：pacman -S nvidia nvidia-utils

#### 图形界面配置

显示服务器有Xorg 和 Wayland，推荐Xorg：`pacman -S xorg`

显示管理器有gdm(Gnome),sddm(Kde)的，我偏爱Gnome环境 `pacman -S gdm` `systemctl enable gdm`

桌面环境也是Gnome：`pacman -S gnome`

```
xorg
概述
xorg是x11的一个实现，而x window system是一个C/S结构的程序，xorg只是提供一个X server，负责底层的操作。当你运行一个程序的时候，这个程序会链接到X server上，由X server接收键盘鼠标输入和负责屏幕输出窗口的移动、窗口标题的样式等。

X window 是由X server 和 X client组成，X server 和 X client之间的通信是通过 X 协议。

x server
仅仅负责鼠标、键盘、显卡、显示器这些输入输出部件。由于硬件厂商很多，所以x server不能自动识别出所有需要的参数，如果识别不出来，那么就需要编辑一下/etc/X11/xorg.conf文件进行配置。

x client
负责处理程序的运行。比如单击一下gvim图标，x server会告诉x client用户刚才移动鼠标到什么位置并做了什么操作，x client收到后会识别操作并作出相应的反馈，打开gvim程序，然后x client让x server在显示器上显示一个gvim的画面。

xorg与桌面环境的关系
先介绍几个概念：窗口管理器、显示管理器和文件管理器

窗口管理器则是为了实现一个屏幕上显示多个X程序,实现调整程序大小,标题栏,最大化,最小化,关闭按钮,虚拟桌面这些功能。如果没有窗口管理器，那么一次只能运行一个GUI程序,而且分辨率锁死,显然很不符合使用习惯。窗口管理器往往集成在常见的桌面环境中，比如Xfce使用的窗口管理器为Xfwm,此外还有Gnome的mutter,KDE的Kwin等。

显示管理器（display manager）,用于开机后显示登陆界面,并启动窗口管理器等X组件.没有显示管理器,Linux开机会显示命令行登陆界面,需要使用命令行登陆后手动启动Xserver和窗口管理器才能显示GUI,显示管理器自动的完成这些工作.常见的有GDM、LightDM、DDM。
```

#### yaourt配置

需要添加国内源才能用`pacman -S yaourt`

#### 中文字体配置

字体可以随便下载，但是下载的字体如果不选择默认字体（不编辑~/.config/fontconfig/fonts.conf）的话，linux渲染对的字体可能来自不同字体集合，导致有的字体粗，有的字体细。可以通过下载fontweak这个gui文件可视化的编辑，更加方便`yaourt fontweak`

全部字体可以在wiki内看到：[字体](https://wiki.archlinux.org/title/Fonts_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E4%B8%AD%E6%96%87%E5%AD%97)

这里推荐win10的字体感觉还蛮好看的

`yaourt -S ttf-ms-win10-zh_cn`

#### 手动安装中文字体（可选）

如果字体不好安装，仓库中下载失败，可以手动安装win10字体（从win10系统中拷贝过来，可以使用
盘）

```shell
➜  ~ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
nvme0n1     259:0    0 476.9G  0 disk 
├─nvme0n1p1 259:1    0   100M  0 part 
├─nvme0n1p2 259:2    0    16M  0 part 
├─nvme0n1p3 259:3    0 237.4G  0 part 
├─nvme0n1p4 259:4    0     1G  0 part 
├─nvme0n1p5 259:5    0 188.2G  0 part 
├─nvme0n1p6 259:6    0     2G  0 part [SWAP]
├─nvme0n1p7 259:7    0   512M  0 part /boot
└─nvme0n1p8 259:8    0  47.7G  0 part /
➜  ~ mount /dev/nvme0n1p3 ~/ms
mount: /home/tr/ms: must be superuser to use mount.
➜  ~ sudo mount /dev/nvme0n1p3 ~/ms

[root@tr tr]# mkdir /usr/share/fonts/windows
[root@tr tr]# cp ms/Windows/Fonts/* /usr/share/fonts/windows/
[root@tr tr]# chmod -R 777 /usr/share/fonts/windows
[root@tr tr]# fc-cache -vf

```

#### 文泉驿中文（推荐）

```shell
➜  ~ sudo pacman -S wqy-microhei wqy-bitmapfont wqy-zenhei wqy-microhei-lite adobe-source-han-sans-cn-fonts adobe-source-han-serif-cn-fonts
➜  ~ sudo pacman -S ttf-dejavu noto-fonts noto-fonts-extra noto-fonts-emoji noto-fonts-cjk
➜  ~ vim .config/fontconfig/fonts.conf 

```
fonts.conf 内容如下
```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<fontconfig>
  <dir>~/.fonts</dir>
  <match>
    <test name="family">
      <string>sans-serif</string>
    </test>
    <edit binding="strong" mode="prepend" name="family">
      <string>WenQuanYi Micro Hei</string>
      <string>WenQuanYi Zen Hei Sharp</string>
      <string>WenQuanYi Zen Hei</string>
    </edit>
  </match>
  <match>
    <test name="family">
      <string>serif</string>
    </test>
    <edit binding="strong" mode="prepend" name="family">
      <string>WenQuanYi Micro Hei Light</string>
    </edit>
  </match>
  <match>
    <test name="family">
      <string>monospace</string>
    </test>
    <edit binding="strong" mode="prepend" name="family">
      <string>WenQuanYi Micro Hei Mono</string>
      <string>WenQuanYi Zen Hei Mono</string>
    </edit>
  </match>
  <match target="font">
    <edit mode="assign" name="antialias">
      <bool>true</bool>
    </edit>
  </match>
  <match target="font">
    <edit mode="assign" name="hinting">
      <bool>true</bool>
    </edit>
  </match>
  <match target="font">
    <edit mode="assign" name="hintstyle">
      <const>hintfull</const>
    </edit>
  </match>
  <match target="font">
    <edit mode="assign" name="rgba">
      <const>none</const>
    </edit>
  </match>
  <match target="font">
    <edit mode="assign" name="lcdfilter">
      <const>lcddefault</const>
    </edit>
  </match>
  <match target="font">
    <edit mode="assign" name="embeddedbitmap">
      <bool>false</bool>
    </edit>
  </match>
</fontconfig>

```

## 常用软件

### 搜狗拼音

安装拼音管理工具：`pacman -S fcitx-im fcitx-configtool`

修改配置文件：`vim /etc/profile（全局配置）`内容如下：
```shell
# fcitx
export XIM=fcitx
export XIM_PROGRAM=fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
```
安装搜狗拼音：`yaourt fcitx-sogoupinyin`，注销后登录，可以看到fcitx的图形化管理工具，在input method configuration内选择`+`添加搜狗拼音即可,注销后登录即可

### 安装chrome

yaourt -S google-chrome 

需要用非root用户打开

### 安装oh my zsh

[oh my zsh](https://github.com/ohmyzsh/ohmyzsh)

安装zsh : `pacman -S zsh`

安装oh my zsh并安装：`sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"` 

配置文件在：~/.zshrc，可以在里面编辑主题和插件

#### 配置箭头主题的zsh 和 vim

##### zsh
powerline字体和插件配置都可以从wiki找到：[powerline](https://wiki.archlinux.org/title/Powerline)

安装power line字体
```shell
# clone
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts
./install.sh
# clean-up a bit
cd ..
rm -rf fonts

#配置power line字体到oh my zsh 界面：vim ~/.zshrc追加：

powerline-daemon -q
. /usr/share/powerline/bindings/zsh/powerline.zsh

# source ~/.zshrc生效

```

修改配置文件找到ZSH_THEME,修改里面值为agnoster，`source ~/.zshrc`生效

##### vim

安装power line status 状态栏 用于vim等

```shell
 pacman -S powerline-vim  #安装vim插件
 
 # 安装vim插件管理器
 sh -c 'curl -fLo "${XDG_DATA_HOME:-$HOME/.local/share}"/nvim/site/autoload/plug.vim --create-dirs \\n       https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim'

```
 > 编辑~/.vimrc 写入以下内容
 
 ```
" Plugins will be downloaded under the specified directory.
call plug#begin(has('nvim') ? stdpath('data') . '/plugged' : '~/.vim/plugged')

" Declare the list of plugins.
Plug 'tpope/vim-sensible'
Plug 'junegunn/seoul256.vim'
set rtp+=/usr/share/powerline/bindings/vim

" List ends here. Plugins become visible to Vim after this call.
call plug#end()
```
> vim 打开vim 执行 ：PlugInstall

#### 配置zsh的插件zsh-autosuggestions

```shell
# Clone this repository into $ZSH_CUSTOM/plugins (by default ~/.oh-my-zsh/custom/plugins)

git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
# Add the plugin to the list of plugins for Oh My Zsh to load (inside ~/.zshrc):

plugins=( 
    # other plugins...
    zsh-autosuggestions
)


```

source ~/.zshrc 生效配置

默认补全使用右箭头，自动补全可以使用逗号，编辑 ~/.zshrc 写入如下配置即可
```
bindkey ',' autosuggest-accept
```

#### 配置zsh插件 z 

类似 autojump
```
plugins=( 
    # other plugins...
    z
)
```

### 安装配置gnome桌面插件

```shell
$ git clone https://aur.archlinux.org/chrome-gnome-shell.git
$ cd chrome-gnome-shell
$ makepkg -si
```

安装好后可以在软件列表里看到 extensions这个GUI软件

然后就可以登录这个网站下载各种好玩的插件了：[gnome插件](https://extensions.gnome.org/)

比如截图插件，可以修改QQ的快捷键


![upload successful](/images/pasted-139.png)

![upload successful](/images/pasted-140.png)

#### 强烈推荐的插件 

ddterm ：最好用的drop down terminal

### 安装aria2下载器

安装下载器：`pacman -S aria2`

安装pip工具，因为系统自带了python3，而aira的gui工具需要pip所以下载`yaourt -S python-pip`

安装前端工具：`sudo pacman -S persepolis`

开始菜单里就有了，点开即用

### 安装clash

直接下载下来解压缩运行就行了[clash gui](https://github.com/Fndroid/clash_for_windows_pkg)

执行 `/opt/clash/cfw`

系统配置下手动开启vpn的配置：

![upload successful](/images/pasted-141.png)

如果觉得gui碍事，可以使用命令行的 参考这篇博客 [clash 配置](http://www.manongjc.com/detail/27-guujywyvgwvhqly.html)

### 安装钉钉

yaourt dingtalk

### 安装vpn

openvpn,用来链接公司的局域网开发：`pacman -S openvpn`，这是个命令行，界面下面两个

安装l2tp隧道：
```
sudo pacman -S networkmanager-l2tp strongswan  
systemctl start strongswan.service 
```

#### eovpn界面
安装eovpn:对应的GUI界面，之前需要安装Flatpak，这是一个包管理器，最大的好处是依赖可以隔离，下载的软件也需要这个

1. `sudo pacman -S flatpak` 安装 flatpak
2. `yaourt -S eovpn`
3. [去github上下载对应的flatpak包](https://github.com/jkotra/eOVPN)点击flatpak图标就下载了
4. flatpak install flathub com.github.jkotra.eovpn # 安装
5. flatpak run com.github.jkotra.eovpn # 执行

#### networkmanager-openvpn界面

```shell
sudo pacman -S networkmanager-openvpn    
```
在vpn上选择+号，即可配置l2tp隧道了

![upload successful](/images/pasted-143.png)

![upload successful](/images/pasted-147.png)
 
![upload successful](/images/pasted-146.png)

![upload successful](/images/pasted-148.png)

链接成功，可以干活了

### redis desktop manager

[官网](https://snapcraft.io/redis-desktop-manager)

1. 下载snap ： `pacman -S snapd`
2. 启动： `systemctl start snapd.socket`
3. 下载：`sudo snap install redis-desktop-manager`
4. 链接snap仓库：`sudo ln -s /var/lib/snapd/snap /snap  `
5. 启动：`snap/bin/redis-desktop-manager.resp  `

### docker

pacman -S docker

> 如果是非root用户不可以执行docker命令，两种方式解决

1. 将用户添加且暂时切换到docker组，`user mod -aG docker 用户名`，`newgrp docker`

2. 修改docker可执行权限，`sudo chmod 666 /var/run/docker.sock
`

### 电源管理工具（省电）

[Laptop Mode Tools ](https://wiki.archlinux.org/title/Laptop_Mode_Tools_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E5%AE%89%E8%A3%85)

### 蓝牙管理器

[bluetooth](https://wiki.archlinux.org/title/Bluetooth)



### maven

1. 下载maven压缩包，解压到目录 `export PATH=/opt/maven3.8/bin:$PATH `
2. `vim /opt/maven3.8/conf/settings.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

<!--
 | This is the configuration file for Maven. It can be specified at two levels:
 |
 |  1. User Level. This settings.xml file provides configuration for a single user,
 |                 and is normally provided in ${user.home}/.m2/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -s /path/to/user/settings.xml
 |
 |  2. Global Level. This settings.xml file provides configuration for all Maven
 |                 users on a machine (assuming they're all using the same Maven
 |                 installation). It's normally provided in
 |                 ${maven.conf}/settings.xml.
 |
 |                 NOTE: This location can be overridden with the CLI option:
 |
 |                 -gs /path/to/global/settings.xml
 |
 | The sections in this sample file are intended to give you a running start at
 | getting the most out of your Maven installation. Where appropriate, the default
 | values (values used when the setting is not specified) are provided.
 |
 |-->
<settings xmlns="http://maven.apache.org/SETTINGS/1.2.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.2.0 https://maven.apache.org/xsd/settings-1.2.0.xsd">
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
  <localRepository>/home/mavenRepo</localRepository>

  <!-- interactiveMode
   | This will determine whether maven prompts you when it needs input. If set to false,
   | maven will use a sensible default value, perhaps based on some other setting, for
   | the parameter in question.
   |
   | Default: true
  <interactiveMode>true</interactiveMode>
  -->

  <!-- offline
   | Determines whether maven should attempt to connect to the network when executing a build.
   | This will have an effect on artifact downloads, artifact deployment, and others.
   |
   | Default: false
  <offline>false</offline>
  -->

  <!-- pluginGroups
   | This is a list of additional group identifiers that will be searched when resolving plugins by their prefix, i.e.
   | when invoking a command line like "mvn prefix:goal". Maven will automatically add the group identifiers
   | "org.apache.maven.plugins" and "org.codehaus.mojo" if these are not already contained in the list.
   |-->
  <pluginGroups>
    <!-- pluginGroup
     | Specifies a further group identifier to use for plugin lookup.
    <pluginGroup>com.your.plugins</pluginGroup>
    -->
  </pluginGroups>

  <!-- proxies
   | This is a list of proxies which can be used on this machine to connect to the network.
   | Unless otherwise specified (by system property or command-line switch), the first proxy
   | specification in this list marked as active will be used.
   |-->
  <proxies>
    <!-- proxy
     | Specification for one proxy, to be used in connecting to the network.
     |
    <proxy>
      <id>optional</id>
      <active>true</active>
      <protocol>http</protocol>
      <username>proxyuser</username>
      <password>proxypass</password>
      <host>proxy.host.net</host>
      <port>80</port>
      <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
    </proxy>
    -->
  </proxies>

  <!-- servers
   | This is a list of authentication profiles, keyed by the server-id used within the system.
   | Authentication profiles can be used whenever maven must make a connection to a remote server.
   |-->
  <servers>
    <!-- server
     | Specifies the authentication information to use when connecting to a particular server, identified by
     | a unique name within the system (referred to by the 'id' attribute below).
     |
     | NOTE: You should either specify username/password OR privateKey/passphrase, since these pairings are
     |       used together.
     |
    <server>
      <id>deploymentRepo</id>
      <username>repouser</username>
      <password>repopwd</password>
    </server>
    -->

    <!-- Another sample, using keys to authenticate.
    <server>
      <id>siteServer</id>
      <privateKey>/path/to/private/key</privateKey>
      <passphrase>optional; leave empty if not used.</passphrase>
    </server>
    -->
  </servers>

  <!-- mirrors
   | This is a list of mirrors to be used in downloading artifacts from remote repositories.
   |
   | It works like this: a POM may declare a repository to use in resolving certain artifacts.
   | However, this repository may have problems with heavy traffic at times, so people have mirrored
   | it to several places.
   |
   | That repository definition will have a unique id, so we can create a mirror reference for that
   | repository, to be used as an alternate download site. The mirror site will be the preferred
   | server for that repository.
   |-->
  <mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
     -->
    <mirror>
           <id>alimaven</id>
           <name>aliyun maven</name>
           <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
           <mirrorOf>central</mirrorOf>
     </mirror>

    <mirror>
      <id>maven-default-http-blocker</id>
      <mirrorOf>external:http:*</mirrorOf>
      <name>Pseudo repository to mirror external repositories initially using HTTP.</name>
      <url>http://0.0.0.0/</url>
      <blocked>true</blocked>
    </mirror>
  </mirrors>

  <!-- profiles
   | This is a list of profiles which can be activated in a variety of ways, and which can modify
   | the build process. Profiles provided in the settings.xml are intended to provide local machine-
   | specific paths and repository locations which allow the build to work in the local environment.
   |
   | For example, if you have an integration testing plugin - like cactus - that needs to know where
   | your Tomcat instance is installed, you can provide a variable here such that the variable is
   | dereferenced during the build process to configure the cactus plugin.
   |
   | As noted above, profiles can be activated in a variety of ways. One way - the activeProfiles
   | section of this document (settings.xml) - will be discussed later. Another way essentially
   | relies on the detection of a system property, either matching a particular value for the property,
   | or merely testing its existence. Profiles can also be activated by JDK version prefix, where a
   | value of '1.4' might activate a profile when the build is executed on a JDK version of '1.4.2_07'.
   | Finally, the list of active profiles can be specified directly from the command line.
   |
   | NOTE: For profiles defined in the settings.xml, you are restricted to specifying only artifact
   |       repositories, plugin repositories, and free-form properties to be used as configuration
   |       variables for plugins in the POM.
   |
   |-->
  <profiles>
    <!-- profile
     | Specifies a set of introductions to the build process, to be activated using one or more of the
     | mechanisms described above. For inheritance purposes, and to activate profiles via <activatedProfiles/>
     | or the command line, profiles have to have an ID that is unique.
     |
     | An encouraged best practice for profile identification is to use a consistent naming convention
     | for profiles, such as 'env-dev', 'env-test', 'env-production', 'user-jdcasey', 'user-brett', etc.
     | This will make it more intuitive to understand what the set of introduced profiles is attempting
     | to accomplish, particularly when you only have a list of profile id's for debug.
     |
     | This profile example uses the JDK version to trigger activation, and provides a JDK-specific repo.
    <profile>
      <id>jdk-1.4</id>

      <activation>
        <jdk>1.4</jdk>
      </activation>

      <repositories>
        <repository>
          <id>jdk14</id>
          <name>Repository for JDK 1.4 builds</name>
          <url>http://www.myhost.com/maven/jdk14</url>
          <layout>default</layout>
          <snapshotPolicy>always</snapshotPolicy>
        </repository>
      </repositories>
    </profile>
    -->

    <!--
     | Here is another profile, activated by the system property 'target-env' with a value of 'dev',
     | which provides a specific path to the Tomcat instance. To use this, your plugin configuration
     | might hypothetically look like:
     |
     | ...
     | <plugin>
     |   <groupId>org.myco.myplugins</groupId>
     |   <artifactId>myplugin</artifactId>
     |
     |   <configuration>
     |     <tomcatLocation>${tomcatPath}</tomcatLocation>
     |   </configuration>
     | </plugin>
     | ...
     |
     | NOTE: If you just wanted to inject this configuration whenever someone set 'target-env' to
     |       anything, you could just leave off the <value/> inside the activation-property.
     |
    <profile>
      <id>env-dev</id>

      <activation>
        <property>
          <name>target-env</name>
          <value>dev</value>
        </property>
      </activation>

      <properties>
        <tomcatPath>/path/to/tomcat/instance</tomcatPath>
      </properties>
    </profile>
    -->
  </profiles>

  <!-- activeProfiles
   | List of profiles that are active for all builds.
   |
  <activeProfiles>
    <activeProfile>alwaysActiveProfile</activeProfile>
    <activeProfile>anotherAlwaysActiveProfile</activeProfile>
  </activeProfiles>
  -->
</settings>

```


## 配置软件桌面图标

`ls /usr/share/applications` 这里面存放了所有软件图标，可以依葫芦画瓢，类似其他应用，创建自己的应用，只要路径和图标自己配就行

### 配置clash的图标

之前安装了clash的gui软件，但是没默认创建图标，自己弄一个

软件执行位置在/opt/clash/cfw

1. 首先截个图，clash的图标，放入:`sudo mv /home/tr/Pictures/clash.png /usr/share/icons/gnome/48x48 `

2. 创建桌面图标：` vim /usr/share/applications/clash.desktop`写入以下内容

```shell
[Desktop Entry]
Categories=Network;
Comment=clash
Exec=/opt/clash/cfw
GenericName=clash
Icon=/usr/share/icons/gnome/48x48/clash.png
Keywords=clash;
MimeType=x-scheme-handler/clash;
Name=Clash
Name=Clash
Name[zh_CN]=Clash
Type=Application
X-Deepin-Vendor=user-custom

```

然后程序列表就有了

## 配置虚拟机

### boxes

> gnome桌面自带了boxes软件，这个软件真的很方便，但是简化的过头了，导致用户无法配置网络

> boxes的使用就不说了，点开一目了然，这里推荐如何高级配置boxes内的虚拟机

1. 下载 virt-manager

2. 打开软件点击 `file` `add connection`选择如下，点击链接

![upload successful](/images/pasted-149.png)

![upload successful](/images/pasted-151.png)

3. 点击页面即可

### virtual machine（推荐）

这是专业点的软件，可以配置修改很多东西

```shell
# 安装虚拟机内核
sudo pacman -S virtualbox-host-modules-arch
# 安装虚拟机软件
pacman -S virtualbox
# 安装后不重启需要手动加载内核
sudo modprobe vboxnetadp
sudo modprobe vboxnetflt
sudo modprobe vboxnetpci
sudo modprobe vboxpci
```

> 打开虚拟机使用的时候要注意提供的网络模式
> + host-only 绑定虚拟网卡，需要用户手动创建网卡
> + nat 网络地址转换，可以上网，但是宿主机和虚拟机无法互通（互通需要设置，建议看archlinux文档）
> + bridge 桥接模式，使用一张网卡（可以是虚拟网卡）分配地址，可以和宿主机互通，但是如果网卡不是混合模式是无法上网的

最好的既可以上网又可以和宿主机互通的方式如下设置：

1. 创建host-only的虚拟网卡

![upload successful](/images/pasted-152.png)

2. 关闭虚拟机，将两个网卡（虚拟网卡，和nat模式网卡）配置上即可

![upload successful](/images/pasted-153.png)

![upload successful](/images/pasted-154.png)

![upload successful](/images/pasted-157.png)

![upload successful](/images/pasted-156.png)