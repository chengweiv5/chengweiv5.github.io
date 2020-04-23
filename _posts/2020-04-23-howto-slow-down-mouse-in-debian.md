---
layout: post
title: 怎样修改 Debian 中鼠标移动的速度
tags:
  - mouse
---

在 debian 上被我的有线 USB 鼠标的移动速度困扰很久了，比正常的鼠标移动得快，
所以非常难操控，特别是在画图的时候。

可以使用下面 xinput 来修改鼠标的移动速度，关键是找到对应的鼠标以及对应的属性就行了。

下面是我电脑上的连接的输入设备，用 xinput 命令查看：

```
$ xinput list
⎡ Virtual core pointer                          id=2    [master pointer  (3)]
⎜   ↳ Virtual core XTEST pointer                id=4    [slave  pointer  (2)]
⎜   ↳ PS/2 Generic Mouse                        id=16   [slave  pointer  (2)]
⎜   ↳ SynPS/2 Synaptics TouchPad                id=17   [slave  pointer  (2)]
⎜   ↳ Logitech USB Receiver Consumer Control    id=9    [slave  pointer  (2)]
⎜   ↳ Logitech USB Receiver                     id=11   [slave  pointer  (2)]
⎜   ↳ USB Optical Mouse  Mouse                  id=13   [slave  pointer  (2)]
⎜   ↳ USB Optical Mouse  Consumer Control       id=20   [slave  pointer  (2)]
⎣ Virtual core keyboard                         id=3    [master keyboard (2)]
    ↳ Virtual core XTEST keyboard               id=5    [slave  keyboard (3)]
    ↳ Power Button                              id=6    [slave  keyboard (3)]
    ↳ Video Bus                                 id=7    [slave  keyboard (3)]
    ↳ Sleep Button                              id=8    [slave  keyboard (3)]
    ↳ HP HD Camera: HP HD Camera                id=14   [slave  keyboard (3)]
    ↳ AT Translated Set 2 keyboard              id=15   [slave  keyboard (3)]
    ↳ HP Wireless hotkeys                       id=18   [slave  keyboard (3)]
    ↳ HP WMI hotkeys                            id=19   [slave  keyboard (3)]
    ↳ Logitech USB Receiver Consumer Control    id=10   [slave  keyboard (3)]
    ↳ USB Optical Mouse  Keyboard               id=12   [slave  keyboard (3)]
    ↳ USB Optical Mouse  Consumer Control       id=21   [slave  keyboard (3)]
```

通过名字可以看到速度太快的鼠标是 `id=13`, `id=20` 那一对；但是要修改的是 `id=13` 那个，
查看它的属性，如下：

```
$ xinput --list-props 13
Device 'USB Optical Mouse  Mouse':
        Device Enabled (153):   1
        Coordinate Transformation Matrix (155): 1.000000, 0.000000, 0.000000, 0.000000, 1.000000, 0.000000, 0.000000, 0.000000, 1.000000
        libinput Natural Scrolling Enabled (288):       0
        libinput Natural Scrolling Enabled Default (289):       0
        libinput Scroll Methods Available (290):        0, 0, 1
        libinput Scroll Method Enabled (291):   0, 0, 0
        libinput Scroll Method Enabled Default (292):   0, 0, 0
        libinput Button Scrolling Button (293): 2
        libinput Button Scrolling Button Default (294): 2
        libinput Middle Emulation Enabled (295):        0
        libinput Middle Emulation Enabled Default (296):        0
        libinput Accel Speed (297):      0.000000
        libinput Accel Speed Default (298):     0.000000
        libinput Accel Profiles Available (299):        1, 1
        libinput Accel Profile Enabled (300):   1, 0
        libinput Accel Profile Enabled Default (301):   1, 0
        libinput Left Handed Enabled (302):     0
        libinput Left Handed Enabled Default (303):     0
        libinput Send Events Modes Available (273):     1, 0
        libinput Send Events Mode Enabled (274):        0, 0
        libinput Send Events Mode Enabled Default (275):        0, 0
        Device Node (276):      "/dev/input/event9"
        Device Product ID (277):        7119, 83
        libinput Drag Lock Buttons (304):       <no items>
        libinput Horizontal Scroll Enabled (305):       1
```

可以看到两个属性：

  - libinput Accel Speed (297):      0.000000
  - libinput Accel Speed Default (298):     0.000000

普通用户没有权限修改 `Default` 属性的值，只需要修改 `297` 这个属性的值即可，修改为负数表示减慢速度，
负数越大越慢，所以调整到一个合适的负数就行。

例如：

```
$ xinput --set-prop 13 297 -0.75
```

修改会立即生效，也可以通过名字来修改，并没有任何区别，如下：

```
$ xinput --set-prop "USB Optical Mouse  Mouse" "libinput Accel Speed" -0.75
```

但是这样修改后，鼠标插拔后会失效，系统重启后也会失效，xinput 并没有提供配置文件来持久化配置，
但是可以在 X 启动的时候执行，例如：把上面的命令写入 `~/.xinitrc` 中，注意：使用名字配置的那个版本，
因为 ID 会变。

这样就解决了重启失效的问题，但是鼠标插拔失效还是没有解决，可以借助 udev 规则来自动设置，在 `/etc/udev/rules.d`
目录下新建一个文件如下：

```
# cat 50-slow-usb-mouse-speed.rules
ACTION=="add", KERNEL=="event9", SUBSYSTEM=="input", ATTRS{name}=="USB Optical Mouse  Mouse", RUN+="/home/chengwei/.xinput-slow-mouse.sh"
```

上面的过滤条件根据情况调整，可以用 udevadm 命令来查看鼠标设备的这些属性，这里不再介绍。

然后 `.xinput-slow-mouse.sh` 脚本内容如下：

```
$ cat .xinput-slow-mouse.sh
#!/bin/bash

# at doesn't support now + seconds, use sleep
echo "DISPLAY=:0 su chengwei -c 'sleep 3 && xinput --set-prop \"USB Optical Mouse  Mouse\" \"libinput Accel Speed\" -0.75'" | at now
```

这里之所以要写这么麻烦，是因为 udev RUN 是一个阻塞的运行，它执行完之后，xinput 才能找到设备，
所以在泽哥脚本里要想用 xinput 设置属性是不可能的，所以引入了 `at` 命令；它会在指定的时间在后台运行命令。

但是，`at` 命令不支持几秒后执行，最小的粒度是分钟后，或者指定绝对时间，所以，这里使用了 `sleep` 命令。
