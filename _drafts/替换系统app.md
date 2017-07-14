---
layout: post
title: 替换系统apk
categories: [android]
description: 已root手机如何替换系统apk，前提电脑装有adb环境
keywords: android, system, apk
---

# 替换系统apk

## 步骤

#### 1. 认识`/data/local/tmp`目录

该目录是一个临时目录，可以通过`adb pull` 命令 将电脑的文件传输到这个目录。常用示例

```
adb pull ~/SysteUI-not-modify.apk /data/local/tmp/SystemUI.apk
```

这条命令的意思是将电脑用户目录的`SysteUI-not-modify.apk`文件拷贝到了`/data/local/tmp`目录，并改名为`SystemUI.apk`
	
#### 2. `adb push`命令

从电脑将文件复制到手机
	
	通用是上面的命令，就是将电脑的文件`~/SysteUI-not-modify.apk`复制到手机的`/data/local/tmp/SystemUI.apk`这里。
	
#### 3. `mount`命令

该命令执行后，查找`system`关键字。

```shell
rootfs / rootfs ro,relatime 0 0
tmpfs /dev tmpfs rw,seclabel,nosuid,relatime,mode=755 0 0
devpts /dev/pts devpts rw,seclabel,relatime,mode=600 0 0
proc /proc proc rw,relatime 0 0
...
adb /dev/usb-ffs/adb functionfs rw,relatime 0 0
/dev/block/platform/ff0f0000.rksdmmc/by-name/system /system ext4 ro,seclabel,noatime,nodiratime,noauto_da_alloc,data=ordered 0 0
/dev/block/platform/ff0f0000.rksdmmc/by-name/cache /cache ext4 rw,seclabel,nosuid,nodev,noatime,nodiratime,discard,noauto_da_alloc,data=ordered 0 0
/dev/block/platform/ff0f0000.rksdmmc/by-name/metadata /metadata ext4 rw,seclabel,nosuid,nodev,noatime,nodiratime,discard,noauto_da_alloc 0 0
/dev/block/platform/ff0f0000.rksdmmc/by-name/userdata /data ext4 rw,seclabel,nosuid,nodev,noatime,nodiratime,discard,noauto_da_alloc,data=ordered 0 0
...
```
	
执行完`mount`命令后，从结果中找到有关`system`分区的那行，也就是
	
```shell
/dev/block/platform/ff0f0000.rksdmmc/by-name/system /system ext4 ro,seclabel,noatime,nodiratime,noauto_da_alloc,data=ordered 0 0
```
	
然后将system分区挂载成读写模式
	
```shell
mount -o remount,rw /dev/block/platform/ff0f0000.rksdmmc/by-name/system /system
```
	
为了保险可以再执行下`mount`命令查看`system`分区是否从`ro`变成了`rw`。
	
#### 4. 最后一步，替换文件

执行`cat`命令，拷贝文件

```shell
cat /data/local/tmp/SystemUI.apk /system/priv-app/SystemUI.apk
``` 

#### 5. 重启电脑

查看效果




