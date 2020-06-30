---
title: Proxmark3全指令入门
date: 2020-6-16 9:31:14
categories:
- RFID
tags:
- Proxmark3
---

> 很久以前我刚接触Proxmark3这种设备的时候，一直苦于它是一个面向专业开发者和安全研究者的设备，对RFID新手并不十分友好。最近刚好有一点时间，也碰巧在用Proxmark3进行研究和测试，所以现在记录一下Proxmark3 RRG固件支持的指令，这些指令的格式，特性等等，一方面作为完整的熟悉这一设备的方法，另一方面也作为以后的参考，同时也算给刚刚接触这一方面的人一个简单的指引。

## 平台信息

### 虚拟机信息

虚拟机的运行环境是VMware Workstation 15.5 Pro，4G内存，双核CPU，50G硬盘空间。物理机的蓝牙保持开启状态，虚拟机的蓝牙也可以正常启动。

### 操作系统

虚拟机内运行的操作系统是Kali Linux，因为它内置了大多数常用安全设备的驱动和依赖，省去了自己编译的过程。

```Bash
root@kali:~# cat /proc/version
Linux version 5.6.0-kali2-amd64 (devel@kali.org) (gcc version 9.3.0 (Debian 9.3.0-13)) #1 SMP Debian 5.6.14-1kali1 (2020-05-25)
```

### Proxmark客户端及固件

Proxmark3 硬件是RDV4.1版，安装了BlueShark扩展组件，使用标准的Q值可调天线。通过USB数据线连接虚拟机，使用``ttyACM0``。

```Proxmark3
[usb] pm3 --> hw version

 [ Proxmark3 RFID instrument ]

 [ CLIENT ]
  client: RRG/Iceman/master/v4.9237-345-g494a28da 2020-06-13 20:40:16
  compiled with GCC 9.3.0 OS:Linux ARCH:x86_64

 [ PROXMARK3 RDV4 ]
  external flash:                  present
  smartcard reader:                present

 [ PROXMARK3 RDV4 Extras ]
  FPC USART for BT add-on support: present

 [ ARM ]
  bootrom: RRG/Iceman/master/v4.9237-345-g494a28da 2020-06-13 20:33:02
       os: RRG/Iceman/master/v4.9237-345-g494a28da 2020-06-13 20:33:19
  compiled with GCC 8.3.1 20190703 (release) [gcc-8-branch revision 273027]

 [ FPGA ]
  LF image built for 2s30vq100 on 2020-02-22 at 12:51:14
  HF image built for 2s30vq100 on 2020-01-12 at 15:31:16

 [ Hardware ]
  --= uC: AT91SAM7S512 Rev B
  --= Embedded Processor: ARM7TDMI
  --= Nonvolatile Program Memory Size: 512K bytes, Used: 257472 bytes (49%) Free: 266816 bytes (51%)
  --= Second Nonvolatile Program Memory Size: None
  --= Internal SRAM Size: 64K bytes
  --= Architecture Identifier: AT91SAM7Sxx Series
  --= Nonvolatile Program Memory Type: Embedded Flash Memory
```

```Proxmark3
[usb] pm3 --> hw status
[#] Memory
[#]   BigBuf_size.............43584
[#]   Available memory........43584
[#] Tracing
[#]   tracing ................1
[#]   traceLen ...............0
[#] Current FPGA image
[#]   mode.................... HF image built for 2s30vq100 on 2020-01-12 at 15:31:16
[#] Flash memory
[#]   Baudrate................24 MHz
[#]   Init....................OK
[#]   Memory size.............2 mbits / 256 kb
[#]   Unique ID...............0xD567A882A7587725
[#] Smart card module (ISO 7816)
[#]   version.................v2.06
[#] LF Sampling config
[#]   [q] divisor.............95 ( 125.00 kHz )
[#]   [b] bits per sample.....8
[#]   [d] decimation..........1
[#]   [a] averaging...........Yes
[#]   [t] trigger threshold...0
[#]   [s] samples to skip.....0
[#] LF T55XX config
[#]            [r]               [a]   [b]   [c]   [d]   [e]   [f]   [g]
[#]            mode            |start|write|write|write| read|write|write
[#]                            | gap | gap |  0  |  1  | gap |  2  |  3
[#] ---------------------------+-----+-----+-----+-----+-----+-----+------
[#] fixed bit length (default) |  29 |  17 |  15 |  47 |  15 | N/A | N/A |
[#]     long leading reference |  29 |  17 |  15 |  47 |  15 | N/A | N/A |
[#]               leading zero |  29 |  17 |  15 |  40 |  15 | N/A | N/A |
[#]    1 of 4 coding reference |  29 |  17 |  15 |  31 |  15 |  47 |  63 |
[#]
[#] Transfer Speed
[#]   Sending packets to client...
[#]   Time elapsed............500ms
[#]   Bytes transferred.......278528
[#]   Transfer Speed PM3 -> Client = 557056 bytes/s
[#] Various
[#]   Max stack usage so far..4096
[#]   DBGLEVEL................1
[#]   ToSendMax...............-1
[#]   ToSendBit...............0
[#]   ToSend BUFFERSIZE.......2308
[#]   Slow clock..............31177 Hz
[#] Installed StandAlone Mode
[#]   HF - Reading Visa cards & Emulating a Visa MSD Transaction(ISO14443) - (Salvador Mendoza)
[#] Flash memory dictionary loaded
```

## 指令及简单介绍

这一部分将按照客户端里呈现的指令结构进行介绍，方便查阅。

### help

``help``指令用于显示帮助信息。直接使用``help``指令将显示客户端基本的帮助信息。

使用方法：

```Proxmark3
[usb] pm3 --> help
help             This help. Use '<command> help' for details of a particular command.
auto             Automated detection process for unknown tags
analyse          { Analyse utils... }
data             { Plot window / data buffer manipulation... }
emv              { EMV ISO-14443 / ISO-7816... }
hf               { High frequency commands... }
hw               { Hardware commands... }
lf               { Low frequency commands... }
mem              { Flash Memory manipulation... }
reveng           { CRC calculations from RevEng software }
sc               { Smart card ISO-7816 commands... }
script           { Scripting commands }
trace            { Trace manipulation... }
usart            { USART commands... }
wiegand          { Wiegand format manipulation... }

hints            Turn hints on / off
pref             Edit preferences
msleep           Add a pause in milliseconds
rem              Add a text line in log file
quit
exit             Exit program
```

这条指令用在忘记了自己需要输入的指令具体怎么拼写或者具体在哪个分类下的情况下可以用......基本没有实际作用，只是以防万一。

### auto

``auto``指令用来自动检测放置在Proxmark3上的Tag的类型和基本信息，例如UID等等。

```Proxmark3
[usb] pm3 --> auto help
Run LF SEARCH / HF SEARCH / DATA PLOT / DATA SAVE

Usage:  auto <ms>
Options:
       h          This help

Examples:
       auto
```

可以从帮助信息中看到，auto其实是自动执行了四条指令，并且在最后将获得的数据以图表的形式展示出来。执行完之后会打开一组新的plot窗口，显示捕获的信息，并且将信息保存在工作路径下的一个``*.pm3``文件中。``auto``的参数只有一个，可以不带，是用来指定检测之间间隔的毫秒数的。

使用``auto``指令一般是完全不知道卡片类型，连是低频卡还是高频卡都不确定的时候。这个时候需要注意，高频卡一般判断比较准确，但是进行低频卡的搜索的时候有一定概率会将噪声当作读取的卡片信息，特别是在本身不是支持的射频卡的情况下。例如，如果一张老式的银行卡（接触式CPU和磁条）被放置在天线上的时候，因为内部电路的原因，和天线会产生微弱的耦合，会有一些看起来像是数据的噪声。这时候运行低频卡的搜索，可能出现误识别，导致之后无论怎么测试也不对（因为这种银行卡压根没有无线功能）。

### analyse

``analyse``指令用来进行校验操作。

```Proxmark3
[usb] pm3 --> analyse help
help             This help
lcr              Generate final byte for XOR LRC
crc              Stub method for CRC evaluations
chksum           Checksum with adding, masking and one's complement
dates            Look for datestamps in a given array of bytes
tea              Crypto TEA test
lfsr             LFSR tests
a                num bits test
nuid             create NUID from 7byte UID
demodbuff        Load binary string to demodbuffer
```

可以看到，Proxmark3的客户端其实使用的是树形结构的指令树，``analyse``指令的参数其实形成的是不同的指令，在这篇文章中我将这些指令当作独立的指令处理，而不是简单的参数替换。

#### analyse lcr

``analyse lcr``指令是用来通过LRC和来补全最后一位UID的。这条指令有一个参数，是需要补全的UID值加上已知的LRC校验值，并且要求以16进制格式指定。指令的输出是异或校验所需的最后一个字节。

```Proxmark3
[usb] pm3 --> analyse lcr h
Specifying the bytes of a UID with a known LRC will find the last byte value
needed to generate that LRC with a rolling XOR. All bytes should be specified in HEX.

Usage:  analyse lcr [h] <bytes>
Options:
           h          This help
           <bytes>    bytes to calc missing XOR in a LCR

Examples:
      analyse lcr 04008064BA
expected output: Target (BA) requires final LRC XOR byte value: 5A
```

#### analyse crc

``analyse crc``是一个测试性指令，只有一个作用，就是计算输入的序列的CRC值，方便自己手动计算，也可以用来调试自己修改过的固件。它有一个参数，是16进制格式的需要计算CRC的序列。

```Proxmark3
[usb] pm3 --> analyse crc help
A stub method to test different crc implementations inside the PM3 sourcecode. Just because you figured out the poly, doesn't mean you get the desired output

Usage:  analyse crc [h] <bytes>
Options:
           h          This help
           <bytes>    bytes to calc crc

Examples:
      analyse crc 137AF00A0A0D
```

#### analyse chksum

```Proxmark3
[usb] pm3 --> analyse chksum help
The bytes will be added with eachother and than limited with the applied mask
Finally compute ones' complement of the least significant bytes

Usage:  analyse chksum [h] [v] b <bytes> m <mask>
Options:
           h          This help
           v          suppress header
           b <bytes>  bytes to calc missing XOR in a LCR
           m <mask>   bit mask to limit the outpuyt

Examples:
      analyse chksum b 137AF00A0A0D m FF
expected output: 0x61
```

#### analyse dates

#### analyse tea

#### analyse lfsr

#### analyse a

```Proxmark3
[usb] pm3 --> analyse a help
Iceman's personal garbage test command

Usage:  analyse a [h] d <bytes>
Options:
           h          This help
           d <bytes>  bytes to send to device

Examples:
      analyse a d 137AF00A0A0D
```

#### analyse nuid

```Proxmark3
[usb] pm3 --> analyse nuid help
Generate 4byte NUID from 7byte UID

Usage:  analyse hid [h] <bytes>
Options:
           h          This help
           <bytes>  input bytes (14 hexsymbols)

Examples:
      analyse nuid 11223344556677
```

#### analyse demodbuff

```Proxmark3
[usb] pm3 --> analyse demodbuff help
loads a binary string into demod buffer

Usage:  analyse demodbuff [h] <binarystring>
Options:
           h                This help
           <binarystring>   Binary string to load

Examples:
      analyse demodbuff 0011101001001011
```

### data

```Proxmark3
[usb] pm3 --> data help
help             This help
askedgedetect    [threshold] Adjust Graph for manual ASK demod using the length of sample differences to detect the edge of a wave (use 20-45, def:25)
autocorr         [window length] [g] -- Autocorrelation over window - g to save back to GraphBuffer (overwrite)
biphaserawdecode [offset] [invert<0|1>] [maxErr] -- Biphase decode bin stream in DemodBuffer (offset = 0|1 bits to shift the decode start)
bin2hex          <digits> -- Converts binary to hexadecimal
bitsamples       Get raw samples as bitstring
buffclear        Clears bigbuff on deviceside and graph window
convertbitstream Convert GraphBuffer's 0/1 values to 127 / -127
dec              Decimate samples
detectclock      [<a|f|n|p>] Detect ASK, FSK, NRZ, PSK clock rate of wave in GraphBuffer
fsktonrz         Convert fsk2 to nrz wave for alternate fsk demodulating (for weak fsk)
getbitstream     Convert GraphBuffer's >=1 values to 1 and <1 to 0
grid             <x> <y> -- overlay grid on graph window, use zero value to turn off either
hexsamples       <bytes> [<offset>] -- Dump big buffer as hex bytes
hex2bin          <hexadecimal> -- Converts hexadecimal to binary
hide             Hide graph window
hpf              Remove DC offset from trace
load             <filename> -- Load trace (to graph window
ltrim            <samples> -- Trim samples from left of trace
rtrim            <location to end trace> -- Trim samples from right of trace
mtrim            <start> <stop> -- Trim out samples from the specified start to the specified stop
manrawdecode     [invert] [maxErr] -- Manchester decode binary stream in DemodBuffer
norm             Normalize max/min to +/-128
plot             Show graph window (hit 'h' in window for keystroke help)
printdemodbuffer [x] [o] <offset> [l] <length> -- print the data in the DemodBuffer - 'x' for hex output
rawdemod         [modulation] ... <options> -see help (h option) -- Demodulate the data in the GraphBuffer and output binary
samples          [512 - 40000] -- Get raw samples for graph window (GraphBuffer)
save             Save trace (from graph window)
setgraphmarkers  [orange_marker] [blue_marker] (in graph window)
scale            <int> -- Set cursor display scale in carrier frequency expressed in kHz
setdebugmode     <0|1|2> -- Set Debugging Level on client side
shiftgraphzero   <shift> -- Shift 0 for Graphed wave + or - shift value
dirthreshold     <thres up> <thres down> -- Max rising higher up-thres/ Min falling lower down-thres, keep rest as prev.
tune             Get hw tune samples for graph window
undec            Un-decimate samples by 2
zerocrossings    Count time between zero-crossings
iir              apply IIR buttersworth filter on plotdata
ndef             Decode NDEF records
```
