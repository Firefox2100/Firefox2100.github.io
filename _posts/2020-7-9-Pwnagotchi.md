---
title: Pwnagotchi配置及测试笔记
date: 2020-7-9 22:44:20
categories:
- Raspberry Pi
tags:
- Pwnagotchi
- RTC
- E-Paper
---

> 这周一偶然发现Pwnagotchi更新了，于是在更新我的镜像和配置的同时，我也尝试为我的设备加入了更多的功能，断断续续的折腾了几天，终于完成了，在这里记录一下过程，因为中国国内这方面资料极少，所以起一个备忘的作用。同时因为我成功在这个项目上**同时使用了UPS Lite，DS1302 RTC模块和Waveshare 2.13 inch e-paper显示屏**，根据我在论坛上搜索的结果，有很少的人尝试过这种配置，但是没有人发出完整的配置教程和配置方法，所以作以记录，供有类似需要的人参考。

## Pwnagotchi简介

Pwnagotchi是一个基于强化学习人工智能的项目，目的是尝试这种人工智能在信息安全领域的应用，官方网站在[这里](https://pwnagotchi.ai/)。它是使用基于Python编写的神经网络和bettercap结合，尝试自动抓取WiFi握手包并微调自身参数的设计。官方介绍是：

> Instead of merely playing Super Mario or Atari games like most reinforcement learning based “AI” (yawn), Pwnagotchi tunes its own parameters over time to get better at pwning WiFi things in the environments you expose it to.

同时它也是一个有些怀旧的项目，向我们童年的记忆Tamagotchi致敬。所以它的界面也走的是可爱风格。~~一定几率降低被发现在破解WiFi的概率~~

Pwnagotchi是为Raspberry Pi 0w设计的，但是根据用户测试，官方提供的镜像同时也可以在RPi 3/RPi 4上运行，而只要依赖满足，这个项目也可以被单独安装在任何GNU/Linux发行版上（但是我尝试了很久还没有成功，后续如果完成了我会发出对应的笔记）。

## 安装并配置镜像

官方推荐的安装方法是直接使用已经配置好的镜像，在其[GitHub Release](https://github.com/evilsocket/pwnagotchi/releases)页上提供。镜像的安装方法非常简单，使用一张大于8G（个人建议最好16G以上，因为后续折腾的时候不确定需要多少空间，而且读写速度一定要够）的Micro SD卡，只分一个区并且格式化为FAT32文件格式。然后烧写入下载的镜像即可。在Windows环境下建议使用Balena Etcher烧写存储卡。完成之后不要急着拔出卡，还需要写入一个初始配置文件用来初始化Pwnagotchi服务。

此时电脑能够识别出存储卡内的boot分区，FAT32格式。在boot分区根目录下新建一个文件，名为``config.toml``。这个文件包含基础的配置信息，在Pwnagotchi第一次启动之后会被移入文件系统的``/etc/pwnagotchi/config.toml``，并从boot分区移除。之后直接修改新的文件即可。在启动一次插件配置之后这个文件将包含完整的配置文本，但是我们现在不需要配置全部的信息，只需要让它能够启动即可。在文件里写入：

```yml
main.name = "pwnagotchi"
main.lang = "en"
main.whitelist = [
  "EXAMPLE_NETWORK",
  "ANOTHER_EXAMPLE_NETWORK",
  "fo:od:ba:be:fo:od",
  "fo:od:ba"
]

main.plugins.grid.enabled = true
main.plugins.grid.report = true
main.plugins.grid.exclude = [
  "YourHomeNetworkHere"
]

ui.display.enabled = true
ui.display.type = "waveshare_2"
ui.display.color = "black"

main.plugins.bt-tether.enabled = false

main.plugins.bt-tether.devices.android-phone.enabled = false          # the name of this entry is android-phone
main.plugins.bt-tether.devices.android-phone.search_order = 1         # in which order the devices should
                                                                      ## be searched. E.g. this is #1
main.plugins.bt-tether.devices.android-phone.mac = ""                 # you need to put your phones
                                                                      ## bt-mac here (settings > status)
main.plugins.bt-tether.devices.android-phone.ip = "192.168.44.44"     # this is the static ip of your pwnagotchi
                                                                      ## adjust this to your phones pan-network
                                                                      ## (run "ifconfig bt-pan" on your phone)
                                                                      ## if you feel lucky,
                                                                      ## try: 192.168.44.44 (Android) or
                                                                      ## 172.20.10.6 (iOS)
                                                                      ## 44 is just an example, you can choose
                                                                      ## between 2-254 (if netmask is 24)
main.plugins.bt-tether.devices.android-phone.netmask = 24             # netmask of the PAN
main.plugins.bt-tether.devices.android-phone.interval = 1             # in minues, how often should
                                                                      ## the device be searched
main.plugins.bt-tether.devices.android-phone.scantime = 10            # in seconds, how long should be searched
                                                                      ## on each interval
main.plugins.bt-tether.devices.android-phone.max_tries = 10           # how many times it should try to find the
                                                                      ## phone (0 = endless)
main.plugins.bt-tether.devices.android-phone.share_internet = false   # set to true if you want to have
                                                                      ## internet via bluetooth
main.plugins.bt-tether.devices.android-phone.priority = 1             # the device with the highest
                                                                      ## priority wins (1 = highest)
```

注意这只是一个模板，具体的配置根据自己的需要修改。

## 在使用了微雪2.13寸屏幕和UPS Lite的情况下添加RTC

基础的配置在官网上介绍的非常详细，不再赘述，下面进入本文重点。

树莓派全部型号均没有RTC模块，所以掉电之后时间偏差是很常见的情况。对于能够一直联网（旁路由，气象站等）或者时间是否准确没什么关系（媒体播放器，机器人中控）等的项目没有多大影响，但是很不幸地，Pwnagotchi没有办法一直联网（除非你用手机一直通过蓝牙共享网络），而且也需要准确时间来进行记录和调试等等，所以有必要安装一个RTC模块。而另外一个不幸的是，UPS Lite只有串口检测电池电压和USB转串口的功能，并不像为树莓派3B等设计的UPS一样自带RTC，所以这表示我需要额外装一个硬件模块。

但是我同时使用了微雪的2.13寸电子墨水屏，为了避免引脚冲突，我进行了必要的调整，下面详细说明调整过程。

首先，查找微雪的资料，找到这一屏幕的原理图。官网在[这里](https://www.waveshare.net/wiki/2.13inch_e-Paper_HAT)。分析原理图发现，除了VCC/GND之外，这一屏幕还使用了另外6根引脚，分别是：

- GPIO0，即BCM17，是树莓派的GPIO引脚之一，连接屏幕的RST。
- GPIO5，即BCM24，也是GPIO引脚，连接屏幕的BUSY。
- GPIO6，即BCM25，也是GPIO引脚，连接屏幕的D/C。
- MOSI，即BCM10，是树莓派SPI总线的主输出从输入，连接屏幕的DIN。
- SCLK，即BCM11，是树莓派SPI总线的同步时钟，连接屏幕的CLK。
- CE0，即BCM8，是树莓派两条SPI总线其中一条的片选使能信号，连接屏幕的CS。

可以看出来这个屏幕是SPI设备，使用了一条SPI总线。再查找UPS Lite的资料（其实不用找，挺明显的......）UPS用了引脚编号为1~10的10根引脚，分别是：

- 1根3.3V，两根5V，两根GND，这是供电部分，UPS本身的作用就是供电，这没什么好说的。
- SDA.1，即BCM2，是树莓派的I2C信号线，用来进行电池信息的获取。
- SCL.1，即BCM3，是树莓派的I2C时钟线。注意树莓派默认只有一条I2C总线，如果还需要的话就需要另外指定新的总线。
- GPIO7，即BCM4，是树莓派的一个GPIO口，这里的资料我没有看，不过应该是片选或者复位一类的信号线。
- RXD/TXD，即BCM15/14，是树莓派的串口。本来UPS是不需要这个接口的，但是UPS Lite将这一接口引至板子上自带的USB转串口芯片，允许直接用电脑连接UPS的接口就可以一边给UPS供电一边打开串口Shell，方便调试。

可以看出来UPS Lite是一个I2C设备，和屏幕既没有引脚冲突也没有总线或地址冲突，可以一起使用。但是，RTC时钟一般是I2C或SPI的，都有潜在的冲突风险。这里有这样的几个选择：

- 一是使用I2C时钟，因为一方面可以新建立I2C总线，另一方面，UPS Lite避开了RTC模块使用的绝大多数地址，所以不会出现地址冲突，即使共用引脚和总线也没有关系。一般常见的使用I2C的RTC模块是基于PCF8523，DSL1307或DS3231等芯片的。这个方案有一个问题，就是如果要共用引脚，因为UPS使用顶针连接树莓派排针的背面，所以焊接线路必须飞线焊接在排针正面，容易断，而且也很难做绝缘。开启新的总线会对设备造成压力，而且不利于使用其他的I2C设备（各种传感器大多是I2C的）。
- 还可以选择使用SPI的RTC模块。树莓派本身有两个SPI总线，所以很容易就能安装两个SPI设备并确保不会冲突。但是这样需要WiringPi的支持，很多项目不兼容这一驱动方式，大部分不是Raspbian（现在貌似叫Raspberry OS）的发行版都用不了。所以我选择了另外一个方式，就是使用一个第三方的C程序，专门用来处理RTC设备。这样因为C语言直接操作BCM2835芯片，也能进行底层编程，就不需要SPI总线，只需要三个GPIO就可以，和其他设备都不会冲突。

我获取这一个驱动程序的位置在[这里](https://zedt.eu/tech/hardware/using-ds1302-real-time-clock-module-raspberry-pi/)，我使用了一个国产的DS1302芯片的板子，取下了上面的芯片座并且将芯片直接焊在板子上，取下电池座用电工胶布直接把飞线粘在电池上来减小板子的厚度，换了进口的晶振确保稳定性。有一个使用DS1302的注意事项就是，**务必在VCC和DAT之间焊一个电阻**，我用的是20K的，否则没有上拉电阻，DAT输出很容易乱码，一旦树莓派把格式错误的时间装载进系统就会永久死机。当然如果你不连接VCC也是可以的。DS1302是双电源输入，引出来的其实是VCC1，在VCC1>VCC2+0.2V的时候会切换至VCC1供电，就是使用树莓派的3.3V电源；当不满足条件的时候使用VCC2供电，我的VCC2接的是CR2032电池，标准输出接近3V，保证RTC一直有电。如果不连接VCC1，就会一直用电池供电，虽然比较耗电，但是稳定一些。

下载驱动程序之后打开看一下，发现使用的是以下引脚（除了VCC和GND）

- GPIO0，即BCM17，连接模块的RST。
- GPIO1，即BCM18，连接模块的DAT。
- GPIO2，即BCM27，连接模块的CLK。

于是发现了一个问题......就是GPIO0冲突了。因为屏幕是物理接线决定的，很难修改，所以只能改这一个驱动的配置。下面进入重点：BCM2835的寄存器配置。

## BCM2835寄存器配置

用上文提到的驱动程序中的配置作为示例。首先我们可以看到：

```C
#define GPIO_ADD    0x20200000L // physical address of I/O peripherals on the ARM processor
#define GPIO_SEL1   1           // Offset of SEL register for GP17 & GP18 into GPIO bank
#define GPIO_SEL2   2           // Offset of SEL register for GP21 into GPIO bank  
#define GPIO_SET    7           // Offset of PIN HIGH register into GPIO bank  
#define GPIO_CLR    10          // Offset of PIN LOW register into GPIO bank  
#define GPIO_INP    13          // Offset of PIN INPUT value register into GPIO bank  
#define PAGE_SIZE   4096
#define BLOCK_SIZE  PAGE_SIZE
```

GPIO_ADD定义为``0x20200000``。查阅手册第6页，发现，物理地址``0x20000000``到``0x20FFFFFF``是分配给周边设备的。总线地址``0x7Ennnnnn``被映射到了``0x20nnnnnn``的物理地址。继续查阅，在手册第90页，发现GPFSEL，即GPIO Function SElect配置寄存器的启示地址是``0x7E200000``。所以将GPIO_ADD定义为``0x20200000``。继续查阅，还是在90页，得知GPFSEL是32位一组，地址偏移4个字节，也就是说紧密相连的。在92页发现，32位的寄存器被分为3*10组，分别对应10个GPIO的状态配置。那么，对于0~9号GPIO（注意这里是**BCM编号不是物理编号**），可以将基础偏移设为0，10~19是1，以此类推。同样在92页找到配置表。

```C
#define IO_INPUT    *(gpio+GPIO_SEL1) &= 0xF8FFFFFFL
#define CE_OUTPUT   *(gpio+GPIO_SEL1) &= 0xFF1FFFFFL; *(gpio+GPIO_SEL1) |= 0x00200000L
```

这是源程序中的两个配置。通过查表我们发现，若要将某一个GPIO配置为输入，那么需要将它的寄存器置为000。那么，思考这样的情况：如果我直接对寄存器赋值，势必影响全部32位的寄存器，这样可能在无意间改变了其他GPIO的功能。所以，将现有值和一个数值与运算，这个数值在这三位是000，其他位数是1，就确保了与之后这三位一定是0，其他的本来是什么还是什么。这个``0xF8FFFFFF``的计算方式就是\\(FFFFFFFFH-(111B<<3*n)\\)，n是该分组内的GPIO编号，例如，IO这个引脚是18，那么就需要位移\\(3*8=24\\)位。

同样地，如果需要将某个GPIO配置为输出，需要将对应寄存器设置为001。那么此时如果用与，不能保证一定形成001，还可能形成000。用或则可能形成111.所以先用一次与将其置为000，再用或置为001。地址的计算方法和上面是一样的。所以按照这个方法我们将代码修改为：

```C
#define CE_OUTPUT   *(gpio+GPIO_SEL2) &= 0xFFFFFE3FL; *(gpio+GPIO_SEL2) |= 0x00000040L
```

我将CE配置到了GPIO3，也就是BCM22。继续往下看：

```C
#define CE_HIGH     *(gpio+GPIO_SET) = 0x00020000L  
#define CE_LOW      *(gpio+GPIO_CLR) = 0x00020000L
```

熟悉单片机或者类似的通信的可以看出来，这是控制输出高低电平的寄存器和储存输入信号的寄存器。回到手册90页，可以发现，控制输出的寄存器地址偏移是7组，32位，每一位对应一个GPIO；输入的偏移则是10，同样是一位对应一个GPIO。所以按照一一对应，这段代码可以修改为：

```C
#define CE_HIGH     *(gpio+GPIO_SET) = 0x00400000L  
#define CE_LOW      *(gpio+GPIO_CLR) = 0x00400000L
```

按照这个原则可以按自己的喜好定义不同的GPIO功能和状态。

## 完成状态

这是现在我的完成状态，之后会给它设计一个外壳。

<img src="{{site.baseurl}}/assets/images/in_posts/2020_7_9/1.jpg">

<img src="{{site.baseurl}}/assets/images/in_posts/2020_7_9/2.jpg">
