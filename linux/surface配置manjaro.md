## Config Surface

### check the official site , download the ImageWriter in windows

if choose other writer , use DD mode

### important : close the SecureBootMode On Surface!!!

push the voice-plus button and powerOn to get into UEFI

----

### plug in the USB

 1.	#### set the install option

> 1. tz(timeZone) = Asia/Shanghai
>
> 2. lang = zh_CN 
> 3. driver = free/nonfree (专属或者开源) 
> 4. **boot** 

2.	####  install with windows

----------------------

### configuration after installing

1. #### configure source

   1.  sudo pacman-mirrors -i -c China -m rank   : choose the top one
   2. sudo pacman-Syy    :refresh
   3. **add archlinux chinese repository**  
   	vim /etc/pacman.conf ---> then add :
	[archlinuxcn]
	Server = https://mirrors.shu.edu.cn/archlinuxcn/$arch

2. #### import GPG KEY 

   1. sudo pacman -Sy archlinuxcn-keyring

3. #### install yaourt and pacaur and vim

4. #### update system
   1. sudo pacman -Syu

### Some softwares

1. #### 搜狗拼音

   1. sudo pacman -S fcitx-sogoupinyin
   2. sudo pacman -S fcitx-im
   3. sudo pacman -S fcitx-configtool
   4. vim ~/.xprofile : add:
   	export GTK_IM_MODULE=fcitx
   	export QT_IM_MODULE=fcitx
   	export XMODIFIERS="@im=fcitx"


2. #### JDK

   1. sudo pacman -R jdk8-openjdk
   2. sudo pacman -R jre8-openjdk
   3. sudo pacman -R jre8-openjdk-headless
   4. yaourt jdk

3. #### git
   1. sudo pacman -S git

