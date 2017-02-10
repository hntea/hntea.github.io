---
layout: post
title: "ROS+Kinect2 音频采集"
modified:
categories: [ROS实战]
excerpt: 该博客详细介绍在ROS系统下使如何用ALSA音频库结合 Kinect2.0 的麦克风阵列进行音频信号采集
tags: [Linux,ALSA,ROS,Kinect2]
image: 
  feature: so-simple-sample-image-7.jpg
date: 2017-1-21T15:39:55-04:00
---

ROS是基于Linux的二次操作系统，就好比如说是安装在Linux系统下的大型软件，既然它是Linux的一款大型软件，那么Linux下的应用开发套路必然也适用于ROS系统。因此，适合 ROS 的音频开发库也就容易搜索，只需要适用 Linux即可。其实对于嵌入式系统来说，若对应的类库支持该嵌入式平台，那么在PC上开发的上层应用通过相应的交叉编译也能在嵌入式平台上正常运行。不过正确的开发方式是边开发边测试。本次所在的系统环境为

- ubuntu14.04
- ROS Indigo

下面简要列举较为常用的音频开发基础类库

##  Linux下的音频开发库

- [ALSA](http://www.alsa-project.org/alsa-doc/alsa-lib/)

- [PortAudio](http://portaudio.com/docs/v19-doxydocs/tutorial_start.html)

- [GStreamer](https://gstreamer.freedesktop.org/)

在使用音频开发库之前，简单的了解相关概念有助于我们的进一步开发。

##  音频相关概念

音频采集首先要了解的几个概念：

- 样本精度：样本是记录音频数据最基本的单位,常见有8位和16

- 采样频率：每秒采样次数，可类比成每次中断读取一次数据

- 通道数：分单声道(1)、立体声(2)、多通道

- 桢：桢记录了一个声音单元,其长度为样本长度与通道数的乘积

- 交错模式(interleaved):是一种音频数据的记录方式,在交错模式下,数据以连续桢的形式存放,即首先记录完桢1的左声道样本和右声道样本(假设为立体声格式),再开始桢2的记录。

- 非交错模式：首先记录的是一个周期内所有桢的左声道样本,再记录右声道样本,数据是以连续通道的方式存储。**假如需要对多通道的音频信号分离各自通道的音频信号，使用这种数据存储方式比较合适。**

注意：本文主要针对**Kinect2麦克风阵列**，由于该麦克风阵列是四个麦克风，在采集上可使用单通道或多通道的方式，鉴于接下去任务的衔接，该文会使用**4通道**来进行介绍，而对于信号的存储，将会采用**非交错模式**，这样有利于对每一通道进行单独分析。

---

##  工作环境测试

概念了解清楚了，接下来就找找工具来操练一下，现在准备以下几样东西：

### ## 硬件配备

- USB麦克风(也可以是带有麦克风的USB摄像头,Kinect2.0)


### ## 软件配备

ALSA是 Linux 的先进音频库来说，适用于绝大多数音频设备，所以在测试硬件是否能正常工作可以先安装该库，只需在终端简单输入一下命令安装即可（树莓派3同样适用）：

```
sudo apt-get install alsa-tools alsa-oss flex zlib1g-dev libc-bin libc-dev-bin python-pexpect libasound2-dev
```

简要说明一下重要的库：

- alsa-tools: 该库提供了对音频操作的相关指令
- libasound2-dev：提供alsa应用编程API，如果使用c/c++编程会用到该库的一些函数。


### ## 测试

首先查看系统上的声卡，在终端上执行

```
hntea@HnteaPC:~/gitSoft$ arecord -l
**** CAPTURE 硬體裝置清單 ****
card 1: PCH [HDA Intel PCH], device 0: ALC887-VD Analog [ALC887-VD Analog]
  子设备: 1/1
  子设备 #0: subdevice #0
card 1: PCH [HDA Intel PCH], device 2: ALC887-VD Alt Analog [ALC887-VD Alt Analog]
  子设备: 1/1
  子设备 #0: subdevice #0
card 2: U0x46d0x825 [USB Device 0x46d:0x825], device 0: USB Audio [USB Audio]
  子设备: 1/1
  子设备 #0: subdevice #0
card 3: Sensor [Xbox NUI Sensor], device 0: USB Audio [USB Audio]
  子设备: 1/1
  子设备 #0: subdevice #0
```

或者

```
hntea@HnteaPC:~/gitSoft$ cat /proc/asound/pcm 
00-03: HDMI 0 : HDMI 0 : playback 1
01-00: ALC887-VD Analog : ALC887-VD Analog : playback 1 : capture 1
01-02: ALC887-VD Alt Analog : ALC887-VD Alt Analog : capture 1
02-00: USB Audio : USB Audio : capture 1
03-00: USB Audio : USB Audio : capture 1
```

通过上面的指令,就可以具体指定使用哪个录音设备来测试了，假如要使用[Xbox NUI Sensor],那么只需将下面指令的设备名改成：plughw:3,0 

在终端执行以下命令录制一段音频

```
arecord -d 10 -D plughw:1,0 test.wav
```

arecord 参数说明：

- -d: 录制时间（秒） 
- **-D: 指明设备名（plughw:i,j）:其中i是卡号,j是这块声卡上的设备号**
- plughw:,表示一个插件,ALSA库使用逻辑设备名而不是设备文件,详细介绍如下：
	- 默认设备名： default
	- 硬件名字使用**hw:i,j**格式：i是卡号,j是这块声卡上的设备号


更多详细内容请在终端输入

```
arecord -help
```
录制完之后肯定播放一下啦，你可以使用简单的应用工具打开音频文件，当然也可以用命令方式，当先安装如下工具

```
sudo apt-get install sox
```
之后插上耳机执行如下：

```
play xxx.wav 
```
不出意外，你应该能成功录音并且播放，到现在为止，硬件方面的测试已经成功了！基础搭好了，现在该如何做呢？

---


## 使用ROS下的通用音频包


ROS为开发者提供了一套分布式的通信框架，其主要目的是避免重复造轮子。既然如此，就本着拿来主义的精神，先到收集有没有相应的功能包。<br>

官网介绍，该功能包具有音频捕获，音频播放的功能，那么，依照给出的步骤安装并测试:<br>

[<u>audio_common 使用说明</u>](http://wiki.ros.org/audio_common/Tutorials/Streaming%20audio)

### ##  操作音频包

默认情况下，该包已随版本一起发布:**ros-<*>-audio-common**

简单测试该包有没有被安装，只需在终端简单执行以下命令：

```
roslaunch audio_capture capture.launch
```

倘若没有再执行以下命令进行安装

```
sudo apt-get install ros-indigo-audio-common
```

官网说明已经很详细，这里就不再说明。不过，该包使用的音频设备是系统默认的录音设备，也就是说对于使用使用像 罗技USB摄像头和 Kinect2 所携带的麦克风，运行音频捕获节点会不起作用。

到这里应该注意到，既然该节点配置了启动文件，那么我们是不是可以去看看配置文件呢？很幸运在 `/opt/ros/indigo/share/audio_capture/launch` 中找到了启动选项，启动有**device**配置项，可如何修改并没有作相应的说名，按照Linux系统的音频设备挂载规则来修改，不幸的并没有起任何作用。

简单查看了一下源码，该包使用的是**GStreamer**库，本着试一下修改源码的态度，直接下载源码自行编译：

按照官网指示，很不幸无法成功编译通过，原因在于库文件少了！按照提示错误：

> 无法找到`gst/app/gstappsink.h`

该文件是存在于插件中，这样就容易处理了，安装对应的版本插件即可，执行：

```
sudo apt-get install libgstreamer1.0-dev
sudo apt-get install libgstreamer-plugins-base1.0-dev
```
	
解决缺少库文件后还需要修改对应的源码，以适用于自己的硬件设备，由于是针对Kinect2的，所以需要对源码进行适当的修改。具体修改地方请参考：<br>

[<u> audio_common 音频捕获源码修改指南 </u>](http://answers.ros.org/question/48684/kinect-microphone-array-linux-hark-in-combination-with-openni/)



注意：一个问题是单通道与多通道的差别，该包对于有多通道要求不适应！搜索到一个日本开发的音频包，感觉很厉害的样子，有兴趣的人可以去了解下：

[<u>HARK 一个堪比 OpenCV的音频工具包</u>](http://www.hark.jp/wiki.cgi?page=HARK%2DKINECT)


---


## 使用ALSA库写应用

前面辛辛苦苦走了很多弯路，到头来感觉还白忙活一场。不过看别人源码也是学习的过程，也不冤。下面介绍使用 ALAS库来开发自己的音频捕获程序。在使用库之前，很多时候官网给出的例程已经可以直接拿来使用了，对于详细的说明也要仔细的查看API文档。

在自己编写代码之前应该尽可能的想清楚需求，不要急急忙忙动手。争取提升代码的可复用性。个人觉得应该往以下方面靠拢

- 编写该代码的用途
- 函数接口如何预留
- 后期可能遇到的情况

对于音频采集这个小功能，想清楚也要花点心思。首先，对于设备初始化，程序运行可能使用的参数配置；其次获取的音频数据要以哪种方式存储，是流还是块？或是文件？因为考虑到采集的数据会被进一步处理，而就我目前所想到的，使用块或者文件存储会比较方便，因为数字信号处理的滤波、傅立叶变换等使用特定大小的块会比较方便。正是这个原因，在ROS音频捕获节点中我使用了块（也就是固定大小的数组）来存储数据，这样在话题订阅中就可以很方便的使用C++中的容器操作，亦或是Python中的列表操作，这样能提高开发的效率。

最后在动手时尽量使用最简单的测试框架，在调试时使用以下的方式来测试执行效果。

### ##  API测试伪代码

对于 Linux 来说，一切皆文件，除了套接字有点不一样。那么操作音频设备也像操作文件一样简单。


```

打开回放或录音接口

设置硬件参数(访问模式,数据格式,信道数,采样率,等等)

while 有数据要被处理
{
　　　读PCM数据(录音)　或 写PCM数据(回放)
}

关闭接口

```

这个程序和ROS并没有挂钩，只要功能实现了，再把ROS的节点模板一套，就很简单了。在具体编写实现应用之前，最好看看相关的API说明，这样能起到事半功倍的效果。为满足上面提到的伪代码，需要使用下面几个API，具体解析如下面所述。


### ##  API详解

- 设备打开操作

```c
/* 
* 参数说明：
*  		pcm：当设备打开成功时会填充该结构体以此来充当会话语句柄，类似打开文件操作
* 		name：设备名：plughw:i,j 若使用系统默认设备则填充 default
*		stream：枚举变量
*						SND_PCM_STREAM_PLAYBACK ：向声卡写数据
* 						SND_PCM_STREAM_CAPTURE：向声卡读数据
* 		mode：填0即可
*/
int snd_pcm_open(snd_pcm_t **pcm, const char *name, 
		 snd_pcm_stream_t stream, int mode);
```


- 设备配置 

由于硬件配置有对应的结构体，按照C的编程套路,使用时需要自行分配结构体的内存空间

```c
//参数为打开设备返回的语句柄
int snd_pcm_hw_params_malloc(snd_pcm_hw_params_t **ptr);

//成功分配完结构体后要简单填充，使程序健壮
int snd_pcm_hw_params_any(snd_pcm_t *pcm, snd_pcm_hw_params_t *params);
```

配置设备，API提供以下类型函数，只需根据需要简单调用即可

```c
// 注意参数使用必须使用打开设备申请到的 pcm 语句柄
snd_pcm_hw_params_set_xxx()
```

- 读/写 数据

 这里需要注意一点，对于数据存储模式不同时需要选择不同的读写函数

```c
//数据以交错模式存储
snd_pcm_sframes_t snd_pcm_writei(snd_pcm_t *pcm, const void *buffer, snd_pcm_uframes_t size);
snd_pcm_sframes_t snd_pcm_readi(snd_pcm_t *pcm, void *buffer, snd_pcm_uframes_t size);

//数据以非交错模式存储
snd_pcm_sframes_t snd_pcm_writen(snd_pcm_t *pcm, void **bufs, snd_pcm_uframes_t size);
snd_pcm_sframes_t snd_pcm_readn(snd_pcm_t *pcm, void **bufs, snd_pcm_uframes_t size);
```
最明显的可以看看缓存指针，对于非交错模式下，多通道采样时每个通道各占一维。

- 关闭

注意在没有使用时一定要把该语句柄释放，防止占用端口导致其它程序无法正常进行

```c
int snd_pcm_close(snd_pcm_t *pcm);
```

### ## 关键点

- 打开设备时返回会话语句柄与打开一个文件返回的fd参数一个道理，对于C++对象编程时，需要注意该语句柄的生存周期，需要长期有效直到对象销毁

- 对于多通道，硬件的设置参数推荐使用非交错模式

- ROS 无非就是一套通信框架，只要适当修改即可实现不同节点间的通信，也就是不同进程间通信，只不过它使用的是ROS TCP/IP 协议，本质上也是网络协议。

### ## 完整实现代码请查阅

- [ALSA音频捕获封装](https://github.com/hntea/speech-system-zh/tree/master/src/lib/audio_bace) ：该库文件可以直接使用。

- [Kinect2.0 麦克风阵列捕获音频节点例程](https://github.com/hntea/speech-system-zh/blob/master/src/audio_capture/src/kinect2_capture.cpp)<br>


正常来说运行上面的ROS节点已经能够使用KINECT2.0来对音频进行捕获了。不过捕获后的数据要如何处理呢？如何实现更高级的操作，如信号的预处理，声源定位，语音活性检测等是接下去要处理的问题。其实能够直接使用别人开发好的不是更好吗？很遗憾到目前为止我还没找到能在Ubuntu下使用KINECT2.0麦克风阵列的例程，为何只开发Window的？！
