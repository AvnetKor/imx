

uuu  uuu.auto




$ cat /proc/partitions
major minor  #blocks  name

   7        0      15100 loop0
   7        1      45960 loop1
   7        2       4300 loop2
   7        3        956 loop3
   7        4      93504 loop4
   7        5      55952 loop5
   7        6       3736 loop6
   7        7      93560 loop7
  11        0    1048575 sr0
   8        0  536870912 sda
   8        1  536868864 sda1
   7        8       3736 loop8
   7        9       4300 loop9
   7       10      55952 loop10
   7       11     163996 loop11
   7       12      45240 loop12
   7       13       1008 loop13
   7       14      15100 loop14
   7       15     160440 loop15
   8       16   15558144 sdb
   8       17   15554048 sdb1
   
   Copying the full SD card image
$ sudo dd if=<image name>.wic of=/dev/sdx bs=1M && sync

Partitioning the SD/MMC card
$ sudo fdisk /dev/sdb
p,d,n,p,1,20480,1024000,p n,p,2,1228800,enter,p,w

$ sudo dd if=<U-Boot image> of=/dev/sdx bs=1k seek=<offset> conv=fsync
Where offset is:
• 1 - for i.MX 6 or i.MX 7
• 33 - for i.MX 8QuadMax A0, i.MX 8QuadXPlus A0, i.MX 8M Quad, and i.MX 8M Mini
• 32 - for i.MX 8QuadXPlus B0 and i.MX 8QuadMax B0

$ sudo dd if=u-boot.bin-sd of=/dev/sdb bs=1k seek=33 conv=fsync

$ sudo dd if=u-boot-spl.bin-imx8mqevk-sd of=/dev/sdb bs=1k seek=33 conv=fsync



bs = block size  seek = target memory skip
$ sudo mkfs.ext4 /dev/sdb2
$ mkdir /home/avnet/mountpoint
$ sudo mount /dev/sdb2 /home/avnet/mountpoint
cd /home/avnet/rootfs
$ tar -jxvf core-image-minimal-imx8mqevk-20191205003335.rootfs.tar.bz2 

$ sudo cp -a * /home/avnet/mountpoint
$ sudo umount /home/avnet/mountpoint
$ sudo umount /home/avnet/rootfs
$ sync
