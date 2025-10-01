---
title: "Build Debian"
date: 2025-10-01 20:12:24 +0800
categories: [Linux]
tags: [Linux]
---
Link: https://forum.digikey.com/t/debian-getting-started-with-the-beaglebone-black/12967

# ARM Cross Compiler: GCC
```c
#user@localhost:~$
wget -c https://mirrors.edge.kernel.org/pub/tools/crosstool/files/bin/x86_64/11.3.0/x86_64-gcc-11.3.0-nolibc-arm-linux-gnueabi.tar.xz
tar -xf x86_64-gcc-11.3.0-nolibc-arm-linux-gnueabi.tar.xz // giải nén
export CC=`pwd`/gcc-11.3.0-nolibc/arm-linux-gnueabi/bin/arm-linux-gnueabi- //để lần sau thay vì gõ lại dòng dài này thì chỉ cần CC
```
Tải file build về

**Test Cross Compiler:**
```c
#user@localhost:~$
${CC}gcc --version
```

**Test Output**
```c
#Test Output:
arm-linux-gnueabi-gcc (GCC) 11.3.0
Copyright (C) 2021 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

# Bootloader: U-Boot
Build U-boot, cài tools như này nhớ chạy trước `sudo apt update`
```c
#Requirements
sudo apt install bison build-essential flex swig
```

Download U-boot

```c
#user@localhost:~$
git clone -b v2022.04 https://github.com/u-boot/u-boot --depth=1
cd u-boot/
```

**Patches:**
apply một pack cho beaglebone
```c
#user@localhost:~/u-boot$
git pull --no-edit https://git.beagleboard.org/beagleboard/u-boot.git v2022.04-bbb.io-am335x-am57xx
```
Config and Build:
```c
#user@localhost:~/u-boot$
make ARCH=arm CROSS_COMPILE=${CC} distclean // build lần đầu không cần clean
make ARCH=arm CROSS_COMPILE=${CC} am335x_evm_defconfig // code uboot nhiều tính năng ví dụ hiển thị logo lên màn hình khi khởi động
make ARCH=arm CROSS_COMPILE=${CC}
```
sản phẩm cuối cùng của u-boot là file `.img` và file `.mlo`.

# Linux Kernel
```c
#~/
git clone https://github.com/RobertCNelson/bb-kernel ./kernelbuildscripts
cd kernelbuildscripts/
```
Dùng kernel v5.4
```c
#user@localhost:~/kernelbuildscripts$
git checkout origin/am33x-v5.4 -b tmp
```

Build Kernel
```c
#user@localhost:~/kernelbuildscripts$
./build_kernel.sh
```
Khi build này nó bị revert hết tải lại code mới build lại từ đầu nên chạy rất lâu.

**Lỗi**
do hệ thống make file không tốt
![alt text](image-13.png)

```c
sudo apt-get install lzop
```

sau khi build xong nếu muốn biết version
```c
#user@localhost:~/kernelbuildscripts$
ls deploy/
```

Done build Kernel

# Root File System
Trong linux ngoài Kernel còn các thành phần khác như app, UI, ... các thao tác người dùng còn kernel chỉ dùng để điều khiển phần cứng. Những cái ngoài app, Ui, ... gọi là root file system.

Downloa root file system đã được build sẵn
```c
#user@localhost:~$
wget -c https://rcn-ee.com/rootfs/eewiki/minfs/debian-11.5-minimal-armhf-2022-10-06.tar.xz
```
Verify:
```c
#user@localhost:~$
sha256sum debian-11.5-minimal-armhf-2022-10-06.tar.xz
```
```c
#sha256sum output:
0ea53b23483af4bcccca53f3fd907b5e404f2f607a6577d78e8c4daf5b869ad0  debian-11.5-minimal-armhf-2022-10-06.tar.xz
```
Extract:
```c
#user@localhost:~$
tar xf debian-11.5-minimal-armhf-2022-10-06.tar.xz
```

# Setup microSD card
```c
#Example: for DISK=/dev/sdX
lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 465.8G  0 disk
├─sda1   8:1    0   512M  0 part /boot/efi
└─sda2   8:2    0 465.3G  0 part /                <- Development Machine Root Partition
sdb      8:16   1   962M  0 disk                  <- microSD/USB Storage Device
└─sdb1   8:17   1   961M  0 part                  <- microSD/USB Storage Partition
```
mất công gõ tên nên export lần sau chỉ cần gõ Disk
```c
#Thus you would use:
export DISK=/dev/sdb
```

Xóa hết tự động trong thẻ
```c
sudo dd if=/dev/zero of=${DISK} bs=1M count=10
```
Install Bootloader:
```c
#user@localhost:~$
sudo dd if=./u-boot/MLO of=${DISK} count=2 seek=1 bs=128k
sudo dd if=./u-boot/u-boot-dtb.img of=${DISK} count=4 seek=1 bs=384k
```
chip bật nguồn lên CPU sẽ nạp boot ROM trong chip BOOT nạp từ khi chip trong nhà máy.

![alt text](image-14.png)

```c
#Check the version of sfdisk installed on your pc is atleast 2.26.x or newer.
sudo sfdisk --version
```

```c
#Example Output
sfdisk from util-linux 2.27.1
```

```c
#sfdisk >= 2.26.x
sudo sfdisk ${DISK} <<-__EOF__
4M,,L,*
__EOF__
```

```c
#mkfs.ext4 -V
sudo mkfs.ext4 -V
mke2fs 1.43-WIP (15-Mar-2016)
        Using EXT2FS Library version 1.43-WIP
```

```c
#mkfs.ext4 >= 1.43
for: DISK=/dev/mmcblkX
sudo mkfs.ext4 -L rootfs -O ^metadata_csum,^64bit ${DISK}p1
 
for: DISK=/dev/sdX
sudo mkfs.ext4 -L rootfs -O ^metadata_csum,^64bit ${DISK}1
```

# Mount:
Mount là hình thức kết nối cây thư mục máy ảo với thư mục trên thẻ nhớ thì gọi là mount.

```c
sudo mkdir -p /media/rootfs/
 
for: DISK=/dev/mmcblkX
sudo mount ${DISK}p1 /media/rootfs/
 
for: DISK=/dev/sdX
sudo mount ${DISK}1 /media/rootfs/
```
## Copy Root File System
```c
#Debian; Root File System: user@localhost:~$
sudo tar xfvp ./debian-*-*-armhf-*/armhf-rootfs-*.tar -C /media/rootfs/
sync ///đẩy từ ram xuống thẻ vì cơ chế cache
```
## Set uname_r in /boot/uEnv.txt
```c
#user@localhost:~$
sudo sh -c "echo 'uname_r=${kernel_version}' >> /media/rootfs/boot/uEnv.txt"
```

## Copy Kernel Image
```c
#user@localhost:~$
sudo cp -v ./kernelbuildscripts/deploy/${kernel_version}.zImage /media/rootfs/boot/vmlinuz-${kernel_version}
```

## Copy Kernel Device Tree Binaries

```c
#user@localhost:~$
sudo mkdir -p /media/rootfs/boot/dtbs/${kernel_version}/
sudo tar xfv ./kernelbuildscripts/deploy/${kernel_version}-dtbs.tar.gz -C /media/rootfs/boot/dtbs/${kernel_version}/
```

## Copy Kernel Modules
```c
#user@localhost:~$
sudo tar xfv ./kernelbuildscripts/deploy/${kernel_version}-modules.tar.gz -C /media/rootfs/
```
## File Systems Table (/etc/fstab)
```c
#user@localhost:~/$
sudo sh -c "echo '/dev/mmcblk0p1  /  auto  errors=remount-ro  0  1' >> /media/rootfs/etc/fstab"
```

## Remove microSD/SD card
```c
sync
sudo umount /media/rootfs
```