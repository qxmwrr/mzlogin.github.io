---
layout: post
title: Android Things-硬件基础
categories: [android things]
description: 本文为您提供了您应该熟悉的基本电子概念的概述，以便将外围设备安全连接到设备，并开始编写能够使其运行的应用程序。
keywords: [android things, 树莓派, 嵌入式]
date: 2017-07-12 12:00:00 +0800
author: 化作春泥
---

> 本文首发于[《简书》](http://www.jianshu.com/p/f0df498e91a1)，转载请保留链接 : )

本文为您提供了您应该熟悉的基本电子概念的概述，以便将外围设备安全连接到设备，并开始编写能够使其运行的应用程序。
在继续之前，您应该基本了解电压，电流，电阻和功率之间的关系，因此，如果您需要更多关于这些主题的详细信息，您可能会发现[DC电路理论](http://www.electronics-tutorials.ws/dccircuits/dcp_1.html)和[欧姆定律和电源](http://www.electronics-tutorials.ws/dccircuits/dcp_2.html)有帮助。

## 面包板

面包板是快速原型开发电路的常用工具，无需将元件焊接在一起。 这允许您在开发期间进行布线更改，直到设计稳定。 面包板也可用于测试，通过允许您轻松连接仪器，并探测电路中的各种连接。

![面包板](http://upload-images.jianshu.io/upload_images/3806049-4c5b40afff060580.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

面包板的孔以行和列内部连接在一起，以允许多个组件共享相同的连接点。 外部行垂直于板的其余部分连接，并且在每行中跨越顶部和底部形成单个总线。 这些行通常用于连接电源和地，或在整个电路上需要的其他公共信号。

## 电源

嵌入式设备包含有源电路，这意味着它们需要外部电源才能正常工作。 电源是从外部电源（如墙上适配器，电池或USB端口）传送到电路板上组件的输入电压。 以下信号由电源提供给电路板：

V<sub>IN</sub>
  连接到电路板的外部电源的电压。 许多电路板支持一系列输入电压，并使用内部稳压器为其余组件提供稳定的电源。

V<sub>CC</sub> 或者 V<sub>DD</sub>
  内部稳压为板上的组件供电。 公共电源电压为+ 5V，+ 3.3V和+ 1.8V。

Ground (GND)
  板上0伏的参考点。 所有其他电压相对于GND测量。 在GND以下测量的电压被认为是负的。

## 模拟和数字I/O

外设通过板上提供的各种输入和输出引脚连接。 输入引脚允许您的应用读取和解释当前的电气状态。 输出引脚允许应用控制引脚的电气状态。 外设和板载I/O本质上是模拟或数字的。

#### 模拟

模拟器件产生的电压与其测量的物理条件成比例。 一个好的例子是温度传感器，其可以产生0-5V之间的输出，对应于0-100℃之间的温度。

模拟输入使用[模数转换器](https://en.wikipedia.org/wiki/Analog-to-digital_converter)（ADC）将离散电压电平转换成比例整数值。 用于表示电压电平的整数范围基于ADC的分辨率，以位表示。 例如，10-bit ADC可以将输入电压表示为0-1023（例如，2<sup>10</sup>个离散步长）之间的值。

#### 数字

数字逻辑将电压信号表示为二进制值：

* 高：当电压处于或接近V<sub>CC</sub>时。 通常表示为逻辑“1”。
* 低：当电压处于或接近GND时。 通常表示为逻辑“0”。

数字信号很少精确地为0V或V<sub>CC</sub>。 大多数数字逻辑器件将接近极限的电压范围解释为有效逻辑电平。 下表列出了每个逻辑状态的共用输入电压范围。

Supply Voltage (VCC)	Logic Low (0)	Logic High (1)
5V (TTL)	< 0.8V	> 2.0V
3.3V (CMOS)	< 0.8V	> 2.0V
1.8V (CMOS)	< 0.6V	> 1.2V

| Supply Voltage (V<sub>CC</sub>)    | Logic Low (0)     | Logic High (1)  |
| -------------------------------------------- |:-------------------:|:-----------------:|
| 5V (TTL)                                                     | < 0.8V                  |  > 2.0V              |
| 3.3V (CMOS)                                             | < 0.8V                  |   > 2.0V             |
| 1.8V (CMOS)                                             |< 0.6V                    |    > 1.2V            |

外设通常以几种常见方式使用数字I/O：

* 稳定状态：单一开/关状态映射到稳定的高或低值。
![稳定状态](http://upload-images.jianshu.io/upload_images/3806049-7d7759fddbfeec3e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 脉冲串：具有可变频率和宽度的一系列数字信号脉冲随时间连续传输。
![脉冲串](http://upload-images.jianshu.io/upload_images/3806049-a5fd7c93dfd5db4e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 串行通信：数字1和0系列表示二进制数的各个位。
![串行通信](http://upload-images.jianshu.io/upload_images/3806049-cedf1327ebc2638b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

有关模拟和数字I/O的更多信息，请参阅[传感器和传感器](http://www.electronics-tutorials.ws/io/io_1.html)和[二进制数](http://www.electronics-tutorials.ws/binary/bin_1.html)

## 上拉和下拉

在许多数字接口电路中，电阻器连接在I/O信号引脚和V<sub>CC</sub>或GND之间。 这些分别称为上拉电阻和下拉电阻。 它们保证每个信号具有系统的其余部分可以依赖的稳定的默认状态，而不直接显着影响输入或输出信号。

没有活动连接到任何信号的数字输入是浮动输入。 浮动输入易受[电磁干扰](https://en.wikipedia.org/wiki/Electromagnetic_interference)，这会影响报告给您应用的值，并导致不可预测的读数。 上拉或下拉电阻确保线路驱动到稳定值，即使没有其他连接。

例如，想象一下一个按钮或开关。 开关是一对触点，在闭合时将输入引脚连接到高电平或低电平，但当开路时使输入悬空。 此外，许多数字传感器使用集电极开路（或漏极开路）输出来报告状态变化。 这些输出像一个简单的开关，需要和外部源来驱动开关打开时的输入。

![上拉和下拉](http://upload-images.jianshu.io/upload_images/3806049-8b4bc572efd00439.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 电阻强度

您选择的电阻值以不同的方式影响系统。 低值电阻被认为是“强”，因为更多的电流流动。 强上拉（或下拉）总体上消耗更多功率，但是它们可以比具有更高值的“弱”电阻更快地将信号重置为空闲电平。
> 注：上拉和下拉电阻值通常在1kΩ和10kΩ之间。

例如，当总线空闲时，I<sup>2</sup>C串行总线使用上拉电阻来保持时钟和数据线稳定。 添加到总线上的每个器件都加载这些线路，使得上拉电路难以将线路保持在适当的电平。 随着总线上器件数量的增加，上拉的强度也必须增加以处理增加的负载。

有关应用和计算适当值的更多详细信息，请参见[上拉电阻](http://www.electronics-tutorials.ws/logic/pull-up-resistor.html)。

#### 信号去抖

许多电输入装置，例如开关和继电器，具有机械部件。随着设备的机械运动稳定，电信号可以在多个值之间暂时振荡或“跳动”。在许多情况下，这将在很短的时间内被您的应用程序看作多个输入事件。

要纠正此问题，必须使用硬件或软件对信号进行去抖动。软件去抖包括在初始输入事件和期望输入稳定（通常不超过几百毫秒）之间设置时间延迟。

要使用硬件去抖动输入，请在输入引脚和器件之间添加一个简单的RC电路（因为它包含一个电阻和电容）。当输入装置改变状态时，电容器将以与输入电阻器的尺寸成比例的速率进行充电和放电，从而有效地减缓由输入引脚看到的转变。

有关计算去抖动和将输入信号连接到器件的其他技术的更多信息，请参见[输入接口电路](http://www.electronics-tutorials.ws/io/input-interfacing-circuits.html)。

![信号去抖](http://upload-images.jianshu.io/upload_images/3806049-e5dfed44d066c01c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 保护I/O引脚

每个输出引脚具有有限的从连接到它的电路源极（当高）或吸收（当低）电流时的能力。 比引脚流过更多电流的外设甚至可以暂时处理 - 可能会损坏输出。 为了保护引脚，请在负载上串联一个限流电阻。
> 注意：串联电阻值通常在100Ω和300Ω之间。

为了控制诸如电动机的较高功率的换能器，使用晶体管或类似的电子控制开关缓冲来自输出引脚的负载，并直接从电源为换能器供电。
> 注：I/O引脚的源/汇容量因器件而异。 检查您的硬件文档，以更好地了解您的电路板可以支持什么。

所有I/O引脚均设计为在0V和V<sub>CC</sub>之间的电压范围内安全工作。 将任何引脚连接到高于该组件电源的电压可能会损坏它。 始终验证由传感器和传感器产生的电压电平与其连接的I/ O引脚匹配。 要将可变电源的器件连接在一起，请使用逻辑电平转换器电路。

有关可用于安全连接数字和模拟I / O的电路的更多示例，请参见[输出接口电路](http://www.electronics-tutorials.ws/io/output-interfacing-circuits.html)。

---

_参考文献_  [https://developer.android.google.cn/things/hardware](https://developer.android.google.cn/things/hardware)

