<!--
author: os4uinfo
head: https://os4u.info/blog/img/sun.png
date: 2017-07-06
title: H3C的POE功能配置
tags: Network
images: https://os4u.info/blog/img/sun.png
category: H3C
status: publish
summary: 现在公司用的无线设备是基于POE的瘦终端，接入层的交换机上POE模块运行时间久了，导致POE的电源模块出现问题，最终导致无线终端因无法通过POE供电而失联。本文主要是定位原因和解决问题而形成的。
-->


### 首先看下组网方式

![topology](https://www.os4u.info/blog/network/images/topology-1.png)


### 然后查看故障设备（主要查版本号）

```
<DEV-SW05>display version
H3C Comware Software, Version 7.1.045, Release 3106
Copyright (c) 2004-2014 Hangzhou H3C Tech. Co., Ltd. All rights reserved.
H3C S5130-28S-HPWR-EI uptime is 7 weeks, 2 days, 22 hours, 16 minutes
Last reboot reason : USER reboot

Boot image: flash:/s5130ei_e-cmw710-boot-r3106.bin
Boot image version: 7.1.045, Release 3106
  Compiled Jul 31 2014 08:36:45
System image: flash:/s5130ei_e-cmw710-system-r3106.bin
System image version: 7.1.045, Release 3106
  Compiled Jul 31 2014 08:36:45


Slot 1:
Uptime is 7 weeks,2 days,22 hours,16 minutes
H3C S5130-28S-HPWR-EI with 1 Processor
BOARD TYPE:         H3C-S5130-28S-HPWR-EI
DRAM:               992M bytes
FLASH:              512M bytes
PCB 1 Version:      VER.B
Bootrom Version:    109
CPLD 1 Version:     002
Release Version:    H3C S5130-28S-HPWR-EI-3106
Patch Version  :    None
Reboot Cause  :     UserReboot
[SubSlot 0] 24GE+4SFP Plus
<DEV-SW05>display current-configuration
#
 version 7.1.045, Release 3106
#
 sysname DEV-SW05
#
 irf mac-address persistent timer
 irf auto-update enable
 undo irf link-delay
 irf member 1 priority 1
#
 lldp global enable
#
 loopback-detection global enable vlan 11 250 253
 loopback-detection global action shutdown
#
 password-recovery enable
#
 poe enable pse 4
#
vlan 1
#
---- More ----
```
网上查到一些解决方案见这个文档--> [文档](https://www.os4u.info/blog/network/files/poe-error.pdf)


是开启了poe支持的。但是接口还是无法使用的。查看这个pse信息（发现PSE的版本信息都没，看来是坏了）

```
<DEV-SW05>display poe pse
 PSE ID                           : 4
 Slot No.                         : 1
 SSlot No.                        : 0
 PSE Model                        : LSP7POED
 PSE Status                       : Enabled
 Power Priority                   : Low
 Current Power                    : 0.0      W
 Average Power                    : 0.0      W
 Peak Power                       : 0.0      W
 Max Power                        : 370.0    W
 Remaining Guaranteed Power       : 370.0    W
 PSE CPLD Version                 : -
 PSE Software Version             : 0
 PSE Hardware Version             : 0
 Legacy PD Detection              : Disabled
 Power Utilization Threshold      : 80
 PD Power Policy                  : Disabled
 PD Disconnect-Detection Mode     : DC
```

在公司机房中找设备，发现好几台交换机设备。还以为都支持POE的。如是乎，进入交换机开启poe支持。

```
<DEV-SW04>syste
<DEV-SW04>system-view
System View: return to User View with Ctrl+Z.
[DEV-SW04]poe enable pse ?
  NO PSE.

[DEV-SW04]poe enable pse
                            ^
 % Incomplete command found at '^' position.
```
这就尴尬啦，没法玩了。我就继续找...

### 继续寻找合适的交换机

终于找到一台，且支持POE功能的交换机，里面替换上就好了。至此POE问题解决，找到的设备信息如下：

```
DEV-SW09>display version
H3C Comware Software, Version 7.1.045, Release 3106
Copyright (c) 2004-2014 Hangzhou H3C Tech. Co., Ltd. All rights reserved.
H3C S5130-52S-PWR-EI uptime is 0 weeks, 0 days, 2 hours, 55 minutes
Last reboot reason : Cold reboot

Boot image: flash:/s5130ei_e-cmw710-boot-r3106.bin
Boot image version: 7.1.045, Release 3106
  Compiled Jul 31 2014 08:36:45
System image: flash:/s5130ei_e-cmw710-system-r3106.bin
System image version: 7.1.045, Release 3106
  Compiled Jul 31 2014 08:36:45


Slot 1:
Uptime is 0 weeks,0 days,2 hours,55 minutes
H3C S5130-52S-PWR-EI with 1 Processor
BOARD TYPE:         H3C-S5130-52S-PWR-EI
DRAM:               992M bytes
FLASH:              512M bytes
PCB 1 Version:      VER.B
Bootrom Version:    109
CPLD 1 Version:     002
Release Version:    H3C S5130-52S-PWR-EI-3106
Patch Version  :    None
Reboot Cause  :     ColdReboot
[SubSlot 0] 48GE+4SFP Plus

DEV-SW09>display poe pse 4
 PSE ID                           : 4
 Slot No.                         : 1
 SSlot No.                        : 0
 PSE Model                        : LSP7POEB
 PSE Status                       : Disabled
 Power Priority                   : Low
 Current Power                    : 25.6     W
 Average Power                    : 25.3     W
 Peak Power                       : 28.6     W
 Max Power                        : 370.0    W
 Remaining Guaranteed Power       : 370.0    W
 PSE CPLD Version                 : -
 PSE Software Version             : 140
 PSE Hardware Version             : 57633
 Legacy PD Detection              : Disabled
 Power Utilization Threshold      : 80
 PD Power Policy                  : Disabled
 PD Disconnect-Detection Mode     : DC
 
```

至此，问题终于解决，可以欢快的玩无线咯。

### 参考资料

> [H3C 交换机POE功能配置文档](https://www.os4u.info/blog/network/files/H3C-SWITCH-POE-Config.pdf)


![微信公众号](https://www.os4u.info/wx.jpg) 

:) 微信扫一扫 关注公众号 