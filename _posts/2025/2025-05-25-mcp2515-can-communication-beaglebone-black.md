---
title: BeagleBone Black - SPI to CAN Communication (MCP2515)
date: 2025-05-25 9:38:00 +0800
categories: [Linux Kernel]
tags: [linux kernel, beagle bone, CAN]
image: /assets/img/posts/can_communication_bbb/cover.png
---
<div style="text-align: justify">
BeagleBone doesn’t have built-in CAN protocol, but we can take advantage of the SPI protocol in it to convert into CAN. In this article, I have a small project using an external SPI to CAN module, integrating with BeagleBone to use CAN communication. Let’s get started.
</div>

## What is CAN ?
<div style="text-align: justify">
CAN – Controller Area Network protocol is a robust, serial communication protocol designed for real-time, high-speed data exchange between electronic control units (ECUs) in vehicles and other embedded systems.
</div>
<div style="text-align: justify">
CAN supports speeds up to 1 Mbps for high-speed CAN, while CAN FD (Flexible Data-rate) enables faster and more flexible data rates (up to 5 Mbps).
</div>
<div style="text-align: justify">
In the system, many CAN nodes can connect to the same network, helping to save wires. CAN provides mechanisms to indentify data, prioritized communication, error detection and handling, etc.
</div>

![](/assets/img/posts/can_communication_bbb/can1.webp)

<div style="text-align: justify">
CAN uses two wires (CAN_H and CAN_L) for differential signaling, making it highly resistant to electrical noise. Simply put, this mechanism determines whether the signal is high or low not relative to GND like TTL, but based on the voltage difference between the two wires CAN_H and CAN_L, so it will be more difficult to be disturbed.
</div>

![](/assets/img/posts/can_communication_bbb/can2.webp)

## MCP2515 Module
<div style="text-align: justify">
In this project, I use MCP2515 SPI to CAN module. This module use MCP2515 as an IC controller to convert SPI signal in to CAN signal, then use TJA as a CAN transceiver IC to turn CAN signal into 2 wire signal CAN_H and CAN_L. From this, we can connect CAN_H and CAN_L to any CAN bus system.
</div>

![](/assets/img/posts/can_communication_bbb/mcp2515_1.webp)

Linux kernel provides a driver to control this module: [mcp251x.c](https://github.com/torvalds/linux/blob/master/drivers/net/can/spi/mcp251x.c)

I created a **.dtsi** file to include into `am335x-boneblack.dts` and named it `am335x-bone-mcp2515.dtsi`

![](/assets/img/posts/can_communication_bbb/mcp2515_2.webp)

## Pin Connection
I use SPI0 to control MCP2515 module, so that first we need to disable SPI0 pins.

![](/assets/img/posts/can_communication_bbb/pin_1.webp)

About IRQ pin, I use P8_11. What is IRQ pin ? According to [MCP2515-Stand-Alone-CAN-Controller-with-SPI-20001801J.pdf](https://ww1.microchip.com/downloads/en/DeviceDoc/MCP2515-Stand-Alone-CAN-Controller-with-SPI-20001801J.pdf)
> The general purpose interrupt pin, as well as status registers (accessed via the SPI interface), can also be used to determine when a valid message has been received.

SPI0 pins have MODE_0, According to [BBB_SRM.pdf](https://cdn-shop.adafruit.com/datasheets/BBB_SRM.pdf)

![](/assets/img/posts/can_communication_bbb/pin_2.webp)

Enable these pins with SPI mode.

![](/assets/img/posts/can_communication_bbb/pin_3.webp)

Let me explain a little bit about IRQ pin configuration: `pinctrl-single,pins = < 0x034 0x37 >;` We have syntax of `pinctrl-single,pins` like this:
```
pinctrl-single,pins = <PIN_OFFSET PIN_CONFIG>;
```
* `PIN_OFFSET`: check the following table in [eagleboneBlackP8HeaderTable.pdf](https://github.com/derekmolloy/exploringBB/blob/master/chp06/docs/BeagleboneBlackP8HeaderTable.pdf) is **0x034**
![](/assets/img/posts/can_communication_bbb/pin_4.webp)
* `PIN_CONFIG` = `PIN_FUNCTION + PIN_MUX_MODE` = `0x37 = (1 << 5) | (1 << 4) | 0x7`
  * `1 << 5`: INPUT_EN (include/dt-bindings/pinctrl/am33xx.h)
  * `1 << 4`: PULL_UP (include/dt-bindings/pinctrl/omap.h)
  * `0x7`: MUX_MODE7: GPIO mode (include/dt-bindings/pinctrl/omap.h)

⇒ `P8_11` is used as a GPIO pin, input pull up for interrupt function.

## MCP2515 Node
### Define clock
![](/assets/img/posts/can_communication_bbb/node1.webp)

### Define CAN node
![](/assets/img/posts/can_communication_bbb/node2.webp)

### Ensure mcp251x.c is built as kernel driver
![](/assets/img/posts/can_communication_bbb/node3.webp)
![](/assets/img/posts/can_communication_bbb/node4.webp)
![](/assets/img/posts/can_communication_bbb/node5.webp)

## Output
Build kernel source, then use image and dtb to boot. Now Beaglebone can use CAN.
### Detect MCP2515 module successfully
```
root@BeagleBone:~# dmesg | grep mcp
[ 5.434073] mcp251x spi0.0 can0: MCP2515 successfully initialized.
root@BeagleBone:~#
```

### Check CAN node
```
root@BeagleBone:~# ifconfig -a can0
can0: flags=128<NOARP> mtu 16
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00 txqueuelen 10 (UNSPEC)
        RX packets 0 bytes 0 (0.0 B)
        RX errors 0 dropped 0 overruns 0 frame 0
        TX packets 0 bytes 0 (0.0 B)
        TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0

root@BeagleBone:~#
```

### Send data (loopback mode)
```
root@BeagleBone:~# ifconfig can0 down
root@BeagleBone:~# ip link set can0 up type can bitrate 500000 loopback on
[ 740.141630] IPv6: ADDRCONF(NETDEV_CHANGE): can0: link becomes ready
root@BeagleBone:~# candump can0 &
[1] 515
root@BeagleBone:~# cansend can0 123#12.34.56.78.90.11.12.13
  can0 123 [8] 12 34 56 78 90 11 12 13
  can0 123 [8] 12 34 56 78 90 11 12 13
root@BeagleBone:~# cansend can0 123#12.34.56.78.90.11.12.15
  can0 123 [8] 12 34 56 78 90 11 12 15
  can0 123 [8] 12 34 56 78 90 11 12 15
root@BeagleBone:~#
```

Hope you learn something helpfull from this post !
