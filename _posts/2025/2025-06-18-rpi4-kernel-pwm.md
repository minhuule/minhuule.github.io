---
title: Raspberry Pi - PWM Pulse (no dtoverlay)
date: 2025-06-18 19:35:00 +0800
categories: [Linux Kernel]
tags: [linux kernel, raspberry pi, pwm]
image: /assets/img/posts/rpi_pwm/cover.png
---
<div style="text-align: justify">
Working with PWM (Pulse Width Modulation) on the Raspberry Pi usually involves enabling overlays through config.txt. But what if you want to explore PWM directly from the kernel side, without relying on dtoverlay ? In this post, I’ll walk through how to set up and control PWM pulses at a lower level. Now let's get started!
</div>

## PWM on RPi4

### 1. BCM2711

I am using Raspberry Pi 4B, which using Broadcom chip BCM2711. To understand the PWM feature supported on this chip, take a look at its datasheet: [bcm2711-peripherals.pdf](https://datasheets.raspberrypi.com/bcm2711/bcm2711-peripherals.pdf)

BCM2711 has 4 PWM channels 0 and 1, each channel has 2 sub channels. Section **5.3 Alternative Function Assignments** or **8.5** shows how these channels are mapped to the GPIOs.

![](/assets/img/posts/rpi_pwm/function.png)

For example **PWM channel 0 subchannel 1** is used as function `alt0` of `GPIO13` and `GPIO45`, or function `alt5` of `GPIO19`.

So which channel should we use ?

### 2. RPi4

For a more complete view, let's see how these channels show up on the board.

![](/assets/img/posts/rpi_pwm/hardware.png)

We can see that only PWM channel 0 is output to the pin on RPi4. I will choose the following pins:
* PWM channel 0 sub channel 0: `GPIO12`
* PWM channel 0 sub channel 1: `GPIO13`

## Kernel Code

PWM channel 0 is defined as `pwm` in `arch/arm/boot/dts/bcm283x.dtsi`

![](/assets/img/posts/rpi_pwm/bcm283x.png)

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> To be sure, check its base address below.
{: .prompt-tip }
<!-- markdownlint-restore -->

![](/assets/img/posts/rpi_pwm/address.png)

All PWM pins are defined in `arch/arm/boot/dts/bcm2711.dtsi`

![](/assets/img/posts/rpi_pwm/pin.png)

Now only one part is missing, enable node pwm. I will added it in `arch/arm/boot/dts/bcm2711-rpi-4-b.dts`

![](/assets/img/posts/rpi_pwm/enable.png)

Now let's build the kernel and boot the Raspberry Pi.

## Verify

First check the PWM pins function and available PWM
```
root@raspberrypi4-64:~# cat /sys/kernel/debug/pinctrl/fe200000.gpio-pinctrl-bcm2835/pinmux-pins | grep pwm
pin 12 (gpio12): fe20c000.pwm (GPIO UNCLAIMED) function alt0 group gpio12
pin 13 (gpio13): fe20c000.pwm (GPIO UNCLAIMED) function alt0 group gpio13
root@raspberrypi4-64:~#
```
```
root@raspberrypi4-64:~# cat /sys/kernel/debug/pwm
platform/fe20c000.pwm, 2 PWM devices
 pwm-0   ((null)              ): period: 0 ns duty: 0 ns polarity: normal
 pwm-1   ((null)              ): period: 0 ns duty: 0 ns polarity: normal
root@raspberrypi4-64:~#
```

Okay let's turn it on
```
root@raspberrypi4-64:~# echo 0 > /sys/class/pwm/pwmchip0/export
root@raspberrypi4-64:~# echo 10000 > /sys/class/pwm/pwmchip0/pwm0/period
root@raspberrypi4-64:~# echo 4000 > /sys/class/pwm/pwmchip0/pwm0/duty_cycle
root@raspberrypi4-64:~#
root@raspberrypi4-64:~# echo 1 > /sys/class/pwm/pwmchip0/export
root@raspberrypi4-64:~# echo 20000 > /sys/class/pwm/pwmchip0/pwm1/period
root@raspberrypi4-64:~# echo 15000 > /sys/class/pwm/pwmchip0/pwm1/duty_cycle
root@raspberrypi4-64:~#
root@raspberrypi4-64:~# echo 1 > /sys/class/pwm/pwmchip0/pwm0/enable
root@raspberrypi4-64:~# echo 1 > /sys/class/pwm/pwmchip0/pwm1/enable
root@raspberrypi4-64:~#
```
![](/assets/img/posts/rpi_pwm/pulse.png)

Inverse the pulse

```
root@raspberrypi4-64:~# echo inversed > /sys/class/pwm/pwmchip0/pwm0/polarity
root@raspberrypi4-64:~# echo inversed > /sys/class/pwm/pwmchip0/pwm1/polarity
root@raspberrypi4-64:~#
```

![](/assets/img/posts/rpi_pwm/pulse_inversed.png)

<div style="text-align: justify">
That's all about how to trigger PWM on kernel side of Raspberry Pi 4. Hope you learn something helpful from my blog. See you!!
</div>
