---
layout: post
title: MTD 设备及 JFFS2, UBIFS 文件系统的使用简介
tagy:
  - filesystem
---

#### MTD设备

MTD (Memory Technology Devices) 更多的代表Flash设备，它提供了统一接口来处理各种裸Flash设备，例如NAND， NOR， OneNAND等。对于使用FTL技术模拟成块设备的Flash，并不属于MTD设备。例如MMC, eMMC, SD等。
/dev/mtdN, MTD字符设备，代表MTD设备，可以进行各种ioctl操作
/proc/mtd, 过时的proc接口
sysfs接口是正在使用的接口，包括 /sys/devices,/sys/class,/sys/module 接口
mtdram, mtdblock，mtdram模拟NOR flash设备，而mtdblock将NOR flash模拟成块设备，但是它并不高级，远没有提供FTL的功能，例如写均衡，坏块管理等，它只是一个简单的模拟块设备的接口。通过对象的/dev/mtdblockN 可以以访问块设备的方式访问NOR flash。

mtdram 用内存模拟NOR Flash
block2mtd 用块设备模拟NOR Flash
nandsim 用内存会文件模拟NAND Flash
1. 首先要编译内核支持MTD相关选项，不管是内置还是编译为模块。
2. 安装mtd-utils工具
3.1. mtdram, mtdblock
#modinfo mtdram
可以看出mtdram有两个参数，total_size, erase_size，都以KiB为单位，下面创建一个32MiB的MTD设备，擦除块大小为128KiB
#modprobe mtdram total_size=$((32*1024)) erase_size=128
#cat /proc/mtd // 查看当前的mtd设备，另外，还可以通过/sys/devices/virtual/mtd/目录来查看MTD设备
#ls -l /dev/mtd* // 自动创建了 /dev/mtd0, /dev/mtd0ro 设备节点
#modprobe mtdblock
#ls -l /dev/mtd* // 自动创建了和 /dev/mtd0 对应的块设备节点 /dev/mtdblock0
3.2. block2mtd
block2mtd顾名思义使用块设备模拟MTD设备，这里的块设备既可以是真正的块设备，也可以是使用losetup模拟的块设备。
#dd if=/dev/zero of=block.img bs=4K count=8096 // 创建一个32M大小的文件
#losetup /dev/loop0 block.img // 模拟块设备文件，可以使用 losetup -f block.img, losetup -a，如果/dev/loop0已经被占用。
#modinfo block2mtd
可以看出block2mtd模块有一个参数block2mtd，设置块设备文件和擦除块大小
#modprobe block2mtd block2mtd=/dev/loop0,128KiB
#cat /proc/mtd
#ls -l /dev/mtd*
3.3. nandsim
nandsim模拟NAND flash设备，前两种都是模拟NOR flash设备，也可以使用mtdinfo <device>来查看，可以看出，通过mtdram，block2mtd模拟的flash设备最小输入输出单位是字节。
#modinfo nandsim
nandsim有一堆参数，其中first_id_byte, second_id_byte分别指明设备制造商ID和芯片ID，如果不指明，会创建一个默认128M大小的flash设备
#modprobe nandsim first_id_byte=0x20 second_id_byte=0x33
#cat /proc/mtd
#ls -l /dev/mtd*
#mtdinfo /dev/mtdN // 查看MTD的信息，例如这里创建的NAND flash信息为
Name: NAND simulator partition 0
Type: nand
Eraseblock size: 16384 bytes, 16.0 KiB
Amount of eraseblocks: 1024 (16777216 bytes, 16.0 MiB)
Minimum input/output unit size: 512 bytes
Sub-page size: 256 bytes
OOB size: 16 bytes
Character device major/minor: 90:4
Bad blocks are allowed: true
Device is writable: true
4. Flash文件系统
现在，有了MTD设备，还没有合适的文件系统，JFFS2和UBIFS是两个比较流行的flash文件系统。
4.1. JFFS2
JFFS2文件系统用户空间工具为mtd-utils，提供创建jffs2的工具mkfs.jffs2和jffs2dump。创建jffs2文件系统映像后可以用nand工具烧写到flash上。
#mkfs.jffs2 --root=jffs2.dir --pagesize=4KiB --eraseblock=128KiB --pad=$((32*1024))KiB --output=jffs2.img
上面命令以目录jffs2.dir为根创建了一个jffs2.img文件系统映像，--pad表示使用0xFF填充剩余的空间，如果没有参数表示填充到最后一块即止。这里填充到32MiB大小。
在Linux上，可以通过mtdram/mtdblock来查看修改jffs2.img。但是不能直接通过loop back来挂载，因为它工作在MTD之上，而不是块设备之上。
#modprobe mtdram total_size=$((1024*32)) erase_size=128
#modprobe mtdblock
#dd if=jffs2.img of=/dev/mtdblock0
#mount -t jffs2 /dev/mtdblock0 /mnt
或者使用nandsim模拟，然后使用nandwrite将jffs2.img写入MTD设备
#modprobe nandsim // 默认模拟一个128MiB的nand flash
#nandwrite -j /dev/mtd1 jffs2.img
#mount -t jffs2 /dev/mtdblock1 /mnt

UBIFS
UBIFS工作在UBI层之上，UBI是对MTD设备的又一层抽象。首先，要将MTD设备加入UBI池，然后再在UBI之上创建UBIFS，有点类似LVM的概念。
#modprobe nandsim // 创建一个nand flash设备，128MiB
#mtdinfo /dev/mtd0 // flash的物理擦除块大小为16KiB
#modprobe ubi mtd=0 // 将 /dev/mtd0 加入ubi设备池
或者
#modprobe ubi
#ubiattach /dev/ubi_ctrl -m 0 // 现在将会生成/dev/ubi0 设备
#ubinfo /dev/ubi0 // ubi设备的逻辑擦除块大小为15.5KiB, 15872B，这是因为创建ubi设备时需要在每个物理擦除块写入头。
#ubimkvol /dev/ubi0 -N ubi-vol0 -s 32MiB // 创建一个32MiB的volume,/dev/ubi0_0
#ubinfo /dev/ubi0_0
#mount -t ubifs ubi0:ubi-vol0 /mnt
#echo "hello ubifs" > /mnt/hello.txt
#mkfs.ubifs -r /mnt -m 512 -e 15782 -c 2115 -o ubifs.img
上面命令中的最小IO单位512字节，逻辑擦除块大小，以及文件系统最大可扩展至2115个逻辑块，这些信息可以使用如下命令获得
#ubinfo /dev/ubi0 // 获得最小IO单位，逻辑擦除块大小
#ubinfo /dev/ubi0_0 // 文件系统包含的逻辑块数
#cat ubinize.cfg
[ubifs]
mode=ubi
image=ubifs.img
vol_id=0
vol_size=32MiB
vol_type=dynamic
vol_name=ubi-vol0
vol_flags=autoresize
#ubinize -o ubi.img -m 512 -s 256 -p 16KiB ubinize.cfg
这里和mkfs.ubifs的参数不同，-p的参数为物理擦除块大小，也就是ubi工作在MTD层之上，所以需要MTD的参数，即物理参数，而ubifs工作在ubi之上，所以需要ubi的参数，即逻辑参数。
现在，ubi设备映像已经被保存在了ubi.img中，不仅包含ubifs信息，还包含ubi信息，所以可以直接烧写到MTD设备上即可。
#modprobe nandsim
#dd if=ubi.img /dev/mtd0
#modprobe ubi mtd=0
#mount -t ubifs ubi0_0 /mnt
#ls /mnt
hello.txt

 注意：如果出现ubiattach错误，很可能是使用了block2mtd模拟MTD设备，因为这时设备是没有格式化的，所以会出现ubi_read_volume_table: the layout volume was not found，通过dmesg查看。那么，首先需要格式化MTD设备
#ubiformat /dev/mtd0
#ubiattach /dev/ubi_ctrl -m 0

