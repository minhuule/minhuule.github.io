---
title: BeagleBone Black - I2C LCD1602 Console Display
date: 2025-04-10 18:27:00 +0800
categories: [Linux Kernel]
tags: [linux kernel, beagle bone, lcd1602, I2C]
image: /assets/img/posts/lcd1602_i2c_bbb/cover.png
---
<div style="text-align: justify">
Controlling an LCD1602 display using the BeagleBone and I2C protocol is a powerful way to create a flexible and efficient display system for embedded projects. By leveraging the I2C-LCD module and a custom kernel driver, you can simplify the connection and control of the LCD1602, reducing the number of GPIO pins required and making the setup more scalable. In this article, we’ll explore how to set up and control the LCD1602 display from a BeagleBone running Linux, using the I2C protocol.
</div>

## Target
<div style="text-align: justify">
In this example, I want to use Beagle Bone Black (BBB) to control LCD1602 through I2C-LCD module and use it as a console that display any characters I type on keyboard. Let’s take a look at the result:
</div>

{% include embed/youtube.html id='ap59oXkuTcQ' %}

## Physical Connection
### 1. LCD1602
<div style="text-align: justify">
The LCD1602 is a 16×2 character Liquid Crystal Display (LCD) module commonly used in microcontroller-based projects. It features 16 columns and 2 rows, allowing it to display up to 32 characters at a time. The module uses the Hitachi HD44780 controller, which supports both 8-bit and 4-bit modes for data communication.
</div>
To understand how it works, operating modes and communicates with this LCD, take a look at its datasheet: [HD44780.pdf](https://cdn.sparkfun.com/assets/9/5/f/7/b/HD44780.pdf)

### 2. Connect
<div style="text-align: justify">
Shared I2C bus of BeagleBone will be used to interact with module I2C-LCD. BeagleBone provide 2 I2C buses: I2C1 & I2C2. In this article I will try both of these I2C buses. I2C0 pins are not connected to the header so it is difficult to use. This image demonstrate how BeagleBone communicate with LCD1602 through I2C-LCD module:
</div>

![](/assets/img/posts/lcd1602_i2c_bbb/connect.png)

### 3. Default address
Module I2C-LCD is controlled by IC PCF8574. Its address depends on pins A0, A1 and A2.

![](/assets/img/posts/lcd1602_i2c_bbb/address.png)

Default address of the module is 0x27, but can be changed by sodering pins A0, A1 and A2.

![](/assets/img/posts/lcd1602_i2c_bbb/address2.png)

## Detect I2C Devices
Beagle Bone Black has 3 I2C buses (0, 1 and 2). Here is the command to detect all I2C devices of a bus:
```
i2cdetect –y –r <i2c bus number>
```
For example to detect I2C0:
```
i2cdetect –y –r 0
```
![](/assets/img/posts/lcd1602_i2c_bbb/detect.png)

<div style="text-align: justify">
You can see that we have 4 devices detected. Their addresses are 0x24, 0x34, 0x50, 0x70. The addresses 0x24, 0x50, 0x70 are represented by the letter “UU” on the table. That means these hardware has been probed and taken over by the driver. Hardware with address 0x34 is represented by its own address on the board, meaning it is still available.
</div>

Now I connect pins of I2C-LCD module to I2C2 of BBB.
```
VCC ===> SYS 5V (P9_7)
GND ===> GND (P9_1)
SCL ===> I2C2_SCL (P9_19)
SDA ===> I2C2_SDA (P9_20)
```

After booting, make sure our module is detected:
```
i2cdetect -y -r 2
```
![](/assets/img/posts/lcd1602_i2c_bbb/detect2.png)

## Device Tree
<div style="text-align: justify">
As I mentioned above, I2C0 does not have pins, so no external pin can be connected to this I2C bus through the header. We can only use I2C1 or I2C2. In this article I will try I2C2 by changing the device tree of Beagle Bone Black.
</div>

Beagle Bone Black’s I2C nodes are defined at `am335x-bone-common.dtsi`

First is the pin configuration. Based on the system reference manual of BBB, we can see that I2C2 is mux mode 3 of header P9, pins 19 and 20.

![](/assets/img/posts/lcd1602_i2c_bbb/pin.png)

In the dtsi, I2C2 pins have been initialized with MUX_MODE3, so it can be understood that I2C2 pins have been initialized according to its function as I2C, we do not need to affect this config.

![](/assets/img/posts/lcd1602_i2c_bbb/pin2.png)

To add an additional node for the LCD, simply find the i2c2 node in the device tree and add it:

![](/assets/img/posts/lcd1602_i2c_bbb/pin3.png)

<div style="text-align: justify">
In this case, we are using BBB as the master, which can control multiple slaves at the same time on the same I2C bus. Just make sure that hadware on the same bus does not have the same address. In I2C2 node, by default it has defined many eeproms with addresses 0x54, 0x55, 0x56 and 0x57. The I2C-LCD module with address 0x27 can be added without having to change its address.
</div>

## Device Driver
My kernel driver is used to communicate between user and device. So that I define a code structure like this:

![](/assets/img/posts/lcd1602_i2c_bbb/driver.png)

About device communication, I need to define I2C interfaces to write command to I2C bus, then use the interfaces to define LCD1602 commands to control the LCD.

<div style="text-align: justify">
About user communication, I define IOCTL interfaces to execute specific commands from user when interacting with the LCD as a console display like backspace, clear screen, enter to a new line, …
</div>

I also have READ method to print out text on the LCD when executing cat `/dev/<device_file_name>` and WRITE method to display text to the LCD when `echo “some text” > /dev/<device_file_name>`.

### 1. Device Communication
#### 1.1. I2C API
To read/write data to I2C buses on linux OS, there are two common interfaces: **Plain I2C** and **SMBus**.

<div style="text-align: justify">
Plain i2c implements the i2c protocol at a general level, while SMBus is essentially a subset of the I2C protocol, which means that it uses the same physical layer and basic communication principles as I2C. However, SMBus introduces additional features and constraints to standardize communication for specific use cases.
</div>

Based on [https://www.kernel.org/doc/Documentation/i2c/writing-clients](https://www.kernel.org/doc/Documentation/i2c/writing-clients)
> If you can choose between plain I2C communication and SMBus level
communication, please use the latter. All adapters understand SMBus level commands, but only some of them understand plain I2C!

In my source code `lcd1602_drv.h`, I have a macro USE_SMBUS to choose between Plain I2C and SMBus:
```
#define USE_SMBUS

static int i2c_master_write_byte(struct i2c_client *client, u8 value) {
    #ifdef USE_SMBUS // Use SMBus API
        return i2c_smbus_write_byte(client, value);
    #else // Use plain I2C API
        u8 buf[1];
        buf[0] = value;
        return i2c_master_send(client, buf, sizeof(buf));
    #endif
}
```

#### 1.2. LCD1602 Command
To implement commands for controlling LCD1602, I use mainly rely on the following document and source code:
* [HD44780.pdf](https://www.sparkfun.com/datasheets/LCD/HD44780.pdf)
* [https://github.com/ayleenw/lcd_i2c_stm32](https://github.com/ayleenw/lcd_i2c_stm32)
* [Arduino-LiquidCrystal-I2C-library](https://github.com/fdebrabander/Arduino-LiquidCrystal-I2C-library)

In my code, I use 4-bit mode interface to communicate.

There are two type of interfaces: command interface and data interface:
* Command interface: send command to LCD: lcd1602_command . I create some APIs: lcd1602_init, lcd1602_home, lcd1602_clear, …
* Data interface: print character on LCD: lcd1602_data . I implement lcd1602_print to print a string on LCD.

### 2. User Communication
For user communication, I simple use struct file_operations to create method IOCTL, READ, WRITE, …
```
static struct file_operations fops = 
{ 
    .owner = THIS_MODULE, 
    .read = lcd1602_read, 
    .write = lcd1602_write, 
    .open = lcd1602_open, 
    .unlocked_ioctl = lcd1602_ioctl, 
    .release = lcd1602_release, 
};
```

Then I create a device file `/dev/lcd1602_device` to interact with kernel in user space
```
/*Creating device file*/ 
if(IS_ERR(device_create(dev_class,NULL,dev,NULL,"lcd1602_device"))){ 
    pr_err("Cannot create the Device file\n"); 
    goto r_device; 
}
```

#### 2.1. IOCTL
I use IOCTL method to implement user command: enter single character, enter, clear screen, backspace.

In kernel space:

![](/assets/img/posts/lcd1602_i2c_bbb/driver2.png)
![](/assets/img/posts/lcd1602_i2c_bbb/driver3.png)

In user space, I create a C application that use ioctl API from `#include <sys/ioctl.h>`

![](/assets/img/posts/lcd1602_i2c_bbb/driver4.png)

#### 2.2. READ
`cat /dev/lcd1602_device`: Read and print current string on LCD to terminal

![](/assets/img/posts/lcd1602_i2c_bbb/driver5.png)

#### 2.3. WRITE
`echo “Welcome” > /dev/lcd1602_device`: Print Welcome to LCD

![](/assets/img/posts/lcd1602_i2c_bbb/driver6.png)

## Demo
![](/assets/img/posts/lcd1602_i2c_bbb/demo1.png)

{% include embed/youtube.html id='ap59oXkuTcQ' %}