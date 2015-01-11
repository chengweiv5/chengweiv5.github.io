---
layout: post
title: MBR -- 磁盘主引导记录分析
tags:
  - ubuntu
---

在x86平台下，以BIOS启动磁盘操作系统时，BIOS首先会载入MBR，然后将控制权交给装载进内存的bootloader。bootloader接着查找标记为bootable/active的分区，然后加载VBR(Volume Boot Record)，然后继续引导过程。

MBR为磁盘的第一个扇区，大小512字节，主要包括以下几方面内容：
- bootloader， 440字节
- 磁盘签名
- 磁盘分区表
- MBR签名

下面分析一个具体的MBR， 它安装了syslinux bootloader。使用dd可以轻松获得磁盘的MBR，例如

{% highlight bash %}
#dd if=/dev/sda of=sda.mbr bs=512 count=1
{% endhighlight %}

使用xxd可以以16进制方式查看，也可以在vi中打开后执行:%!xxd来查看，方便编辑和保存。

{% highlight bash %}
字节地址    说明
0-439 bootloader写入的加载程序
0000000: fa31 c08e d88e d0bc 007c 89e6 0657 8ec0  .1.......|...W..
0000010: fbfc bf00 06b9 0001 f3a5 ea1f 0600 0052  ...............R
0000020: 52b4 41bb aa55 31c9 30f6 f9cd 1372 1381  R.A..U1.0....r..
0000030: fb55 aa75 0dd1 e973 0966 c706 8d06 b442  .U.u...s.f.....B
0000040: eb15 5ab4 08cd 1383 e13f 510f b6c6 40f7  ..Z......?Q...@.
0000050: e152 5066 31c0 6699 e866 00e8 2101 4d69  .RPf1.f..f..!.Mi
0000060: 7373 696e 6720 6f70 6572 6174 696e 6720  ssing operating 
0000070: 7379 7374 656d 2e0d 0a66 6066 31d2 bb00  system...f`f1...
0000080: 7c66 5266 5006 536a 016a 1089 e666 f736  |fRfP.Sj.j...f.6
0000090: f47b c0e4 0688 e188 c592 f636 f87b 88c6  .{.........6.{..
00000a0: 08e1 41b8 0102 8a16 fa7b cd13 8d64 1066  ..A......{...d.f
00000b0: 61c3 e8c4 ffbe be7d bfbe 07b9 2000 f3a5  a......}.... ...
00000c0: c366 6089 e5bb be07 b904 0031 c053 51f6  .f`........1.SQ.
00000d0: 0780 7403 4089 de83 c310 e2f3 4874 5b79  ..t.@.......Ht[y
00000e0: 3959 5b8a 4704 3c0f 7406 247f 3c05 7522  9Y[.G.<.t.$.<.u"
00000f0: 668b 4708 668b 5614 6601 d066 21d2 7503  f.G.f.V.f..f!.u.
0000100: 6689 c2e8 acff 7203 e8b6 ff66 8b46 1ce8  f.....r....f.F..
0000110: a0ff 83c3 10e2 cc66 61c3 e862 004d 756c  .......fa..b.Mul
0000120: 7469 706c 6520 6163 7469 7665 2070 6172  tiple active par
0000130: 7469 7469 6f6e 732e 0d0a 668b 4408 6603  titions...f.D.f.
0000140: 461c 6689 4408 e830 ff72 1381 3efe 7d55  F.f.D..0.r..>.}U
0000150: aa0f 8506 ffbc fa7b 5a5f 07fa ffe4 e81e  .......{Z_......
0000160: 004f 7065 7261 7469 6e67 2073 7973 7465  .Operating syste
0000170: 6d20 6c6f 6164 2065 7272 6f72 2e0d 0a5e  m load error...^
0000180: acb4 0e8a 3e62 04b3 07cd 103c 0a75 f1cd  ....>b.....<.u..
0000190: 18f4 ebfd 0000 0000 0000 0000 0000 0000  ................
00001a0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00001b0: 0000 0000 0000 0000 
{% endhighlight %}

440-443 磁盘签名, 0x85dc6b88
    886b dc85 

444-445 无用
  0000 

446-509 磁盘分区，4条记录，每条长16字节
解读第一条记录为例：

{% highlight bash %}
0 80: bootable
1-3 第一个扇区的CHS地址
4 83，分区类型
5-7 最后一个扇区的CHS地址
8-11 第一个扇区的LBA地址
12-15 分区大小（单位为扇区）
0x000620d8H --> 401624
8000  .........k......
00001c0: 0200 83fe 3f18 0100 0000 d820 0600 0000  ....?...... ....
00001d0: 0119 83fe ffff d920 0600 2330 9a12 00fe  ....... ..#0....
00001e0: ffff 82fe ffff fc50 a012 c539 0100 0000  .......P...9....
00001f0: 0000 0000 0000 0000 0000 0000 0000 

510-511 MBR签名 0xaa55
   55aa  ..............U.
0000200: 0a                                       .
{% endhighlight %}

下面是fdisk -l /dev/sda的结果。

{% highlight bash %}
Disk /dev/sda: 160.0 GB, 160041885696 bytes
255 heads, 63 sectors/track, 19457 cylinders, total 312581808 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x85dc6b88


   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *           1      401624      200812   83  Linux
/dev/sda2          401625   312496379   156047377+  83  Linux
/dev/sda3       312496380   312576704       40162+  82  Linux swap / Solaris
{% endhighlight %}
