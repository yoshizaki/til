# Installing VyOS x64 in KVM.

VyOS 1.1.8 x64 を Ubuntu 16.04 KVM/Openvswitch環境下にインストール
Openvswitchのことは省略して、br0-ext, br1-extを設定済み。
細かいところは今回は気にしない。

```shell
$ sudo qemu-img create -f raw /var/lib/libvirt/images/vyos-1.1.8-amd64.img 2G
$ sudo virt-install \
>   --connect qemu:///system  \
>   --name VyOS_1.1.8 \
>   --vcpus 1 \
>   --ram 512 \
>   --network bridge=br0-ext,model=virtio,virtualport_type=openvswitch \
>   --network bridge=br1-ext,model=virtio,virtualport_type=openvswitch \
>   --file /var/lib/libvirt/images/vyos-1.1.8-amd64.img \
>   --file-size 2 \
>   --cdrom /var/lib/libvirt/images/vyos-1.1.8-amd64.iso \
>   --accelerate \
>   --nographics
```

インストール作業が始まるので、基本は全てdefault設定

```
WARNING  CD-ROMメディアでのインストールの場合、デフォルトではテキストコンソールに何も出力されません。--location を使用することをお勧めします。 --locationやCD-ROMの指定方法や記述例は man ページをご参照ください。

インストールの開始中...
ドメインを作成中...                                                                                    |    0 B  00:00:02
ドメイン VyOS_1.1.8 に接続しました
エスケープ文字は ^] です

ISOLINUX 4.02 debian-20101014  Copyright (C) 1994-2010 H. Peter Anvin et al

*************************************************************
*                                                           *
* Welcome to VyOS, an open source network operating system! *
*                                                           *
* Version: 1.1.8 (Helium)                                   *
*                                                           *
* Default login:    vyos                                    *
* Default password: vyos                                    *
*                                                           *
* To start installation, login and use command:             *
*     install image                                         *
*                                                           *
*************************************************************

Press control and F then 1 for help, or ENTER to
boot:

Welcome to VyOS - vyos ttyS0

vyos login: vyos
Password:
Linux vyos 3.13.11-1-amd64-vyos #1 SMP Sat Nov 11 12:10:30 CET 2017 x86_64
Welcome to VyOS.
This system is open-source software. The exact distribution terms for
each module comprising the full system are described in the individual
files in /usr/share/doc/*/copyright.
vyos@vyos:~$ install image
Welcome to the VyOS install program.  This script
will walk you through the process of installing the
VyOS image to a local hard drive.
Would you like to continue? (Yes/No) [Yes]: yes
Probing drives: OK
Looking for pre-existing RAID groups...none found.
The VyOS image will require a minimum 1000MB root.
Would you like me to try to partition a drive automatically
or would you rather partition it manually with parted?  If
you have already setup your partitions, you may skip this step

Partition (Auto/Parted/Skip) [Auto]:

I found the following drives on your system:
 sda    2147MB


Install the image on? [sda]:

This will destroy all data on /dev/sda.
Continue? (Yes/No) [No]: yes

How big of a root partition should I create? (1000MB - 2147MB) [2147]MB:

Creating filesystem on /dev/sda1: OK

Done!
Mounting /dev/sda1...
What would you like to name this image? [1.1.8]: OK.  This image will be named: 1.1.8
Copying squashfs image...
Copying kernel and initrd images...
Done!
I found the following configuration files:
    /config/config.boot
    /opt/vyatta/etc/config.boot.default
Which one should I copy to sda? [/config/config.boot]:

Copying /config/config.boot to sda.
Enter password for administrator account
Enter password for user 'vyos':
Retype password for user 'vyos':
I need to install the GRUB boot loader.
I found the following drives on your system:
 sda    2147MB


Which drive should GRUB modify the boot partition on? [sda]:

Setting up grub: OK
Done!
```

終わり


