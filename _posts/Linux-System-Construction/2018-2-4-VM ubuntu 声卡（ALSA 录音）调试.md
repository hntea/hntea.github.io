
---
layout: post
title: "VM-ubuntu ALSA 录音调试"
modified:
categories: [系统搭建]
excerpt: 该博客记录了虚拟机下使用ALSA录音调试的过程。
tags: [系统搭建]
image: 
  feature: so-simple-sample-image-7.jpg
date: 2018-2-4T15:39:55-04:00
---

vm 在启动时如果没有对声卡进行配置，那么默认为使用主机默认的声卡，也就是说，如果你当前有一个USB的声卡，那么不对虚拟机进行配置是无法使用ALSA驱动调用的。如何在虚拟机中使用USB麦克风进行录音就是这篇文章所记录的。

## 系统版本说明

本次测试使用的是ubuntu14.04及以上的桌面版本，这些版本默认带有ALSA驱动的。但是ALSA相关操作工具不一定自带，所以在使用时将安装对应的工具包。

## 配置步骤

### 1.修改虚拟机配置文件

- 关闭虚拟机，找到虚拟机所在安装目录
- 用文本编辑器打开虚拟机配置 (.vmx) 文件。
- 添加 sound.skipAlsaVersionCheck 属性并将其设置为 TRUE。
例如：sound.skipAlsaVersionCheck = "TRUE"

### 2.为虚拟机配置ALSA声卡

- 选择虚拟机，然后选择虚拟机 > 设置。
- 在硬件选项卡中，选择声卡。
- 选择已连接和启动时连接。
- 选择**指定主机声卡**。

### 3.安装工具包

在终端输入：

    sudo apt-get install alsa-tools alsa-oss flex zlib1g-dev libc-bin libc-dev-bin python-pexpect libasound2-dev


### 4.使用工具检测声卡

在终端输入：

    aplay -L //检测播放设备
    arecord -L //检测录音设备

如果上面上面没有正常显示出声卡设备，那么应该是 alsa-lib 版本安装出错。请前往官网下载较低的版本：
[1.0.15版本](ftp://ftp.alsa-project.org/pub/lib/)

进入目录安装：
    
    ./configure
    make
    sudo make install

单击确定保存所做的更改。

### 5.录音

在终端查看录音设备
    
    arecord -l 

例如：

```
    hntea@ubuntu:~$ arecord -l
**** List of CAPTURE Hardware Devices ****
card 0: AudioPCI [Ensoniq AudioPCI], device 0: ES1371/1 [ES1371 DAC2/ADC]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: U0x46d0x825 [USB Device 0x46d:0x825], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

使用 usb麦克风录音测试：

```
arecord -d 5 -D plughw:1,0 test.wav
```

播放

    aplay ./test.wav
    
    
结束。

## 注意
如果播放时出现特别大的噪声，那么需要到系统设置 system setting 的音频模块进行调节，将输入的增益调节到最小，这样就能避免过度放大导致失真。