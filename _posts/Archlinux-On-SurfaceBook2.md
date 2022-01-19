title: Archlinux On SurfaceBook2
author: tr
tags:
  - Linux
categories:
  - Linux
date: 2020-05-29 17:23:00
---
### Archlinux On SurfaceBook2

win10的内置linux还是效率低，和真正的linux环境还是不一样，但是原来使用子系统就是linux对surfacebook2的适配太差了

但是最近找到了github上的一个surfacebook2的内核包，可以完美适配，因此换了个win/arch双系统，在此记录配置经历。

最终结果是获得win+arch 支持secureboot和适配surface硬件的自定义内核的双系统。

备注：我的方式是先win10后linux，如果反过来请参考

[https://wiki.archlinux.org/index.php/Dual_boot_with_Windows#Important_information](dual boot with windows)

截图：

<!--more-->
![upload successful](/images/pasted-34.png)

#### 烧录u盘启动盘

没必要用烧录软件，下载archlinux最新镜像，我的版本是`archlinux-2020.05.01-x86_64`,解压缩iso镜像到u盘，进入`loader\entries`目录，编辑`archiso-x86_64.conf`，看到类似`archisolabel=ARCH_202005`的信息，将`ARCH_202005`作为自己的设备名。


禁用secureboot，重启按住`音量+`，禁用secureboot

设置u盘优先启动

#### 基本系统安装

参考arch wiki

[https://wiki.archlinux.org/index.php/Installation_guide](Installation guide)

进入u盘的linux安装环境。

##### 分区

1. `fdisk -l` 查看区块状态，fdisk内`w`写入修改，`fdisk -h` 帮助

2. 分区自己的磁盘，我的磁盘是nvme协议的磁盘且预留了50G，`fdisk /dev/nvme0n1`

3. 创建EFI逻辑分区，512MB就够了。

	创建efi目录（efi启动目录，所有boot loader相关） `mkdir -p /boot/efi`
    
   创建efi分区，`n` 创建新分区，回车确定分区起始地址，`+512M` 给出分区大小，这个逻辑分区名是:`nvme0n1p7` 修改分区类型，`t`命令指定类型，输入`1`，确定自己换的就是EFI SYSTEM。
   
   创建swap分区，这个看个人，随意大小，格式化swap分区`mkswap /dev/nvme0n1p8`，挂载交换区`swapon /dev/nvme0n1p8`
   
   创建mnt主分区，要有其他文件夹想自设分区可以随意 `n`->`回车确定开始地址`->`回车确定结束地址`，自动获得了剩余所有空间的逻辑分区`nvme0n1p9`，格式化分区`mkfs.ext4 /dev/nvme0n1p9`
   
##### 挂载分区

 1. `mkswap /dev/nvme0n1p8` : 缓存
 
 2. `swapon /dev/nvme0n1p8`
 
 3. `mount /dev/nvme0n1p9 /mnt` ： 主分区
 
 4. `mkdir -p /mnt/boot/efi`  `mount /dev/nvme0n1p7 /mnt/boot/efi`：挂载efi分区
 
 5. `wifi-menu`：设置网络连接
 
 6. `vim /etc/pacman.d/mirrorlist`：将CN的放在最前面
 
##### 安装系统
 
 1. `pacstrap -i /mnt base base-devel linux linux-firmware`：安装系统和基本开发组件
 
 2. `genfstab -U /mnt >> /mnt/etc/fstab`：生成挂载信息，以后系统启动执行挂载
 
 3. `arch-chroot /mnt`：进入安装好的系统
 
##### 用户管理

 1. 不建议直接使用root用户操作，很多软件yaourt和chromium等不可以root执行
 
 2. `useradd -m -g users tr`：`-m`为生成主目录（不生产可能导致软件无法运行，因为无法在主目录写配置） `-g users`为将用户放入users组
 
 3. `passwd tr`：设置密码
 
 4. `chmod u+w /etc/sudoers`：给用户权限文件write权限
 
 5. `visudo`：在root下面写上类似语句`tr ALL=(ALL) ALL`，此操作可让用户执行root命令
 
 6. `passwd`：设置root密码
 
##### 时区+本地化+驱动设置

 1. `pacman -S root`：安装root管理
 
 2. `echo TrArch >/etc/hostname`：设置主机名
 
 3. `pacman -S vim , vim /etc/locale.gen`：！！本地化设置，将en_US和zn_CN等取消注释！！
 
 4. `locale-gen`：生成本地化配置
 
 5. `ln -s /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime`：设置时区
 
 6. `hwclock --localtime`：防止和windows时间冲突，将时间设为本机时间而不是UTC时间
 
 7. `pacman -S dialog wpa_supplicant netctl wireless_tools chcpcd`：安装网络工具，防止待会启动无法联网
 
 8. `pacman -S xf86-video-intel xf86-input-synaptics`：安装显卡和触摸板驱动
 
##### 重要！：设置启动管理器（bootloader）

启动项管理器负责管理启动项，在uefi启动表中添加启动项，在uefi配置界面可以看到。在linux中这是由`efibootmgr`管理的，`bootloader`是存放在`efi`系统格式分区的引导器，在机器通电后，首先根据启动项加载对应`bootloader`，`bootloader`负责继续引导操作系统内核的启动，在`kernel`真正启动前，会引导执行`initramfs：init ram file system`，确保内核能够访问文件系统，然后执行`init`线程启动内核。

1. 安装bootloader管理器：`pacman -S efibootmgr`

这里推荐两个常用的`bootloader`：grub2 和 refind

###### grub

1. `pacman -S grub-efi-x86_64`

2. `grub-install --efi-directory=/boot/efi/ --bootloader-id=archlinux --recheck`：--efi-directory 参数指定自己的efi分区位置，`bootloader`都是存放在efi分区内单独管理的，如果没有按照前文操作，现在划点空间生成一个efi分区挂载即可`mount /dev/... /boot/efi`。--bootloader-id：为生成的文件夹的名字，文件夹位于/boot/efi/EFI/archlinux

3. `pacman -S os-prober`：双系统检测windows的

4. `grub-mkconfig -o /boot/grub/grub.cfg`：生成引导文件，这步会自动识别boot/efi分区内的系统，生成引导文件。

5. `reboot`即可

###### rEFInd

推荐使用，比较美观，secureboot配置方便，配置简单，后面有讲解配置secureboot。

1. 在确保挂载了efi分区的情况下（mount /dev/.. /boot/efi）`pacman -S refind`

2. `pacman -S os-prober`

3. `refind-install`


##### 安装桌面环境

1. `pacman -S  accountsservice gvfs gvfs-mtp gvfs-afc ntfs-3g exfat-utils`：安装基本包

2. `pacman -S networkmanager network-manager-applet gnome-keyring`：安装网络管理器等

3. `systemctl enable NetworkManager.service`：开机自启动网络控制

4. `pacman -S gnome gnome-extra gdm`：安装gnome

5. `systemctl enable gdm `

6. 这时候重启，可能存在缺少字符`terminal`无法打开和无法显示中文文字的问题，下载中文字符:`pacman -S wqy-zenhei ttf-fireflysung` `locale-gen `

7. 重启后若还是无法打开`terminal`，按下`windows`键，打开`settings`，在`region&language`内添加英文系统，`logout`即可

#### surface-archlinux 自定义内核配置

##### 生成内核

使用了`refid`作为`bootloader`，如果没有最好下载配置一个。

1. `pacman -S wget`：安装下载器

2. `vim /etc/hosts`：防止githubcontent无法访问，文件内配置`151.101.76.133 raw.githubusercontent.com`

3. `wget https://raw.githubusercontent.com/linux-surface/linux-surface/master/pkg/keys/surface.asc` ：下载key文件

4. `pacman-key --add surface.asc `：添加key文件

5. `pacman -Syy`：更新下载库

6. `pacman -S linux-surface-headers linux-surface surface-ipts-firmware`：

```
NOTE:总共大概80m下载很慢的话，配置一个aria2，`vim /etc/pacman.conf `配置aria2加速，这样pacman下载的时候会显示下载地址，把地址拷贝下来，用其他软件下载，下载好后`pacman -U xxx`安装 或者 `rm /var/cache/pacman/pkg/linux-surface-*` 删除pacman下载缓存，`cp linux-surface-headers-5.6.14.arch1-1-x86_64.pkg.tar.zst  surface-ipts-firmware-20200402-1-any.pkg.tar.zst /var/cache/pacman/pkg/`，再执行`pacman -S linux-surface-headers linux-surface surface-ipts-firmware`
```
            
7. 内核安装完成，在/boot下面会多出`vmlinuz-linux-surface`等文件，此时`refind-install`会出问题，建议手动配置`refind`

##### refind手动配置启动项

以下所有操作建议在u盘arch内执行，否则可能无法执行

进入u盘的archlinux live cd

1. `wifi-menu`

2. `mount /dev/nvme0n1p9 /mnt`

3. `mount /dev/nvme01p7 /mnt/boot/efi`

4. `arch-chroot /mnt`

5. `vim  /boot/efi/EFI/refind/refind.conf`配置内容。

```
以下为配置的标准模板，需要注意的是`volume`,`loader`,`initrd`，`PARTUUID`，这四个要是配置不对就启动失败。

1. volume: 这个是识别地址，支持卷标（类似windows下的C盘D盘概念），也支持GUID，我们就用卷标就够了，这里的值应该填写/boot/vmlinuz-linux所在的地址，使用`ls -l /dev/disk/by-label/`看所有设备的label，我的存放boot文件夹的分区是`/dev/nvme0n1p9`，但是这个分区没有label,手动生成卷标`e2label /dev/nvme0n1p9 ArchLinux`，这时候就有卷标了

2. loader：这个是内核启动器地址，一般默认放在/boot/vmliunz-linux，自己可以ls看一下，不存在没有的情况，如果volume设置没问题了，正常设置路径即可

3. initrd：这个是内核初始化ram和文件系统以确保内核访问正常的工具，同上

4. PARTUUID：这个内核启动后的参数设置，也是linux系统所在的分区的UUID，我的系统在/dev/nvme0n1p9，使用`ls -l /dev/disk/by-partuuid/|grep nvme0n1p9`查看uuid，将uuid写入

最后我的设置如下

普通linux内核：

menuentry "ArchLinux" {
	icon     /EFI/refind/icons/os_arch.png
	volume   "Arch Linux"
	loader   /boot/vmlinuz-linux
	initrd   /boot/initramfs-linux.img
	options  "root=PARTUUID=你-自-己-的-UUID rw add_efi_memmap initrd=boot\intel-ucode.img initrd=boot\amd-ucode.img"
	submenuentry "Boot using fallback initramfs" {
		initrd /boot/initramfs-linux-fallback.img
	}
	submenuentry "Boot to terminal" {
		add_options "systemd.unit=multi-user.target"
	}
}

surface的linux内核：

menuentry "ArchLinux" {
	icon     /EFI/refind/icons/os_arch.png
	volume   "Arch Linux"
	loader   /boot/vmlinuz-linux-surface
	initrd   /boot/initramfs-linux-surface.img
	options  "root=PARTUUID=你-自-己-的-UUID rw add_efi_memmap initrd=boot\intel-ucode.img initrd=boot\amd-ucode.img"
	submenuentry "Boot using fallback initramfs" {
		initrd /boot/initramfs-linux-surface-fallback.img
	}
	submenuentry "Boot to terminal" {
		add_options "systemd.unit=multi-user.target"
	}
}
```

reboot应该可以看到手动配置的两个内核了，如果没错两个都是可以正常启动。`uname -a`如果是surface内核可以看到带surface字样。

#### 支持SecureBoot配置

SecureBoot原理，这东西的存在是为了防止在引导的时候被恶意软件控制，它的作用机制是使用`MOK（machine own key 系统内置签名）`来验证引导的东西。如果签名符合，那么系统准许引导，反之不许。

但是微软比较流氓，默认只允许自家的windows启动，但是也支持其他系统的启动引导（需要工具）。

首先进入SeruceBoot，开启第三方（third party）支持。这里用的是`rEFInd+shim`，`shim`支持签名和hash。另一个工具叫`PreLoader`，不做介绍

`shim原理`，secureboot启动会做一个验证，微软支持`shim`和`PreLoader`，所以检测到`shim`，由于信任机制会继续启动，`shim`来启动`shim`所信任的系统内核，所以这是一个信任链，那么`shim`怎么信任内核呢。两种方式，hash或者签名，如果shim内存在系统的hash（最简单的方法）或者签名，那么这个系统就是被信任的系统，可以被继续加载。这就是下面要做的配置。


##### shim签名

`shim`在`AUR`库内，这里用`yaourt`安装`shim`

1. `vim /etc/pacman.conf`:写入：
	```
	[archlinuxfr]
    Server = http://repo.archlinux.fr/$arch
	```
2. `pacman -Sy`同步库

3. `pacman -S yaourt`

4. `yaourt shim-signed `：安装shim

5. `pacman -S sbsigntools`：安装签名工具

6. `refind-install --shim /usr/share/shim-signed/shimx64.efi --localkeys`：配置refind的shim keys目录，可以在/boot/efi/EFI/refind/keys存放内核的签名了。

7. `sbsign --key /etc/refind.d/keys/refind_local.key --cert /etc/refind.d/keys/refind_local.crt --output /boot/vmlinuz-linux /boot/vmlinuz-linux`：可以在/boot/efi/EFI/refind/keys内看到签名，这里可以再执行几次签名其他自定义内核，本质上是将启动的内核引导文件添加签名信息。

NOTE: 实际可以不用添加key，直接refind启动界面，使用enroll hash，将/boot/vmlinuz-linux（或者其他内核文件）引入即可，系统启动bootloader，bootloader启动内核文件查看签名或者hash

8. `reboot`：打开secureboot支持第三方（third party），重启。进入shim控制界面，选择`Enroll KEY`，点击文件系统，找到`/boot/efi/EFI/refind/keys/.....key` 确定添加，然后重启即可。


#### 独显和集显交火

1. 参考 https://wiki.archlinux.org/index.php/Bumblebee_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)



#### 常用配置

#### 桌面环境Gnome

``` shell
sudo pacman -S gnome gnome-extra gdm xorg xorg-server xorg-xinit
sudo systemctl enable gdm
```

如果不需要开机自启动gnome，`systemctl disable gdm`

手动启动： `systemctl start gdm`

#### 输入法

```
sudo pacman -S fcitx fcitx-configtool fcitx-im fcitx-googlepinyin
cat > ~/.xprofile <<EOF
export LC_ALL=zh_CN.UTF-8
export XIM=fcitx
export XIM_PROGRAM=fcitx
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS="@im=fcitx"
EOF
```

#### 字体

```
sudo pacman -S wqy-microhei wqy-microhei-lite wqy-bitmapfont wqy-zenhei ttf-arphic-ukai ttf-arphic-uming adobe-source-han-sans-cn-fonts adobe-source-han-serif-cn-fonts noto-fonts-cjk ttf-dejavu ttf-liberation
fc-cache -fv
```

power line fonts:

```
git clone https://github.com/powerline/fonts.git --depth=1
cd fonts
./install.sh
cd ..
rm -rf fonts
```