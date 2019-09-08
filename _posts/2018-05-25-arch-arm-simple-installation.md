---
title: "Arch-ARM: Simple installation"
date:   2018-05-25 00:15:00 -0600
header:
    teaser: "https://raw.githubusercontent.com/martinsgmx/martinsgmx.github.io/master/assets/img/posts/2018-05-25-arch-arm-simple-installation/header.png"
---

> This guide is based on Arch Linux x64 and Raspberry Pi 2 B

Installing Arch-ARM on Raspberry Pi 2 B
------

First to all we need connect our SD-Card via an adapter to our PC. After that, we need to check what's devices is now recognized by system, type on terminal: fdisk -l 
In this case I used a 16 Gigabytes micros-SD.

```bash
[root@arch ~]: fdisk -l
...
Disk /dev/sdb: 14.6 GiB, 15707668480 bytes, 30679040 sectors
Units: sectors of 1 * 512 = 512 bytes
...
[root@arch ~]:
```

Well, the micro-SD is recognized by /dev/sdb. Now, the next step is format it and create two partitions.

> The next commands is for create the /boot/ (sdb1) partition on your SD Card.

> IMPORTANT: It's very important created the first partition as bootable partition!

```bash
[root@arch ~]: fdisk /dev/sdb

Welcome to fdisk (util-linux 2.32).
...

Command (m for help): o
Created a new DOS disklabel with disk identifier 0x000000000f0.

Command (m for help): n
Partition type
    p   primary (0 primary, 0 extended, 4 free)
    e   extended (container for logical partitions)
Select (default p): ''ENTER''

Using default response p.
Partition number (1-4, default 1): ''ENTER''
First sector (2048-30679039, default 2048): ''ENTER''
Last sector, +sectors or +size{K,M,G,T,P} (2048-30679039, default 30679039): +100M

Created a new partition 1 of type 'Linux' and of size 100 MiB.
Partition 1 contains a vfat signature.

Do you want to remove the signature? [Y]es/[N]o: Y

The signature will be removed by a write command.

Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): c
Changed type of particion 'Linux' to 'W95 FAT32 (LBA)'.

Command (m for help): a
Selected partition 1
The booteable flag on partition1 is enabled now.
```

> Now, create the last partition (sdb2), aka /

```bash
Command (m for help): n
Partition type
    p   primary (0 primary, 0 extended, 4 free)
    e   extended (container for logical partitions)
Select (default p): ''ENTER''

Using default response p.
Partition number (2-4, default 2): ''ENTER''
First sector (206848-30679039, default 206848): ''ENTER''
Last sector, +sectors or +size{K,M,G,T,P} (206848-30679039, default 30679039): ''ENTER''

Created a new partition 2 of type 'Linux' and of size 14.5 GiB.
Partition 2 contains a ext4 signature.

Do you want to remove the signature? [Y]es/[N]o: Y

The signature will be removed by a write command.

Command (m for help): w
The partitions table has been altered.
Syncing disks.

[root@arch ~]:
```

> IMPORTANT: If you don't see the warning: "Do you want to remove the signature? [Y]es/[N]o:" Don't worry, that warning it's because on my case, the SD  Card contains a previous Arch ARM installation.

Now, need to format the partitions with the correct format. For that, type mkfs.vfat for /dev/sdb1 and mkfs.ext4 for /dev/sdb1:

```bash
[root@arch ~]: mkfs.vfat /dev/sdb1
[root@arch ~]: mkfs.ext4 /dev/sdb2
...
Writing superblocks and filesystem accounting information: done.

[root@arch ~]:
```

Now, the SD is ready to install Arch-ARM. But, before that, we need create two temporally folders ./tmp/root/ and ./tmp/boot/ on our host. After that, we need mount the SD partitions in them.

|Directories|
|Host| |SD-Card|
|:-----:|:-:|:-------:|
|./tmp/boot/|<->|/dev/sdb1|
|./tmp/root/|<->|/dev/sdb2|

```bash
[root@arch ~]: mkdir ./tmp/boot/
[root@arch ~]: mkdir ./tmp/root/
[root@arch ~]: mount /dev/sdb1 ./tmp/boot/
[root@arch ~]: mount /dev/sdb2 ./tmp/root/
```

Now, download the latest version of Arch ARM rpi 2 and uncompressed with bsdtar.

```bash
[root@arch ~]: wget http://os.archlinuxarm.org/os/ArchLinuxARM-rpi-2-latest.tar.gz
[root@arch ~]: bsdtar -xvpf ArhLinuxARM-rpi-2-latest.tar.gz -C ./tmp/root/
```

Once bsdtar finished, in the same folder, we need move the files from: ./tmp/root/boot/ to ./tmp/boot/ and sync.

```bash
[root@arch ~]: mv ./root/boot/* ./boot/
[root@arch ~]: sync
```

The SD-card is ready to Raspberry. Unmount all partitions of host, and insert SD card to Raspberry board.

```bash
[root@arch ~]: umount /dev/sdb1 /dev/sdb2
```

The first boot!
------

After power-on the Raspberry we need connect to it via SSH, for that, we need know the IP address which is assigned by modem.
After know the IP, connect via SSH:

> The default user is: alarm and the password: alarm

> The default root user is: root and password: root

```bash
[root@arch ~]: ssh alarm@192.168.1.XX
```
> Changes 'xx' to IP assign by your router.

The essentials steps.
------

Now you're connected, we need configure somethings. Type su on the console and login with root (password: root)
Removed the default localtime, and linked to new:

> To list all zones available, type: ls /usr/share/zoneinfo/*

```bash
[root@192.168.1.XX ~]: rm /etc/localtime
[root@192.168.1.XX ~]: ln -s /usr/share/zoneinfo/America/Mexico_city /etc/localtime
```

Now, we need generate some environment variables and generate the locale. Uncomment your locale on locale.gen file:

> Tip: On nano editor, to save any change you need press 'CTRL + o' key, and 'CTRL + x' to quit.

```bash
[root@192.168.1.XX ~]: nano /etc/locale.gen
....
#en_DK ISO-8859-1
en_GB.UTF-8 UTF-8
#en_GB ISO-8859-1
...
[root@192.168.1.XX ~]: nano /etc/locale.conf
LANG=en_GB.UTF-8
[root@192.168.1.XX ~]: nano /etc/profile 
...
export LANGUAGE=en_GB.UTF-8
export LANG=en_GB.UTF-8
export LC_ALL=en_GB.UTF-8
...

[root@192.168.1.XX ~]: locale-gen
Generating locales...
  en_GB.UTF-8... done
Generation complete.
```

We need set clock to UTF and set timezone (in my case Mexico_city):

```bash
[root@192.168.1.XX ~]: timedatectl set-local-rtc 0
[root@192.168.1.XX ~]: nano /etc/timezone
```

> Optional step, config static IP

For set an static IP we need modify /etc/systemd/network/eth0.network The final result looks like this:

```bash
[root@192.168.1.XX ~]: nano /etc/systemd/network/eth0.network
[Match]
Name=eth0

[Network]
Address=192.168.1.xx
Gateway=192.168.1.254
DNS=8.8.4.4
```

Now, disable netctl and dhcp services:

```bash
[root@192.168.1.XX ~]: systemctl disable netctl
[root@192.168.1.XX ~]: systemctl stop dhcpcd
[root@192.168.1.XX ~]: systemctl disable dhcpcd
```

Enable and start networkd services:

```bash
[root@192.168.1.XX ~]: systemctl enable systemd-networkd
[root@192.168.1.XX ~]: systemctl start systemd-networkd
```

> If you want change the hostname, you need modified two files: /etc/hostname and /etc/hosts

```bash
[root@192.168.1.XX ~]: nano /etc/hostname
arch-rpi
[root@192.168.1.XX ~]: nano /etc/hosts
127.0.0.1 localhost.localdomain localhost arch-rpi
[root@192.168.1.XX ~]: reboot
```

System update, SUDO and Yaourt installation.
------

First to all, we need generate a pacman key, after that, you can upgrade your system:

```bash
[root@192.168.1.XX ~]: pacman-key --init
...
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
[root@192.168.1.XX ~]: pacman -Syu
...
:: Proceed with installation? [Y/n] y
...
:: Starting full system upgrade...
...
[root@192.168.1.XX ~]: reboot
```

After sudo installation we need uncomment somes lines in visudo: %wheel ALL=(ALL) ALL, %sudo ALL=(ALL) NOPASSWD: ALL, %sudo ALL=(ALL) NOPASSWD: ALL 

```bash
[root@192.168.1.XX ~]: pacman -S sudo
[root@192.168.1.XX ~]: visudo
...
##
## User privilege specification
##
root ALL=(ALL) ALL

## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL) ALL

## Same thing without a password
%wheel ALL=(ALL) NOPASSWD: ALL

## Uncomment to allow members of group sudo to execute any command
%sudo   ALL=(ALL) ALL
```

Now press ESC key and type :wq for save any change and quit.
Now, we need create a group called sudo and add our default user (on this case: alarm)

```bash
[root@192.168.1.XX ~]: groupadd sudo
[root@192.168.1.XX ~]: usermod -a -G sudo alarm
[root@192.168.1.XX ~]: exit
```

Now, install yaourt:

> You need have installed base-devel and wget. Only need type: sudo pacman -S base-devel wget

```bash
[alarm@192.168.1.XX ~]: wget http://aur.archlinux.org/cgit/aur.git/snapshot/package-query.tar.gz 
[alarm@192.168.1.XX ~]: tar -xvzf package-query.tar.gz
[alarm@192.168.1.XX ~]: cd package-query/
[alarm@192.168.1.XX ~]: makepkg -si
[alarm@192.168.1.XX ~]: cd ..
[alarm@192.168.1.XX ~]: wget http://aur.archlinux.org/cgit/aur.git/snapshot/yaourt.tar.gz
[alarm@192.168.1.XX ~]: tar -xvzf yaourt.tar.gz
[alarm@192.168.1.XX ~]: cd yaourt/
[alarm@192.168.1.XX ~]: makepkg -si
[alarm@192.168.1.XX ~]: cd ..
``` 

Well, your Arch ARM it's ready. If you want a graphic environment, check my other post: [Arch-ARM: XFCE and TigerVNC installation.]
On my case, don't need more, it's enough, only use my RPi to AVRdude and other Atmel utilities.

[Arch-ARM: XFCE and TigerVNC installation.]: https://martinsgmx.github.io/arch-arm-xfce4-tigervnc/