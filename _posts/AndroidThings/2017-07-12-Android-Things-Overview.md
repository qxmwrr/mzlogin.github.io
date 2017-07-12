---
layout: post
title: Android Things 概览
categories: [android things]
description: Android Things与Android使用相同的开发工具，同类最佳的Android框架和Google API，从而使开发连接的嵌入式设备变得轻松。
keywords: [android things, 树莓派, 嵌入式]
date: 2017-07-12 09:00:00
author: 化作春泥
---

> 本文首发于[《简书》](http://www.jianshu.com/p/d8b0a785a4cb)，转载请保留链接 : )

  Android Things与Android使用相同的开发工具，同类最佳的Android框架和Google API，从而使开发连接的嵌入式设备变得轻松。

![框架概览](http://upload-images.jianshu.io/upload_images/3806049-53c820d998844540.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  嵌入式设备的应用程序使开发人员更接近硬件外设和驱动程序，而不是手机和平板电脑。 此外，嵌入式设备通常向用户提供单一的应用体验。 本文档介绍了核心Android开发和Android Things之间的主要增加，省略和差异。

  Android Things使用由物理支持库提供的附加API扩展了核心Android框架。 这些API允许应用程序与移动设备上没有的新类型的硬件集成。

  Android Things平台也简化了单个应用程序的使用。 不存在系统应用程序，您的应用程序会在启动时自动启动，让使用者沉浸在应用程序体验中。

# Things 支持库

## 外设I/O API

  外设I/O API使您的应用程序可以使用行业标准协议和接口与传感器和执行器进行通信。 支持以下接口：GPIO，PWM，I2C，SPI，UART。

  有关如何使用API的更多信息，请参阅[外设I/O API指南 http://www.jianshu.com/p/d33c4b832bf0](http://www.jianshu.com/p/d33c4b832bf0)。

## 用户驱动程序API

  用户驱动程序扩展了现有的Android框架服务，并允许应用程序将硬件事件插入其他应用程序可以使用标准Android API访问的框架中。

  有关如何使用API的更多信息，请参阅[用户驱动程序API指南](https://developer.android.com/things/sdk/drivers/index.html)。

# 行为更改


## 核心应用程序包

  Android Things不包括标准的系统应用程序和 Content Providers。所以避免在应用中使用相关Intents以及以下Content Providers API：

CalendarContract
ContactsContract
DocumentsContract
DownloadManager
MediaStore
Settings
Telephony
UserDictionary
VoicemailContract

## 显示模块是可选的

  Android Things支持使用可用于传统Android应用程序的相同UI工具包的图形用户界面。 在图形模式下，应用程序窗口占据显示器的全部空间。 Android Things不包括系统状态栏或导航按钮，应用程序完全控制可视区域。

  然而，Android Things不需要显示时。 在没有图形显示的设备上，Activitys仍然是Android Things应用程序的主要组件。 这是因为框架将所有输入事件传递到具有焦点的前台Activity。 您的应用程式无法通过任何其他应用程序组件（例如Service）接收key事件或motion事件。

## Home activity 支持

  Android Things 希望一个应用程序将其清单中的“Home activity”公开为系统在启动时自动启动的主要入口点。 此活动必须包含一个包含CATEGORY_DEFAULT和IOT_LAUNCHER的Intent Filter。

  为了便于开发，这个activity应包含CATEGORY_LAUNCHER Intent Filter，以便Android Studio可以在部署或调试时将其作为默认活动启动。

```android
<application
    android:label="@string/app_name">
    <activity android:name=".HomeActivity">
        <!-- Launch activity as default from Android Studio -->
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
            <category android:name="android.intent.category.LAUNCHER"/>
        </intent-filter>

        <!-- Launch activity automatically on boot -->
        <intent-filter>
            <action android:name="android.intent.action.MAIN"/>
            <category android:name="android.intent.category.IOT_LAUNCHER"/>
            <category android:name="android.intent.category.DEFAULT"/>
        </intent-filter>
    </activity>
</application>
```

## Google services支持

  Android Things支持Android的Google API的一个子集。 作为一般规则，需要用户输入或身份验证凭据的API不适用于应用。 下表列出了Android Things的API支持：

|Supported APIs|Unavailable APIs|
|-------------------|--------------------|
|Cast|AdMob|
|Drive|Android Pay|
|Firebase Analytics|Firebase App Indexing|
|Firebase Cloud Messaging (FCM)|Firebase Authentication|
|Firebase Crash Reporting|Firebase Dynamic Links|
|Firebase Realtime Database|Firebase Invites|
|Firebase Remote Config|Firebase Notifications|
|Firebase Storage| Maps |
|Fit|Play Games|
|Instance ID|Search|
|Location|Sign-In|
|Nearby| |
|Places| |
|Mobile Vision| ~|

## 权限

  不支持在运行时请求权限，因为嵌入式设备不能保证有一个UI来接受运行时对话框。 在应用清单文件中声明所需的权限。 在您的应用的清单中声明的所有正常和危险权限在安装时都会被授予。

## 通知

  由于Android Things中没有系统范围的状态栏和窗口阴影，因此不支持通知。 避免在应用程序中调用NotificationManager API。

---

_参考文献_ [https://developer.android.google.cn/things/sdk](https://developer.android.google.cn/things/sdk)

