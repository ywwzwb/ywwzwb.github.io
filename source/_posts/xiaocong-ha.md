---
title: 小葱智能墙面插座连接HomeAssistant
date: 2021-10-13 16:34:00
tags: 
- 智能家具
- HomeAssistant
- ESPHome
categories:
- 技术
- 智能家具
---
最近在闲鱼上面买了一个智能插座，带电量检测，20一个，买了一个回家折腾。

## 太长不看

刷入esphome 固件，连到homeassistant. 按键，蓝灯，白灯，继电器分别在5, 16, 4, 14 上。其中两个灯都是低电平开启。
电量检测芯片是v9881, 不知道是什么协议，暂时不折腾。
<!-- more -->
## 正文

机器懒得拆了，上几张买家的图看看吧
![卖家给的接口定义](https://tva1.sinaimg.cn/large/001ur7p1gy1gvdouer07xj60v91votkd02.jpg)
串口接线图
![接线定义](https://tva1.sinaimg.cn/large/001ur7p1gy1gvdovvlsgrj60u01404aq02.jpg)

计量芯片暂时搞不定，就暂时当一个普通智能插座吧
![计量芯片](https://tva1.sinaimg.cn/large/001ur7p1gy1gvdox0qzkdj60u00mgn2f02.jpg)

App 搜了一下，苹果市场貌似下架了。官方平台估计不好使了，自己刷固件折腾了一下。
拆开发现wifi 芯片是 ESP-WROOM-02, 通讯接口都引出来了, 接上3.3v，usb-ttl ，刷机测试。
刷机过程有点曲折。
### 1.0 版本
先来说说背景
因为服务器后期需要连接一个 usb zigbee 网关，我以为 esxi 只能直通u盘，串口设备可能连不上去， 就用树莓派做服务器。
树莓派是老旧的树莓派3b, 性能够呛，一度出现编译固件就直接死机的问题。
扩展虚拟内存后，勉强看可以编译，不过编译一次需要10分钟以上，而且编译出来的固件也有问题，启动不了，我一度以为这个wifi 模块有什么特殊之处。

1.0版本是我自己写的固件，使用mqtt 协议直接和homeassistant 通信。不过我太懒了，写的程序只能连接固定的设备，也不能网络升级，凑合着用。
### 2.0 版本
由于树莓派3b 实在是性能差劲，一直考虑买一个树莓派4，奈何预算有限。
最近想起来还是试一试应该试试虚拟机直通串口设备，发现居然可以，感觉发现了新大陆，使用另一块 esp8266 开发版接上去，直接通过usb刷机，一点问题没有，大喜过望。

又把已经装好的插座拆下来，焊接上数据线，刷机，连接成功，我去，折腾了这么多天。
花了点时间，把配置文件写好了，短按按钮可以开关继电器，长按重启设备。
另外，v9881 目前不知道怎么通信，以后有时间再慢慢折腾。可以 ota 就好办了。
附上我的esphome 配置文件。
``` yml
esphome:
  name: kitchen_plugin_1
  platform: ESP8266
  board: esp_wroom_02

# Enable logging
logger:

# Enable Home Assistant API
api:

ota:
  password: "xxxxx"

wifi:
  ssid: "xxxxxx"
  password: "yyyyy"

  # Enable fallback hotspot (captive portal) in case wifi connection fails
  ap:
    ssid: "zzzzzz"
    password: "yyyy"

captive_portal:
switch: 
  - platform: gpio
    name: "switch state led"
    id: "switch_state"
    internal: true
    pin:
      number: GPIO4
      inverted: true
  - platform: gpio
    name: "插座开关"
    id: "main_switch"
    pin: GPIO14
    on_turn_on:
        then:
          - switch.turn_on: switch_state
    on_turn_off:
      then:
        - switch.turn_off: switch_state
  - platform: restart
    id: "restart_switch"
    internal: true
binary_sensor:
  - platform: gpio
    name: "internal key"
    pin:
      number: GPIO5
      inverted: true
    internal: true
    on_click:
      - min_length: 50ms
        max_length: 350ms
        then:
          - switch.toggle: main_switch
      - min_length: 1000ms
        max_length: 5000ms
        then:
          - switch.toggle: restart_switch
status_led:
  pin:
    number: GPIO16
    inverted: true
```



