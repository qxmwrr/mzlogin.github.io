---
layout: post
title: Raspberry Pi 3刷入Android Things
categories: [android things]
description: Raspberry Pi 3 B型是世界上最受欢迎的单板电脑的最新版本。 它运行在一颗1.2GHz的四核64位ARM Cortex-A53 CPU上，四个USB 2.0端口，有线和无线网络，HDMI和复合视频输出以及一个40针GPIO连接器，用于物理接口项目。

keywords: [android things, 树莓派, 嵌入式, 刷机]
date: 2017-07-14
author: 化作春泥
---

> 本文首发于[《简书》](http://www.jianshu.com/p/f525d574fff2)，转载请保留链接 : )

Raspberry Pi 3 B型是世界上最受欢迎的单板电脑的最新版本。 它运行在一颗1.2GHz的四核64位ARM Cortex-A53 CPU上，四个USB 2.0端口，有线和无线网络，HDMI和复合视频输出以及一个40针GPIO连接器，用于物理接口项目。

![Raspberry Pi 3 B型](http://upload-images.jianshu.io/upload_images/3806049-3821b1192987c1fe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

有关该板上的[外设I/O信号](https://developer.android.google.cn/things/sdk/pio/index.html)的更多详细信息，请参阅[Raspberry Pi I/O](https://developer.android.google.cn/things/hardware/raspberrypi-io.html)。

## Android Things系统刷入
---
在Android Things系统刷入之前，除了Raspberry Pi之外，还需要以下项目：

* HDMI电缆
* 支持HDMI的显示器
* Micro-USB线
* 以太网电缆
* SD卡读卡器

要想将Android Things系统刷入Raspberry Pi中，需要下载最新版本的系统镜像
官方下载地址：[https://developer.android.google.cn/things/preview/download.html](https://developer.android.google.cn/things/preview/download.html)

下载完系统镜像后按以下步骤操作：

1、将8GB或更大的SD卡插入你的电脑。
2、解压下载完成的系统镜像
3、按照官方Raspberry Pi说明方法将系统镜像写入SD卡：
+ [Linux环境刷入方法](https://www.raspberrypi.org/documentation/installation/installing-images/linux.md)
+ Mac环境刷入方法 [https://qxmwrr.github.io](https://qxmwrr.github.io/2017/07/13/Mac环境SD卡刷入Android-Things系统/)
+ Windows环境刷入方法：
 - 将SD卡插入SD卡读卡器，并检查分配了哪个驱动器盘符。 您可以通过查看Windows资源管理器的左侧列，轻松查看驱动器盘符，如`G :`，如果你有SD卡插槽，您可以使用SD卡插槽，或通过SD适配器插在USB端口上。
 - 从Sourceforge项目页面下载Win32DiskImager实用程序文件；您可以从USB驱动器运行。
 - 从zip文件中提取可执行文件并运行Win32DiskImager实用程序；您可能需要以管理员身份运行。 右键单击该文件，然后选择以管理员身份运行。
 - 选择您先前提取的镜像文件。
 - 在设备框中选择SD卡的驱动器盘符。 小心选择正确的驱动器; 如果你得到错误的，你可以破坏计算机的硬盘上的数据！ 如果您在计算机中使用SD卡插槽，并且无法在Win32DiskImager窗口中看到驱动器，请尝试使用外部SD适配器。
 - 单击写入并等待写入完成。
 - 退出打印机并弹出SD卡。

4、将SD卡插入Raspberry Pi
5、安装以下示意图将Raspberry Pi与外设链接

![Raspberry Pi连接示意图](http://upload-images.jianshu.io/upload_images/3806049-ec63b83f35cd33f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 示意图中`1`通过USB线缆连接电源适配器，`2`通过网线连接到局域网，`3`通过HDMI线缆连接显示器。

6、验证Android Things已经运行在设备上了。然后就能看到Raspberry Pi的信息包括IP地址已经显示到Android Things 系统桌面上了。
7、通过adb tool将Raspberry Pi与电脑连接
```
$ adb connect <ip-address>
connected to <ip-address>:5555
```
> 注意：Raspberry Pi通过多播DNS广播主机名Android.local。 如果主机平台支持MDNS，则还可以使用以下命令连接到Raspberry Pi：
```
$ adb connect Android.local
```

## 连接WIFI
---
刷入系统后，强烈建议将其连接到互联网。 这样您的设备就可以提交崩溃报告和接收更新。

> 注意：设备不需要与计算机位于同一网络上。

使用adb将您的Raspberry Pi连接到WiFi：
1、向Wi-Fi Service 发送包含本地网络的SSID和密码的Intent：
```
$ adb shell am startservice -n com.google.wifisetup/.WifiSetupService -a WifiSetupService.Connect -e ssid <Network_SSID> -e passphrase <Network_Passcode>
```

2、通过logcat验证连接是否成功：
```
$ adb logcat -d | grep Wifi
...
V WifiWatcher: Network state changed to CONNECTED
V WifiWatcher: SSID changed: ...
I WifiConfigurator: Successfully connected to ...
```

3、测试您是否可以访问远程IP地址：
```
$ adb shell ping 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=57 time=6.67 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=57 time=55.5 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=57 time=23.0 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=57 time=245 ms
```

## Raspberry Pi I/O
---
本节介绍在Raspberry Pi 3上运行的应用程序可用的外设I/O接口。
以下引脚示意图说明了此板的断开连接器暴露的可用端口的位置：

![pinout-raspberrypi.png](http://upload-images.jianshu.io/upload_images/3806049-15ed8427bcc7bf14.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

_参考文献_  [https://developer.android.google.cn/things/hardware](https://developer.android.google.cn/things/hardware)

