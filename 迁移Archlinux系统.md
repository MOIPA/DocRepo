title: 迁移Archlinux系统
author: tr
tags:
  - Linux
categories:
  - Linux
date: 2022-10-17 07:54:00
---
# 迁移Archlinux系统到新电脑

<!--more-->

## 迁移前的准备

> 下载`usbWriter`和`Archlinux`的系统镜像
>
> 烧录镜像到usb内
>
> 准备容量足够的硬盘或者U盘存储系统所有文件
>
> 在被备份的系统上安装多线程压缩工具`sudo pacman -S zstd`

## 复制系统文件

> linux将所有东西都视为文件，设备也是文件，但是不同电脑设备不同，这个需要后期修改
>
> 插上容量足够的U盘到现有的linux系统，将`/`下的所有文件压缩到一个压缩包到U盘下，执行`sudo tar --use-compress-program=zstd -cvpf /mnt/usb<你的U盘挂载地址>/arch-backup.tgz --exclude=/proc --exclude=/lost+found --exclude=/mnt --exclude=/sys --exclude=/run/media  --exclude=/media  /`
> 
> 这一步比较耗费时间，我的100G空间执行了1个小时左右

## 在新电脑划分分区

> 注意：如果是双系统注意不要覆盖windows的分区
>
> 关闭`secureboot`,插入`archlinux`的系统盘，进入`liveCD`系统
>
> 这时候`lsblk`或者`fdisk -l`查看分区信息，对硬盘执行分区操作：`fdisk /dev/nvmen1`，具体如何分区可以参考安装篇
>
> 划分好分区后，开启缓存，挂载主分区到`/mnt`下，执行`mkdir /mnt/boot`再把boot分区挂载到`/mnt/boot`下
> 

## 解压系统文件到新电脑

> 这时电脑处在`liveCD`下，如果没有可用的U盘存储新系统，可以将两台电脑放在一个局域网下，通过`iwctl命令连接无线网络`，之后可以通过`sftp`命令，将旧电脑的`arch-backup.tgz`文件传输到新电脑
>
> 若有U盘则直接将`arch-backup.tgz`文件复制到`/`目录下，然后执行解压缩命令解压缩到`/mnt/`目录下：`sudo tar -z -c -T0 -18 -v -p -f - arch-backup.zstd -C /mnt`
> 
> 执行完毕后创建刚刚排除的文件夹：`sudo mkdir -pv /mnt/proc sudo mkdir -pv /mnt/sys sudo mkdir -pv /mnt/run sudo mkdir -pv /mnt/dev`
>

## 配置新电脑的硬件信息

> 至此新的系统已经复制到了新电脑，但是还没有配置新系统使用的硬件信息（缓存所在分区，系统主分区信息等），执行`sudo genfstab -U /mnt/ > /mnt/etc/fstab` 可以看到交换区信息等都已写入
>
> 配置系统新系统的引导程序使用的硬件地址，使用的引导软件现在还不知道启动的时候去哪个分区里面加载系统，所以要配置主分区的UUID到引导软件的配置里。
>
> 配置前首先切换到新系统：`arch-chroot /mnt`
>
> 如果和我一样使用`reFind`，那么只需修改`/boot/EFI/refind/refind.conf`的配置文件的信息，使用`ls -l /dev/disk/by-uuid`查看所有分区的UUID信息。
>
> 如果是`grub`引导，执行`grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader=ArchLinux --recheck

update-grub`
>

## 更新Initramfs配置双系统等

> 执行`mkinitcpio -P`
> 
> 挂载windows的ESP分区（很小 只有100
，里面有个Microsoft目录），把目录的Microsoft目录拷贝到/boot/EFI/下，将`Microsoft`文件夹拷贝到`/boot/EFI`下