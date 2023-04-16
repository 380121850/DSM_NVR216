### syno NVR216默认的UBOOT启动参数
```
bootcmd=run syno_showlogo; run syno_bootargs; run bootspi
bootdelay=1
baudrate=115200
ethaddr=00:00:23:34:45:66
ipaddr=192.168.1.10
serverip=192.168.1.2
netmask=255.255.254.0
bootfile="uImage"
bootspi=sf probe 0;sf read ${loadaddr_kernel} ${spi_pt_addr_kernel} ${spi_pt_size_kernel};sf read ${loadaddr_rootfs} ${spi_pt_addr_fs} ${spi_pt_size_fs};bootm ${loadaddr_kernel} ${loadaddr_rootfs}; 
spi_pt_addr_logo=0x0080000
spi_pt_size_logo=14315
spi_pt_addr_kernel=0x00B0000
spi_pt_size_kernel=0x02F0000
spi_pt_addr_fs=0x03A0000
spi_pt_size_fs=0x0430000
syno_showlogo=setvobg 0 0x00000000;startvo 0 36 10;setenv jpeg_addr 0xB8000000;setenv vobuf 0xB9000000;sf probe 0;sf read ${jpeg_addr} ${spi_pt_addr_logo} ${spi_pt_size_logo};decjpg; syno_startgx 0 ${vobuf};
loadaddr_kernel=0x82000000
loadaddr_rootfs=0x92000000
syno_mem=640M
syno_boot_dev=/dev/md0 rw init=/sbin/init
syno_mtd=hi_sfc:704k(RedBoot),3008k(zImage),4288k(rd.gz),64k(vendor),64k(RedBoot+Config),-(FIS+directory)
syno_hw_version=NVR216
syno_phys_memsize=1024
syno_extra_args= 
syno_mtd_layout=hi_sfc:704k(RedBoot),3008k(zImage),4288k(rd.gz),64k(vendor),64k(RedBoot+Config),-(FIS+directory)
syno_hdd_powerup_seq=2
syno_net_if_num=1
syno_bootargs=setenv bootargs console=ttyS0,115200 ip=off mem=${syno_mem} syno_hw_version=${syno_hw_version} syno_phys_memsize=${syno_phys_memsize} initrd=${loadaddr_kernel} root=${syno_boot_dev} mtdparts=${syno_mtd_layout} hd_power_on_seq=${syno_hdd_powerup_seq} ihd_num=${syno_hdd_powerup_seq} netif_num=${syno_net_if_num} flash_size=8 ${syno_extra_args} 
vo_luma=70
vo_contrast=50
jpeg_size=82709
syno_clear_env=sf probe 0; sf erase 0x007E0000 0x10000
stdin=serial
stdout=serial
stderr=serial
verify=n
ver=U-Boot 2010.06 (Jun 27 2017 - 11:57:38)
jpeg_addr=0xB8000000
vobuf=0xB9000000

```
### syno nvr216 启动参数设置命令  
```
setenv spi_pt_addr_logo 0x000C0000 
setenv spi_pt_size_logo 0x00040000  
setenv jpeg_size 0x00040000 

setenv jpeg_addr 0xB8000000 
setenv serverip 192.168.50.112 

setenv spi_pt_addr_kernel 0x00B0000 
setenv spi_pt_size_kernel 0x02F0000
setenv loadaddr_kernel 0x82000000
 
setenv spi_pt_addr_fs 0x03A0000 
setenv spi_pt_size_fs 0x0430000 
setenv loadaddr_rootfs 0x92000000 

setenv vo_luma 70 
setenv vo_contrast 50 
setenv vobuf 0xB9000000 

setenv syno_mem 640M 
setenv syno_hw_version NVR216 
setenv syno_phys_memsize 1024 

setenv syno_extra_args ' ' 
setenv syno_hdd_powerup_seq 2 
setenv syno_net_if_num 1 

setenv syno_boot_dev '/dev/md0 rw init=/sbin/init'
setenv syno_showlogo 'setvobg 0 0x00000000;startvo 0 36 10;sf probe 0;sf read ${jpeg_addr} ${spi_pt_addr_logo} ${spi_pt_size_logo};decjpg; startgx 0 ${vobuf} 3840 0 0 1920 1080;'

setenv syno_mtd_layout 'hinand:1M(RedBoot),8M(zImage),10M(rd.gz),128K(vendor),128K(RedBoot+Config),9M(FIS+directory),-(user)'
setenv mtdparts mtdparts=${syno_mtd_layout}

setenv syno_bootargs 'setenv bootargs console=ttyS0,115200 ip=off mem=${syno_mem} syno_hw_version=${syno_hw_version} syno_phys_memsize=${syno_phys_memsize} initrd=${loadaddr_kernel} root=${syno_boot_dev} mtdparts=${syno_mtd_layout} hd_power_on_seq=${syno_hdd_powerup_seq} ihd_num=${syno_hdd_powerup_seq} netif_num=${syno_net_if_num} flash_size=128 ${syno_extra_args}' 

setenv syno_boot 'nand read ${loadaddr_kernel} zImage ;nand read ${loadaddr_rootfs} rd.gz;bootm ${loadaddr_kernel} ${loadaddr_rootfs}; '
setenv bootcmd 'run syno_showlogo; run syno_bootargs; run syno_boot'

```

### 通过串口网口烧录固件

#### 一、NOR UBOOT和启动LOGO烧录
>  NOR FLASH分配： boot 512KB + 256KB ENV + 256KB LOGO + 1024KB NULL

##### UBOOT烧录
```
mw.b 0x82000000 ff 0x002f0000;tftp 0x82000000 uboot_NVR216.bin;sf probe 0;sf erase 0x00 0x00080000;sf write 0x82000000 0x00000000 0x00080000

```
##### 启动LOGO烧录，logo限制为1080P大小的JPG图片，大小不超过256KB
```
mw.b 0x82000000 ff 0x002f0000;tftp 0x82000000 logo.jpg;sf probe 0;sf erase 0x000C0000 0x00040000;sf write 0x82000000 0x000C0000 0x00040000

```

#### 二、NAND 128MB DSM引导系统烧录
> 1M(RedBoot),8M(zImage),10M(rd.gz),128K(vendor),128K(RedBoot+Config),9M(FIS+directory),-(user)
```
nand erase RedBoot;tftp 0x82000000 uboot_NVR216.bin;nand write 0x82000000 RedBoot;

nand erase zImage;tftp 0x82000000 zImage;nand write 0x82000000 zImage;

nand erase rd.gz;tftp 0x82000000 rd.bin;nand write 0x82000000 rd.gz;

nand erase vendor;tftp 0x82000000 vendor.bin;nand write 0x82000000 vendor;

nand erase RedBoot+Config;tftp 0x82000000 env.bin;nand write 0x82000000 RedBoot+Config;

```

dd if=rd.bin of=ramdisk.lzma bs=64 skip=1;lzma -d ramdisk.lzma

mkdir -p rootfs;cd rootfs;sudo cpio -idv < ../ramdisk

find . |cpio -ov -H newc |lzma >../ramdisk.lzma

mkimage -n "synology_hi3535_nvr216 15284" -A arm -O linux -T ramdisk -C none -a 0x44000000 -e 0x44000000 -d ramdisk.lzma rd.bin

sudo diff -uNr --no-dereference  rd_src/rootfs rd_modify/rootfs > rd.patch

sudo patch -p2 < ../../rd.patch

sudo diff -uNr --no-dereference hda1_src hda1_modify > hda1.patch


	 
synochecksum-emu1 VERSION zImage updater uboot_NVR216.bin uboot_do_upd.sh rd.bin hda1.tgz indexdb.txz synohdpack_img.txz \
packages/SurveillanceStation-hi3535-8.1.1-5408.spk packages/FileStation-hi3535-1.1.4-0123.spk \
texts/chs/strings texts/cht/strings texts/csy/strings texts/dan/strings texts/enu/strings texts/fre/strings \
texts/ger/strings texts/hun/strings texts/ita/strings texts/jpn/strings texts/krn/strings texts/nld/strings \
texts/nor/strings texts/plk/strings texts/ptb/strings texts/ptg/strings texts/rus/strings texts/spn/strings \
texts/sve/strings texts/tha/strings texts/trk/strings > checksum.syno


synochecksum-emu1 VERSION  updater hda1.tgz indexdb.txz synohdpack_img.txz \
packages/SurveillanceStation-hi3535-8.1.1-5408.spk packages/FileStation-hi3535-1.1.4-0123.spk \
texts/chs/strings texts/cht/strings texts/csy/strings texts/dan/strings texts/enu/strings texts/fre/strings \
texts/ger/strings texts/hun/strings texts/ita/strings texts/jpn/strings texts/krn/strings texts/nld/strings \
texts/nor/strings texts/plk/strings texts/ptb/strings texts/ptg/strings texts/rus/strings texts/spn/strings \
texts/sve/strings texts/tha/strings texts/trk/strings > checksum.syno

mkdir -p /nfs;ifconfig eth0 192.168.50.125;mount -t nfs 192.168.50.100:/volume3/docker/ /nfs/


docker run -it -v /volume3/docker/opt/dsm/spksrc:/spksrc --name spksrc synocommunity/spksrc /bin/bash

docker run -it -v /volume3/docker/opt/dsm/spksrc:/spksrc  synocommunity/spksrc /bin/bash

docker run -it -v /root/spksrc:/spksrc synocommunity/spksrc /bin/bash


Synology DS Default Reset Util.
Usage: synodsdefault
  --reset-config Reset DSM configs.
  --reinstall Reset to reinstall DSM.
  --factory-default Reset to brand new DSM. Include remove all data volumes.
  --help Show this help.
root@LeeStationNVR:/# sudo /usr/syno/sbin/synodsdefault ==fact
root@LeeStationNVR:/# sudo /usr/syno/sbin/synodsdefault --reinstall


setenv bootargs console=ttyS0,115200 mem=512M syno_hw_version=NVR216 syno_phys_memsize=512  root=/dev/sda1 rw rootwait init=/sbin/init mtdparts=hi_sfc:704k(RedBoot),3008k(zImage),4288k(rd.gz),64k(vendor),64k(RedBoot+Config),-(FIS+directory) hd_power_on_seq=2 ihd_num=5 netif_num=1 flash_size=16;

setenv bootcmd 'tftp 0x82000000 uImage;bootm 0x82000000'


setenv bootargs console=ttyS0,115200 mem=512M syno_hw_version=NVR216 syno_phys_memsize=512  root=/dev/sdq2 rw rootwait rootfstype=ext4 init=/sbin/init mtdparts=hi_sfc:704k(RedBoot),3008k(zImage),4288k(rd.gz),64k(vendor),64k(RedBoot+Config),-(FIS+directory) hd_power_on_seq=2 ihd_num=5 netif_num=1 flash_size=16;

setenv bootargs console=ttyS0,115200 mem=512M syno_hw_version=NVR216 syno_phys_memsize=512  root=/dev/sdq2 rw rootwait rootfstype=ext3 init=/sbin/init mtdparts=hi_sfc:704k(RedBoot),3008k(zImage),4288k(rd.gz),64k(vendor),64k(RedBoot+Config),-(FIS+directory) hd_power_on_seq=2 ihd_num=5 netif_num=1 flash_size=16;


setenv bootcmd 'usb start; fatload usb 0:1 0x82000000 uImage; bootm 0x82000000'

setenv bootcmd 'usb start; fatload usb 0:2 0x82000000 uImage; bootm 0x82000000'


DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true LC_ALL=C LANGUAGE=C LANG=C chroot /mnt

setenv bootargs console=ttyS0,115200 mem=320M syno_hw_version=NVR216 syno_phys_memsize=512 initrd=0x82000000 root=/dev/md0 rw init=/sbin/init mtdparts=hi_sfc:704k(RedBoot),3008k(zImage),4288k(rd.gz),64k(vendor),64k(RedBoot+Config),-(FIS+directory) hd_power_on_seq=2 ihd_num=5 netif_num=1 flash_size=16;

sf probe 0;sf read 82000000 0x000B0000 0x002f0000;sf read 84000000 0x003A0000 0x00430000;bootm 0x82000000 0x84000000


auto eth0
iface eth0 inet dhcp


sudo debootstrap --arch=armel --foreign --include=sudo,vim,telnetd,net-tools,wget,transmission-common,ssh,samba,nfs-common bullseye debian_bullseye http://mirrors.ustc.edu.cn/debian/

sudo cp /usr/bin/qemu-arm-static debian_bullseye/usr/bin

sudo DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true LC_ALL=C LANGUAGE=C LANG=C chroot debian_bullseye debootstrap/debootstrap --second-stage http://mirrors.ustc.edu.cn/debian/

diff -c -a -b -B -r -q 
diff -c -a -b -B -r -q /nfs/opt/dsm/TVT_DVR_HI3535/modify/hda1/hda1_modify/lib/ /lib | grep diff

sudo debootstrap --arch=armel --foreign --include=qbittorrent-nox,usbutils,ntp,htop,sudo,vim,telnetd,net-tools,wget,transmission-common,ssh,samba,nfs-common buster debian_buster http://mirrors.ustc.edu.cn/debian/

sudo cp /usr/bin/qemu-arm-static debian_buster/usr/bin

sudo DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true LC_ALL=C LANGUAGE=C LANG=C chroot debian_buster debootstrap/debootstrap --second-stage http://mirrors.ustc.edu.cn/debian/

mkdir -p /nfs;ifconfig eth0 192.168.50.86;mount -t nfs 192.168.50.100:/volume3/docker/ /nfs/

DEBIAN_FRONTEND=noninteractive DEBCONF_NONINTERACTIVE_SEEN=true LC_ALL=C LANGUAGE=C LANG=C chroot debian_bullseye debootstrap/debootstrap --second-stage http://mirrors.ustc.edu.cn/debian/

U-Boot 2010.06 (Jun 26 2014 - 17:29:29)

NAND:  Check nand flash controller v504. found
Special NAND id table Version 1.36
Nand ID: 0x98 0xD1 0x90 0x15 0x76 0x14 0x01 0x00
Block:128KB Page:2KB Chip:128MB*1 OOB:64B ECC:4bits/512Byte 
128 MiB
Check spi flash controller v350... Found
Spi(cs1) ID: 0xC2 0x20 0x15 0xC2 0x20 0x15
Spi(cs1): Block:64KB Chip:2MB Name:"MX25L1606E"
*** Warning - bad CRC, using default environment

In:    serial
Out:   serial
Err:   serial
current hi3535 version 201312131026
cur device nand size 131072, spi size 2097152
current used param offset 11111111111111111
current do_check_flash_boot_param
current prod net 1000m
init gpio led setting
dev 0 set background color!
dev 2 set background color!
dev 0 opened!
dev 2 opened!
jpeg decoding ...
<<addr=0x80080000, size=0x9c13, vobuf=0x8fd00000>>
mmu_enable
<<imgwidth=480, imgheight=320, linebytes=960>>
decode success!!!!
dev 0 set background color!
dev 2 set background color!
dev 0 opened!
vo hd 0 end
dev 2 opened!
vo cvbs end
graphic layer 0 opened!
graphic layer 2 opened!
USB:   scanning bus for devices... 1 USB Device(s) found
0 Storage Device(s) found
MAC:   00-00-23-34-45-66
Hit any key to stop autoboot:  0 
hisilicon # print
bootdelay=1
baudrate=115200
ethaddr=00:00:23:34:45:66
ipaddr=192.168.1.10
serverip=192.168.1.2
netmask=255.255.254.0
bootfile="uImage"
stdin=serial
stdout=serial
stderr=serial
verify=n
bootargs=mem=496M console=ttyAMA0,115200 root=/dev/mtdblock2 rootfstype=yaffs2 mtdparts=hi_sfc:2M(boot);hinand:4M(kernel),64M(rootfs),28M(user),32M(nva010000)
bootcmd=nand read 0x82000000 0x0 0x400000;bootm 0x82000000
phyintfx=0
jpeg_addr=0x80080000
jpeg_size=0x9c13
vobuf=0x8fd00000
ver=U-Boot 2010.06 (Jun 26 2014 - 17:29:29)

Environment size: 513/131068 byte


setenv bootargs root=LABEL=ROOTFS rootflags=data=writeback rw console=ttyAML0,115200n8 console=tty0 no_console_suspend consoleblank=0 fsck.fix=yes fsck.repair=yes net.ifnames=0

fatload usb 0 ${dtb_mem_addr} dtb/amlogic/s905l2-stb.img
fatload usb 0 ${loadaddr} uInitrd
fatload usb 0 ${loadaddr} zimage
if fatload usb 0 0x1000000 u-boot.ext; then go 0x1000000; fi;
bootm ${loadaddr}  ${dtb_mem_addr}

docker run -d -p 9000:9000 --name portainer -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer:linux-arm64

sed -i s/"string1"/"string2"/g `grep "string1" -rl ./`