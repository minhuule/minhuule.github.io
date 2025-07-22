---
title: Kconfig and Makefile - How to add a custom driver to Linux Kernel
date: 2024-10-11 7:35:00 +0800
categories: [Linux Kernel]
tags: [linux kernel, kconfig, makefile]
image: /assets/img/posts/kconfig_makefile_kernel/cover.png
---
<div style="text-align: justify">
How kernel manages all its kernel drivers, module drivers, features and susbsystems ? That’s Kconfig and Makefile. Here is the quick introduction about Kconfig and Makefile, how they are used in buiding kernel and managing kernel resources, especially drivers. Let’s get started !
</div>

## Kconfig
<div style="text-align: justify">
Kconfig stands for Kernel Configuration, is a configuration system used in the linux kernel to configure and manage kernel options. It provides a menu-driven interface for users  to select various features and options that determine how the kernel is built. These options control everything from device driver support to kernel features and subsystems, allowing users to tailor the kernel build to their specific needs or hardware requirements.
</div>

When executing command like `make menuconfig`, Kconfig at root level would be loaded. It includes other child Kconfigs from directories. All Kconfigs together will create an interface for users to edit. Here is an example of how an option looks like:

![](/assets/img/posts/kconfig_makefile_kernel/kconfig1.png)

* `tristate`: means that this option can have 3 possible values
  * `y` (yes): The feature will be built into the kernel.
  * `m` (module): The feature will be built as a loadable kernel module (.ko file), meaning it can be loaded or unloaded dynamically.
  * `n` (no): The feature will not be included in the kernel or as a module.
* `depends on`: dependencies configuration of this config. In above example, If condition `(ARCH_BRCMSTB && PHY_BRCM_USB) || COMPILE_TEST` is satisfied, config `USB_BRCMSTB` will follow user’s configuration. If the condition is not satisfied, config `USB_BRCMSTB`​ will always **no** even the users has this config enabled.
* `select`: changes other configs if needed. For example the first **select** line. If `USB_BRCMSTB` is enable (y or m) and `USB_OHCI_HCD` is enable, config `USB_OHCI_HCD_PLATFORM` will follow `USB_BRCMSTB`​.
* `help`: short description about the config.

Beside `tristate`, we have `bool` that specify the option can only have 2 possible values: **y (yes)** and **n (no)**

In some cases, config is enabled by user’s configuration in hardware config files or menuconfig, but the corresponding driver is not built because config’s condition is not satisfied. So that to confirm your configuration, please check `<kernel-source>/.config` file. For example:

```
cat .config | grep CONFIG_<name>
```

## Makefile
<div style="text-align: justify">
Makefile is a script that defines how to compile and link the various components of the kernel. It specifies which source files should be compiled, how they should be compiled, and how the final output (the kernel or kernel modules) should be created. For example, Makefile specifies which source file would be built if the config is enabled:
</div>

![](/assets/img/posts/kconfig_makefile_kernel/makefile1.png)

## Kernel Structure
<div style="text-align: justify">
The Linux kernel uses a complex hierarchy of Makefile and Kconfig scripts, with a top-level Makefile and Kconfig located at the root of the kernel source tree and many subdirectory Makefiles and Kconfigs that handle the compilation of different subsystems, drivers, and architecture-specific components. For example:
</div>

![](/assets/img/posts/kconfig_makefile_kernel/structure1.png)

## Add custom driver to linux kernel
Now I want to add driver `gpio_sysfs.c` to linux kernel as built-in or loadable kernel. Here are the steps:
* Create subdirectories and files like kernel structure at section 3.
* Kconfig:
  * Include gpio-sysfs Kconfig in drivers level Kconfig
  ![](/assets/img/posts/kconfig_makefile_kernel/driver1.png)
  * Add custom config: `CUSTOM_GPIO_SYSFS` to gpio-sysfs Kconfig
  ![](/assets/img/posts/kconfig_makefile_kernel/driver2.png)
* Makefile
  * Include gpio-sysfs Makefile in drivers level Makefile
  ![](/assets/img/posts/kconfig_makefile_kernel/driver3.png)
  * Add corresponding source file for the config in gpio-sysfs Makefile
  ![](/assets/img/posts/kconfig_makefile_kernel/driver4.png)

### Result
In menuconfig, if searching for `CONFIG_CUSTOM_GPIO_SYSFS`, we can see
![](/assets/img/posts/kconfig_makefile_kernel/result1.png)

Set `CONFIG_CUSTOM_GPIO_SYSFS=y`: driver built directly into kernel
![](/assets/img/posts/kconfig_makefile_kernel/result2.png)

Set `CONFIG_CUSTOM_GPIO_SYSFS=m`: driver built as loadable driver (.ko file). This file can be insmoded to kernel after boot.

![](/assets/img/posts/kconfig_makefile_kernel/result3.png)

That’s all. Hope you learn something useful from this article. Good bye and see you in other posts !