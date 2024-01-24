---
layout: post
title:  "【Intel x86 IoT系列】Intel Arduino/Genuino 101 完全开发指南"
tags: intel x86 zh-cn
---

*注：下文中 Intel Arduino/Genuino 101 简称为 Arduino 101*

![](http://icing.fun/img/post/2024/01/20/1.jpg)

# Arduino/Genuino 101 的由来

2015年10月16日[<sup>1</sup>](#jump1)，距离世界上第一台商用可编程计算器 Olivetti Programma 101[<sup>2</sup>](#jump2) 问世过去了整整50年。为了致敬这个伟大的项目，英特尔和 Massimo Banzi[<sup>3</sup>](#jump3)（Arduino项目的联合创始人）在 Maker Faire 宣布推出一款新的开发板，名为Arduino/Genuino 101 —— 一款专为教育用途、创客世界和首次接触编程的人设计的开发板。

# Arduino/Genuino 101 的硬件和软件

Arduino 101是一款学习和开发板，以入门级价格提供英特尔® Curie™ 模块的性能和低功耗以及 Arduino 的简单性。它保留了与 UNO 相同的强大外形和外设列表，并增加了板载低功耗蓝牙®功能和 6 轴加速度计/陀螺仪。

Arduino 101 使用的模块包含两个微型内核，一个 x86 （Quark SE） 和一个 32 位 ARC 架构内核，时钟频率均为 32MHz。英特尔工具链可在两个内核上以最佳方式编译 Arduino 草图，以完成要求最苛刻的任务。英特尔开发的实时操作系统 （RTOS） 和框架是开源的。其中，ARC架构是一种32位的RISC处理器架构，由Synopsys(新思科技)开发。

101 带有 14 个数字输入/输出引脚（其中 4 个可用作 PWM 输出）、6 个模拟输入、一个用于串行通信和草图上传的 USB 连接器、一个电源插孔、一个带 SPI 信号的 ICSP 接头和 I2C 专用引脚。电路板工作电压和 I/O 为 3.3V，但所有引脚均具有 5V 过压保护。

在2016年4月21日，英特尔发布 Arduino 101 固件源代码。包含用于 101 上 Curie 处理器的完整 BSP（板级支持包）。它允许您编译和修改核心操作系统和固件，以管理更新和引导加载程序。固件在 Curie 模块内的 x86 芯片上运行，并使用 *回调* 与 ARC 内核（运行 Arduino 程序）进行通信。x86 内核负责处理低功耗蓝牙® （BLE） 和 USB 通信，从而减轻 ARC 内核的负担。[<sup>4</sup>](#jump4)

# 简单的代码实现

## Blink & Serial Print

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

## IMU

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

# RTC

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

# BLE

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
    BLE.setLocalName("101 LED");
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

# 引用

1. <span id="jump1"><a href="https://en.wikipedia.org/wiki/List_of_Arduino_boards_and_compatible_systems#cite_note-Arduino_101-6">List of Arduino boards and compatible systems - Wikipedia</a></span>
2. <span id="jump2"><a href="https://en.wikipedia.org/wiki/Programma_101">Programma 101 - Wikipedia</a></span>
3. <span id="jump3"><a href="https://massimobanzi.com/about/">Massimo Banzi - massimobanzi.com</a></span>
4. <span id="jump4"><a href="https://blog.arduino.cc/2016/04/21/intel-releases-the-arduino-101-firmware-source-code/">Intel releases the Arduino 101 firmware source code - Arduino</a></span>
