---
layout: default
title:  "【阅板无数】（Intel x86 IoT系列）Intel Arduino/Genuino 101 简单测评"
tags: intel x86 zh-cn
---

*注：下文中 Intel Arduino/Genuino 101 简称为 Arduino 101*

<img src="http://icing.fun/img/post/2024/01/20/1.jpg">
*Arduino 101*

# Arduino/Genuino 101 的由来

2015年10月16日[<sup>1</sup>](#jump1)，距离世界上第一台商用可编程计算器 Olivetti Programma 101[<sup>2</sup>](#jump2) 问世过去了整整50年。为了致敬这个伟大的项目，英特尔和 Massimo Banzi[<sup>3</sup>](#jump3)（Arduino项目的联合创始人）在 Maker Faire 宣布推出一款新的开发板，名为Arduino/Genuino 101 —— 一款专为教育用途、创客世界和首次接触编程的人设计的开发板。

<img src="http://icing.fun/img/post/2024/01/20/2.jpg">
*Olivetti Programma 101*

# Arduino/Genuino 101 的硬件和软件

Arduino 101是一款学习和开发板，以入门级价格提供英特尔® Curie™ 模块的性能和低功耗以及 Arduino 的简单性。它保留了与 UNO 相同的强大外形和外设列表，并增加了板载低功耗蓝牙®（Bluetooth Low Energy，BLE，Nordic 的 nRF512822）功能和 6 轴加速度计/陀螺仪（Bosch 的 BMI160）。

Arduino 101 使用的 Curie™ 模块属于异构双核，包含两个微型内核，一个 x86 （Quark SE，SE 即 Second Edition） 和一个 32 位 ARC 架构内核，时钟频率均为 32MHz。英特尔工具链可在两个内核上以最佳方式编译 Arduino 草图，以完成要求最苛刻的任务。英特尔开发的实时操作系统 （Zephyr） 和框架是开源的。其中，ARC架构是一种32位的RISC处理器架构，由 Synopsys（新思科技）开发。

101 带有 14 个数字输入/输出引脚（其中 4 个可用作 PWM 输出）、6 个模拟输入、一个用于串行通信和草图上传的 USB 连接器、一个电源插孔、一个带 SPI 信号的 ICSP 接头和 I2C 专用引脚。电路板工作电压和 I/O 为 3.3V，但所有引脚均具有 5V 过压保护。

相较于标准 Arduino Uno， Arduino 101 采用的异构双核的 Curie™ 模块比 8位的 Atmel 328p 微控制器更强大，存储空间也更大（Intel 官方的 Curie 规格是 384KB Flash 与 80KB SRAM ，但相关报导写 196KB Flash 与 24KB RAM，Arduino 官网写 384KB Flash 与 80KB SRAM，但 SRAM 部分注明只有 24KB 可供应用程序 Sketch 使用。因此，推估之所以写 196KB Flash，应该也是系统占据一部分，真正可供运用的是 196KB。除此之外电路板上似乎又增设2MB Flash 可供 Curie 使用）

在2016年4月21日，英特尔发布 Arduino 101 固件源代码。包含用于 101 上 Curie 处理器的完整 BSP（板级支持包）。它允许您编译和修改核心操作系统和固件，以管理更新和引导加载程序。固件在 Curie 模块内的 x86 芯片上运行，并使用 *回调* 与 ARC 内核（运行 Arduino 程序）进行通信。x86 内核负责处理低功耗蓝牙® （BLE） 和 USB 通信，从而减轻 ARC 内核的负担。[<sup>4</sup>](#jump4)

## 技术规格[<sup>5</sup>](#jump5)

| Microcontroller             | Intel Curie                                      |
| --------------------------- | ------------------------------------------------ |
| Operating Voltage           | 3.3V (5V tolerant I/O)                           |
| Input Voltage (recommended) | 7-12V                                            |
| Input Voltage (limit)       | 7-17V                                            |
| Digital I/O Pins            | 14 (of which 4 provide PWM output)               |
| PWM Digital I/O Pins        | 4                                                |
| Analog Input Pins           | 6                                                |
| DC Current per I/O Pin      | 20 mA                                            |
| Flash Memory                | 196 kB                                           |
| SRAM                        | 24 kB                                            |
| Clock Speed                 | 32MHz                                            |
| LED_BUILTIN                 | 13                                               |
| Features                    | Bluetooth® Low Energy, 6-axis accelerometer/gyro |
| Length                      | 68.6 mm                                          |
| Width                       | 53.4 mm                                          |
| Weight                      | 34 gr.                                           |

# Arduino 代码实现

## Blink

```c
#include <Arduino.h>

void setup() 
{
    Serial.begin(9600);
    while (!Serial)
    {
        delay(10);
        // wait for serial port to connect. Needed for native USB port only
    }
    pinMode(LED_BUILTIN, OUTPUT);
}

void loop() 
{
    digitalWrite(LED_BUILTIN, HIGH);
    Serial.println("LED ON!");
    delay(1000);
    digitalWrite(LED_BUILTIN, LOW);
    Serial.println("LED OFF!");
    delay(1000);
}
```

## 读取板载 IMU

```c
#include <Arduino.h>
#include "CurieIMU.h"

void setup() 
{
    Serial.begin(9600);
    while (!Serial)
    {
        delay(10);
    }
    // Start the acceleromter
    CurieIMU.begin();

    // Set the accelerometer range to 2G
    CurieIMU.setAccelerometerRange(2);
}

void loop() 
{
    // read accelerometer:
    int x = CurieIMU.readAccelerometer(X_AXIS);
    int y = CurieIMU.readAccelerometer(Y_AXIS);
    int z = CurieIMU.readAccelerometer(Z_AXIS);

    Serial.print("x: ");
    Serial.print(x);
    Serial.print(" y: ");
    Serial.print(y);
    Serial.print(" z: ");
    Serial.print(z);
    Serial.println("");
}
```

# 读取板载 RTC

```c
#include <Arduino.h>
#include <CurieTime.h>

void setup()
{
    Serial.begin(9600);
    while (!Serial)
    {
        delay(10);
    }
    setTime(1, 23, 24, 25, 1, 2024);
}

void loop()
{
    //create a character array of 16 characters for the time
    char clockTime[16];
    //use sprintf to create a time string of the hour, minte and seconds
    sprintf(clockTime, "%2d:%2d:%2d", hour(), minute(), second());

    //create a character array of 15 characters for the date
    char dateTime[16];
    //use sprintf to create a date string from month, day and year
    sprintf(dateTime, "%2d/%2d/%4d", month(), day(), year());

    //print the time and date to the serial monitor
    Serial.print(clockTime);
    Serial.println(dateTime);
    delay(1000);
}
```

# 使用板载 BLE 控制 LED

*需要配合Nordic nRF Connect使用，可以在 Google Play Store 或 App Store 下载*

```c
#include <Arduino.h>
#include <CurieBLE.h>

BLEService ledService("19B10000-E8F2-537E-4F6C-D104768A1214"); // BLE LED Service

// BLE LED Switch Characteristic - custom 128-bit UUID, read and writable by central
BLEUnsignedCharCharacteristic switchCharacteristic("19B10001-E8F2-537E-4F6C-D104768A1214", BLERead | BLEWrite);

const int ledPin = 13; // pin to use for the LED

void setup() 
{
    Serial.begin(9600);

    // set LED pin to output mode
    pinMode(ledPin, OUTPUT);

    // begin initialization
    BLE.begin();

    // set advertised local name and service UUID:
    BLE.setLocalName("Arduino 101");
    BLE.setAdvertisedService(ledService);

    // add the characteristic to the service
    ledService.addCharacteristic(switchCharacteristic);

    // add service
    BLE.addService(ledService);

    // set the initial value for the characeristic:
    switchCharacteristic.setValue(0);

    // start advertising
    BLE.advertise();

    Serial.println("BLE LED Peripheral");
}

void loop() 
{
    // listen for BLE peripherals to connect:
    BLEDevice central = BLE.central();

    // if a central is connected to peripheral:
    if (central) 
    {
        Serial.print("Connected to central: ");
        // print the central's MAC address:
        Serial.println(central.address());

        // while the central is still connected to peripheral:
        while (central.connected()) 
        {
            // if the remote device wrote to the characteristic,
            // use the value to control the LED:
            if (switchCharacteristic.written()) 
            {
                if (switchCharacteristic.value()) 
                {   // any value other than 0
                    Serial.println("LED on");
                    digitalWrite(ledPin, HIGH);         // will turn the LED on
                } 
                else 
                {                              // a 0 value
                    Serial.println(F("LED off"));
                    digitalWrite(ledPin, LOW);          // will turn the LED off
                }
            }
        }

        // when the central disconnects, print it out:
        Serial.print(F("Disconnected from central: "));
        Serial.println(central.address());
    }
}
```

<img src="http://icing.fun/img/post/2024/01/20/3.gif" width="50%">

# 后记

Arduino 101 于 2016年 Q1 季度发布。仅仅过去一年有余，在英特尔宣布 Galileo，Edison 和 Joule 模块停产一个月后，也草草停产 Curie 模块（07/17/2017）[<sup>6</sup>](#jump6)，只能说是非常可惜，英特尔宏图壮志准备在 IoT 领域大展拳脚，但是却因为种种原因，最终只能黯然收场。也导致诸多使用 Curie 模块的企业被迫更换产品线，比如[小米智能跑鞋](https://www.cnx-software.com/2017/03/29/100-xiaomi-90-minutes-ultra-smart-running-shoes-are-equipped-with-intel-curie-module/)。

<img src="http://icing.fun/img/post/2024/01/20/Intel-Curie-Arduino-101-Discontinued.webp">
*图片源自[Intel Curie Module, Arduino 101 Board Are Being Discontinued (Too) - CNX Software](https://www.cnx-software.com/2017/07/26/intel-curie-module-arduino-101-boards-are-being-discontinued-too/)*

《Arduino程序设计基础》 的作者 陈吕洲 先生对 Arduino 101 抱有极高的评价：Genuino 101是一个极具特色的Arduino开发板，它基于Intel Curie模组，不仅有着和Arduino UNO一样特性和外设，还集成了低功耗蓝牙（BLE）和六轴姿态传感器（IMU）功能，借助intel Curie模组上模式匹配引擎，甚至可以进行机器学习操作。因此使用Genuino 101，可以完成一些传统单片机或者Arduino难以胜任的工作，制作更为惊艳的作品。为此他本人还专门为 Arduino 101 著书 —— 《Arduino 101 开发入门》[<sup>7</sup>](#jump7)。

不过 Arduino 101 也有三点比标准 Arduino Uno 差[<sup>8</sup>](#jump8)：

- 只有4组 PWM 脉宽调变输出，Uno 有6组；
- 没有任何的 EEPROM 存储，Uno 至少还有1KB可以使用；
- 单一 I/O 的电流驱动能力最高仅4mA，Uno 可到20mA。

所以 Curie 只是引脚排列与 Arduino 相仿，不能完全保证原有设计电路或者 Arduino Shield可完全相容（兼容）沿用、续用。

如果再与 MediaTek 的 LinkIt ONE 小比一下，LinkIt One也有效能更佳的处理器核心与更多容量的存储，且依然提供EEPROM 可用，但 PWM 方面则只有2组。当然，LinkIt ONE 强在无线通讯（日后会测评一下），如 GPRS、Wi-Fi、GPS等，Curie 略强在惯性感测。

英特尔似乎在 IoT 领域并没有取得太大的成就，但是英特尔的 x86 芯片却是世界上最流行的 CPU 架构，这也是英特尔的核心竞争力，所以英特尔在 IoT 领域的失败并不会影响到英特尔的核心业务，但是英特尔的 IoT 产品却是非常有趣的，比如 Edison、Galileo、Curie、Arduino 101 等等，这些产品都是英特尔的 IoT 产品，但是英特尔并没有将这些产品做成一个系列，而是分散在各个不同的系列中，这也是英特尔在 IoT 领域失败的一个原因。

笔者手上的 Genuino 101 是 2021年在 SeeedStudio 任职时，从 FAE 仓库里找到的，当时已经停产了，但是还是有一些库存，所以就拿了一块回来，但是一直没有时间去折腾，直到最近才拿出来玩一玩，但是发现 Arduino 101 的资料实在是太少了，所以就写了这篇文章，谨此纪念一家伟大的公司一个伟大的产品。

# 引用

1. <span id="jump1"><a href="https://en.wikipedia.org/wiki/List_of_Arduino_boards_and_compatible_systems#cite_note-Arduino_101-6">List of Arduino boards and compatible systems - Wikipedia</a></span>
2. <span id="jump2"><a href="https://en.wikipedia.org/wiki/Programma_101">Programma 101 - Wikipedia</a></span>
3. <span id="jump3"><a href="https://massimobanzi.com/about/">Massimo Banzi - massimobanzi.com</a></span>
4. <span id="jump4"><a href="https://blog.arduino.cc/2016/04/21/intel-releases-the-arduino-101-firmware-source-code/">Intel releases the Arduino 101 firmware source code - Arduino</a></span>
5. <span id="jump5"><a href="https://www.arduino.cc/en/Main/ArduinoBoard101">Arduino 101 - Arduino</a></span>
6. <span id="jump6"><a href="https://ark.intel.com/content/www/cn/zh/ark/products/92347/arduino-101.html">Arduino 101 - Intel</a></span>
7. <span id="jump7"><a href="https://clz.me/101-book/">《Arduino 101 开发入门》 - XX到此一游</a></span>
8. <span id="jump8"><a href="https://makerpro.cc/2015/10/arduino-101-review/">Arduino 101擁抱Intel Curie核心 優缺點比一比 - 陸向陽 MakerPro</a></span>
