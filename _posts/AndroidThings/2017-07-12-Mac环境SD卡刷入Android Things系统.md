---
layout: post
title: Mac环境SD卡刷入Android Things系统
categories: [android things]
description: 在Mac OS上，您可以选择命令行dd工具或使用图形工具ImageWriter将图像写入SD卡。

keywords: [android things, 树莓派, 嵌入式, 刷机]
date: 2017-07-12 15:00:00
author: 化作春泥
---

> 本文首发于[《简书》](http://www.jianshu.com/p/7ec93e11ae04)，转载请保留链接 : )

在Mac OS上，您可以选择命令行dd工具或使用图形工具ImageWriter将图像写入SD卡。

## 图形界面

* 将SD读卡器与SD卡连接。 注意，它必须格式化为FAT32。
* 从Apple菜单中，选择“关于本机”，然后单击“更多信息...”; 如果您使用的是Mac OS X 10.8.x Mountain Lion或更高版本，请点击“系统报告”。
* 单击“USB”（或“读卡器”，如果使用内置的SD卡读卡器），然后在窗口的右上部分搜索您的SD卡。 单击它，然后在右下部分中搜索BSD名称; 它将看起来像`diskn`，其中n是一个数字（例如，`disk4`）。 请务必记下此号码。
* 卸载分区，以便允许您覆盖磁盘。 为此，打开磁盘实用程序并卸载它; 但不要弹出它，否则你将不得不重新以上操作。 请注意，在Mac OS X 10.8.x Mountain Lion上，“验证磁盘”（卸载前）将显示BSD名称为`/dev/disk1s1`或类似，允许您跳过前两个步骤。
* 从终端运行以下命令：
```
sudo dd bs=1m if=path_of_your_image.img of=/dev/rdiskn
```
请记住将n替换为您之前注明的数字！

 + 如果此命令失败，请尝试使用disk替换rdisk：
```
sudo dd bs=1m if=path_of_your_image.img of=/dev/diskn
```

## 命令行

* 如果您习惯用命令行，可以通过命令后将镜像写入SD卡，而无需任何其他软件。 打开终端，然后运行：

```
diskutil list
```

* 确定SD卡的磁盘（而非分区），例如 disk4，而不是disk4s1。
* 使用磁盘标识符卸载SD卡，以准备将数据复制到其中：

```
diskutil unmountDisk /dev/disk<disk# from diskutil>
```

其中`disk`是您的BSD名称，例如 `diskutil unmountDisk/dev/disk4`

* 将数据复制到SD卡：

```
sudo dd bs=1m if=image.img of=/dev/rdisk<disk# from diskutil>
```

其中`disk`是您的BSD名称，例如 `sudo dd bs=1m if=2016-11-25-raspbian-jessie.img of=/dev/rdisk4`

 + 如果你安装了GNU coreutils，这可能导致 `dd: invalid number '1m'`。 在这种情况下，您需要在`bs =`节中使用`1M`的块大小，如下所示：

```
sudo dd bs=1M if=image.img of=/dev/rdisk<disk# from diskutil>
```

这将需要几分钟，具体取决于图像文件大小。 您可以通过发送SIGINFO信号来检查进度（按Ctrl + T）。

 + 如果此命令仍然失败，请尝试使用`disk`替换`rdisk`，例如：

```
sudo dd bs=1m if=2016-11-25-raspbian-jessie.img of=/dev/disk4
//or
sudo dd bs=1M if=2016-11-25-raspbian-jessie.img of=/dev/disk4
```

## 替代方法

__注意：有些用户报告了使用此方法创建SD卡的问题。__
这些命令和操作需要从具有管理员权限的帐户执行。

* 从终端运行`df -h`。
* 将SD读卡器与SD卡连接。
* 再次运行`df -h`并查找上次未列出的新设备。 记录文件系统分区的设备名称，例如`/dev/disk3s1`。
* 卸载分区，以便允许您覆盖磁盘：

```
sudo diskutil unmount /dev/disk3s1
```

或者，打开磁盘实用程序并卸载SD卡的分区; 但不要弹出它，否则你将不得不重新连接它。

* 使用分区的设备名称，通过省略最后的`s1`并用`rdisk`替换`disk`来计算整个磁盘的原始设备名称这是非常重要的，因为如果提供错误的设备名称，您将丢失硬盘驱动器上的所有数据 。确保设备名称是整个SD卡的名称，如上所述，而不只是它的一个分区 - 例如，`rdisk3`，而不是`rdisk3s1`。类似地，您可能有另一个SD驱动器名称/编号，如`rdisk2`或`rdisk4`; 您可以在将SD读卡器插入Mac之前和之后使用`df -h`命令再次检查。 例如，`/dev/disk3s1`变为`/dev/rdisk3`。

* 在终端中，使用此命令，使用上面的原始设备名称将镜像写入卡。 请仔细阅读上述步骤，以确保您在此处使用正确的`rdisk`号：

```
sudo dd bs=1m if=2016-11-25-raspbian-jessie.img of=/dev/rdisk3
```

如果上述命令报告错误`dd: bs: illegal numeric value`，请将块大小`bs=1m`更改为`bs=1M`。
如果上述命令报告错误`dd: /dev/rdisk3: Permission denied`，则表示SD卡的分区表正在受到保护，防止被Mac OS覆盖。 使用此命令擦除SD卡的分区表：

```
sudo diskutil partitionDisk /dev/disk3 1 MBR "Free Space" "%noformat%" 100%
```

该命令还将设置设备上的权限以允许写入。 现在再次尝试dd命令。
请注意，dd不会提供任何屏幕上的信息，直到有错误或完成; 当完成时，将显示信息，磁盘将重新安装。 如果你想查看进度，可以使用Ctrl-T; 这将生成SIGINFO，终端的状态参数，并将显示有关进程的信息。

* dd命令完成后，弹出卡：

```
sudo diskutil eject /dev/rdisk3
```

或者，打开磁盘实用程序，然后使用此按钮弹出SD卡。

---
_本文使用eLinux wiki页面RPi_Easy_SD_Card_Setup中的内容，该页面是根据Creative Commons Attribution-ShareAlike 3.0 Unported许可共享的_

---

_参考文献_  [https://www.raspberrypi.org/documentation/installation/installing-images/mac.md](https://www.raspberrypi.org/documentation/installation/installing-images/mac.md)

