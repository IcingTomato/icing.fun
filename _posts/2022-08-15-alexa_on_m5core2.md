---
layout: default
title:  "【嵌入式实践】Amazon Alexa on M5Core2"
tags: esp32 embedded en-us
---

如何用 ESP32 制作一个语音助手

Amazon Alexa for M5Stack M5Core2(ESP32)

## 0. Briefing of Alexa

**Amazon Alexa**, also known simply as **Alexa**, is a virtual assistant technology largely based on a Polish speech synthesiser named Ivona, bought by Amazon in 2013. It was first used in the Amazon Echo smart speaker and the Echo Dot, Echo Studio and Amazon Tap speakers developed by Amazon Lab126. It is capable of voice interaction, music playback, making to-do lists, setting alarms, streaming podcasts, playing audiobooks, and providing weather, traffic, sports, and other real-time information, such as news. Alexa can also control several smart devices using itself as a home automation system. Users are able to extend the Alexa capabilities by installing "skills" (additional functionality developed by third-party vendors, in other settings more commonly called apps) such as weather programs and audio features. It uses automatic speech recognition, natural language processing, and other forms of weak AI to perform these tasks.

Most devices with Alexa allow users to activate the device using a wake-word (such as Alexa or Amazon); other devices (such as the Amazon mobile app on iOS or Android and Amazon Dash Wand) require the user to click a button to activate Alexa's listening mode, although, some phones also allow a user to say a command, such as "Alexa" or "Alexa wake".

## 1. Introduction

Source Code is [here](https://github.com/IcingTomato/AlexaM5Core2)

<center>
    <img src="http://icing.fun/img/post/2022/08/15/amazon-alexa-logo-300x93.png" alt="Alexa Logo" title="Alexa Logo" width="50%" />
</center>

[Amazon Alexa](https://developer.amazon.com/en-US/alexa) on [M5Core2](https://docs.m5stack.com/en/core/core2) and [M5Core2 for AWS](https://docs.m5stack.com/en/core/core2_for_aws)

You can use M5Core2Alexa like using a real Amazon Echo (sometimes, not always). You can ask M5Alexa questions like "Alexa, what time is sunrise?" or ask her into Japanese 「アレクサ、今何時ですか？」, or maybe you can control your lights and fans (I don't have any Alexa-supported IoT devices). 

## 2. Development Setup

This sections talks about setting up your development host, fetching the git repositories, and instructions for build and flash.

### 2.1 Host Setup

You should install drivers and support packages for your development host. Windows, Linux and Mac OS-X, are supported development hosts. Please see Get Started for the host setup instructions.

### 2.2 Getting the Repositories

```shell
git clone --recursive https://github.com/espressif/esp-idf.git

cd esp-idf; git checkout release/v4.2; git submodule init; git submodule update --init --recursive;

./install.sh

cd ..

git clone https://github.com/IcingTomato/AlexaM5Core2.git
```

### 2.3 Building the Firmware

```shell
cd AlexaM5Core2/examples/amazon_alexa/ 

export ESPPORT=/dev/ttyUSB0 (or /dev/ttycu.SLAB_USBtoUART macOS or COMxx on MinGW)

export IDF_PATH=/path/to/esp-idf

. $IDF_PATH/export.sh
```

Set audio_board path for M5Core2 and AWS_EDUKIT_PATH:

```shell
export AUDIO_BOARD_PATH=/path/to/AlexaM5Core2/components/audio_hal/audio_board/audio_board_m5_core2_aws
export AWS_EDUKIT_PATH=/path/to/AlexaM5Core2/components/core2forAWS
```

Menuconfig is avaliable, also you can change some components:

```shell
idf.py menuconfig
```

## 3. Configuration Steps

Here are the steps to configure the M5Core2:

*   On first boot-up, the M5Core2 is in configuration mode. This is indicated by Orange LED pattern. Please ensure that the LED pattern is seen as described above, before you proceed.
*   Launch the phone app.
*   Select the option *Add New Device*.

<center>
    <img src="http://icing.fun/img/post/2022/08/15/esp_alexa_app_home.png" alt="App Home" title="App Home" width="50%" />
</center>

*   A list of devices that are in configuration mode is displayed. Note that the devices are discoverable over BLE (Bluetooth Low Energy). Please ensure that the phone app has the appropriate permissions to access Bluetooth (on Android the *Location* permission is also required for enabling Bluetooth).

<center>
    <img src="http://icing.fun/img/post/2022/08/15/esp_alexa_app_discover_devices.png" alt="App Discover Devices" title="App Discover Devices" width="50%" />
</center>

*   Now you can sign-in to your Amazon Alexa account. If you have Amazon Shopping app installed on the same phone, app will automatically sign-in with the account the shopping app is signed in to. Otherwise it will open a login page on the phone's default browser. (It is recommended to install the Amazon Shopping app on your phone to avoid any other browser related errors.)

<center>
    <img src="http://icing.fun/img/post/2022/08/15/esp_alexa_app_sign_in.png" alt="App Sign-in" title="App Sign-in" width="50%" />
</center>

*   You can now select the Wi-Fi network that the M5Core2 should connect with, and enter the credentials for this Wi-Fi network.

<center>
    <img src="http://icing.fun/img/post/2022/08/15/esp_alexa_app_wifi_scan_list.png" alt="App Scna List" title="App Scan List" width="50%" />
    <img src="http://icing.fun/img/post/2022/08/15/esp_alexa_app_wifi_password.png" alt="App Wi-Fi Password" title="App Wi-Fi Password" width="50%" />
</center>

*   On successful Wi-Fi connection, you will see a list of few of the voice queries that you can try with the M5Core2.

<center>
    <img src="http://icing.fun/img/post/2022/08/15/esp_alexa_app_things_to_try.png" alt="App Things To Try" title="App Things To Try" width="50%" />
</center>

*   You are now fully setup. You can now say "Alexa" followed by the query you wish to ask.

## 4. Troubleshooting

### 4.1 Music and radio playback (Audible or Podcast) are not supported on this device.

According to Espressif Official, music and radio playback  (Audible or Podcast) services are not supported on Non-Amazon-Official products. `Unfortunately, Amazon music functionality needs whitelisted product. Only the commercial products are whitelisted by Amazon. Let us know if you are going to commercialise it.`

### 4.2 `Alarm Dismiss` doesn't take effect.

When the alarm on, you cannot say "Alexa, stop alarm" to dismiss the alarm. I dno't know why... So, if you want to stop the alarm, you can touch the middle "button" on M5Core2. The middle "button" can also wake up Alexa.

