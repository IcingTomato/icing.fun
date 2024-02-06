---
layout: post
title:  "【nRF5 系列】BBC Micro:bit 简单测评"
tags: nrf microbit zh-cn
---

*注：下文中 BBC Micro:bit 简称为 Micro:bit*

# BBC Micro:bit 的前世今生

BBC Micro:bit 是一款由英国广播公司（BBC）发起的教育计划，旨在帮助学生学习编程。Micro:bit 由 ARM、Nordic Semiconductor、Microsoft、Samsung 和 element14 等公司合作开发，于2016年3月发布。Micro:bit 采用了Nordic Semiconductor的nRF51822 SoC，配备了蓝牙、加速度计、磁力计、温度传感器、5x5 LED 点阵等功能。Micro:bit 的硬件设计开源，软件开发环境支持多种语言，包括Python、JavaScript、C++等。

笔者手上有当年在 SeeedStudio 仓库里面薅出来的<s>一块</s>好几块 Micro:bit，请大家跟随我的脚步，一起来体验一下这款教育板的魅力。

# Micro:bit 的硬件规格

```c++
// GPIO引脚定义
const int rowPins[3] = {26, 27, 28}; // ROW1, ROW2, ROW3
const int colPins[9] = {3, 4, 10, 23, 24, 25, 9, 7, 6}; // COL1, COL2, ..., COL9

void setup() {
  // 初始化所有行和列的引脚为输出
  for (int i = 0; i < 3; i++) {
    pinMode(rowPins[i], OUTPUT);
  }
  for (int i = 0; i < 9; i++) {
    pinMode(colPins[i], OUTPUT);
  }

  // 初始状态，所有ROW引脚设为LOW, 所有COL引脚设为HIGH
  for (int i = 0; i < 3; i++) {
    digitalWrite(rowPins[i], LOW);
  }
  for (int i = 0; i < 9; i++) {
    digitalWrite(colPins[i], HIGH);
  }
}

void controlLED(int row, int col, bool turnOn) {
  // 确保输入的行和列是有效的
  if (row < 1 || row > 3 || col < 1 || col > 9) return;

  // 计算行和列的索引
  int rowIndex = row - 1;
  int colIndex = col - 1;

  // 根据turnOn变量的值点亮或熄灭LED
  // 当turnOn为true时，行设置为HIGH，列设置为LOW，LED亮
  // 当turnOn为false时，行设置为LOW，保持LED熄灭状态
  digitalWrite(rowPins[rowIndex], turnOn ? HIGH : LOW);
  digitalWrite(colPins[colIndex], LOW);
}

void loop() {
  // 点亮LED示例
  controlLED(1, 1, true); // 点亮(1,1)的LED
  delay(1000);            // 保持亮1秒钟

  // 熄灭所有LED
  for (int i = 0; i < 3; i++) {
    digitalWrite(rowPins[i], LOW);
  }
  delay(1000);             // 保持灭1秒钟
}
```

```c++
// GPIO引脚定义
const int rowPins[3] = {26, 27, 28}; // ROW1, ROW2, ROW3
const int colPins[9] = {3, 4, 10, 23, 24, 25, 9, 7, 6}; // COL1, COL2, ..., COL9

void setup() {
  // 初始化所有行和列的引脚为输出
  for (int i = 0; i < 3; i++) {
    pinMode(rowPins[i], OUTPUT);
    digitalWrite(rowPins[i], LOW); // 初始化为LOW，关闭所有行
  }
  for (int i = 0; i < 9; i++) {
    pinMode(colPins[i], OUTPUT);
    digitalWrite(colPins[i], HIGH); // 初始化为HIGH，关闭所有列
  }
}

void turnOnLED(int row, int col) {
  // 点亮特定的LED
  digitalWrite(rowPins[row], HIGH); // 点亮行
  digitalWrite(colPins[col], LOW);  // 点亮列
}

void turnOffLEDs() {
  // 熄灭所有LED
  for (int i = 0; i < 3; i++) {
    digitalWrite(rowPins[i], LOW); // 关闭行
  }
  for (int i = 0; i < 9; i++) {
    digitalWrite(colPins[i], HIGH); // 关闭列
  }
}

void loop() {
  for (int row = 0; row < 3; row++) {
    for (int col = 0; col < 9; col++) {
      turnOnLED(row, col);     // 点亮当前LED
      delay(10);              // 保持亮100毫秒
      turnOffLEDs();           // 关闭所有LED
    }
  }
}
```