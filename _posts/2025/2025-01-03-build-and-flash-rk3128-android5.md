---
title: Build and Flash RK3128 Uboot, Kernel and Android 5.1
date: 2025-01-03 8:52:00 +0800
categories: [Android Automotive]
tags: [rk3128, aosp]
image: /assets/img/posts/build_rk3128_android5/cover.png
---
This blog show how to build u-boot, kernel, android (aosp), then flash image to RK3128 board using AndroidTool and finally boot it up.

## Working environment
Docker based on Ubuntu 20.04
```
root@6e70948f1141/workspace# uname -a
Linux 6e70948f1141 5.15.0-125-generic #135~20.04.1-Ubuntu SMP Mon Oct 7 13:56:22 UTC 2024 x86_64 x86_64 x86_64GNU/Linux
```

## Build RK3128 U-boot, Kernel, Android
### 1. Install packages
```
$ sudo apt-get update
$ sudo apt-get install -y curl wget nano git p7zip-full p7zip-rar bc
$ sudo apt-get install -y git-core gnupg flex bison build-essential zip curl zlib1g-dev libc6-dev-i386 libncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig repo
$ sudo apt-get install -y bc coreutils dosfstools e2fsprogs fdisk kpartx mtools ninja-build pkg-config python3-pip
$ sudo apt-get install -y rsync libpulse0 libnss3 libxcomposite-dev libxcursor-dev libxdamage-dev libxi-dev libxtstdev libasound2t64
$ sudo apt-get install -y lzop libbz2-dev gperf
```
**Note:**
* If you get error not found any package, just skip it first. Only find a workaround solution if the issue you meet in your build related to that package.
* These are just packages that I remember, maybe you need to install more if needed.

### 2. Install OpenJDK 7
Refer to [https://openbravotutorial.wordpress.com/2019/05/11/install-openjdk-7-on-linuxmint-19-1-or-ubuntu-18-04/](https://openbravotutorial.wordpress.com/2019/05/11/install-openjdk-7-on-linuxmint-19-1-or-ubuntu-18-04/)

Download **jdk-7u80-linux-x64.tar.gz** from [https://www.oracle.com/java/technologies/javase/javase7-archive-downloads.html](https://www.oracle.com/java/technologies/javase/javase7-archive-downloads.html)

```
$ cp jdk-7u80-linux-x64.tar.gz /usr/local/java/
$ cd /usr/local/java/ 
$ tar xvzf jdk-7u80-linux-x64.tar.gz 
```

Add the following line to the end of `/etc/profile`
```
JAVA_HOME=/usr/local/java/jdk1.7.0_80
JRE_HOME=/usr/local/java/jdk1.7.0_80
PATH=$PATH:$JRE_HOME/bin:$JAVA_HOME/binexport
export JAVA_HOME
export JRE_HOME
export PATH
```

Then execute the following commands:
```
$ sudo update-alternatives --install "/usr/bin/java" "java" "/usr/local/java/jdk1.7.0_80/bin/java" 1
$ sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/local/java/jdk1.7.0_80/bin/javac" 1
$ sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/local/java/jdk1.7.0_80/bin/javaws" 1
$ sudo update-alternatives --set java /usr/local/java/jdk1.7.0_80/bin/java
$ sudo update-alternatives --set javac /usr/local/java/jdk1.7.0_80/bin/javac
$ sudo update-alternatives --set javaws /usr/local/java/jdk1.7.0_80/bin/javaws
$ source /etc/profile 
```

Verify installation:
```
$ java -version
```
![](/assets/img/posts/build_rk3128_android5/install1.png)

### 3. Install Python2
Try the following commands:
```
$ sudo apt-get update
$ sudo apt-get install python
```

If it install python2 as **python** (Check by command `python –version`)=> OK

If it show error like below => Have to install python2 manually.
![](/assets/img/posts/build_rk3128_android5/install2.png)

Refer to [Download, install, and build Python 2.7.10 in a local folder](https://gist.github.com/ckandoth/f25c7469f23e63e34bee346fcb10ec29)
```
$ export PYTHON_BASE="/workspace/python"
$ export PYTHON_PREFIX=$PYTHON_BASE/python-2.7.10
$ mkdir -p $PYTHON_BASE
$ cd $PYTHON_BASE
$ curl --create-dirs -L -o src/Python-2.7.10.tgz https://www.python.org/ftp/python/2.7.10/Python-2.7.10.tgz
$ cd src
$ tar -zxf Python-2.7.10.tgz
$ cd Python-2.7.10
$ ./configure --prefix=$PYTHON_PREFIX --enable-shared --enable-unicode=ucs4 LDFLAGS="-Wl,-rpath=$PYTHON_PREFIX/lib"
$ make
$ make install
```

**Note**: must export PATH link to python bin file after build
```
$ export PATH=$PATH:/workspace/python/python-2.7.10/bin
```

### 4. Download Android SDK
Download RK3128 Android 5.1 SDK from [https://en.t-firefly.com/doc/download/6.html#other_35](https://en.t-firefly.com/doc/download/6.html#other_35)

```
$ mkdir -p /workspace/RK3128/AOSP
$ cd /workspace/RK3128/AOSP
$ 7z x <path to firefly_rk3288_rk3128_android5.1_git_20211216.7z.001> -r -o. 
```
![](/assets/img/posts/build_rk3128_android5/download1.png)

```
$ git reset --hard
$ git checkout fireprime
$ git pull gitlab fireprime:fireprime 
```
![](/assets/img/posts/build_rk3128_android5/download2.png)

**Note**: Above output is from normal Ubuntu, output from docker is the same.

### 5. Compile
#### 5.1. U-boot
```
$ cd /workspace/RK3128/AOSP
$ cd u-boot/
$ make rk3128_defconfig
$ make -j8 
```
![](/assets/img/posts/build_rk3128_android5/uboot.png)

#### 5.2. Kernel
```
$ cd /workspace/RK3128/AOSP2 cd kernel/
$ ls arch/arm/configs
$ make firefly_defconfig
$ make -j8 rk3128-fireprime.img 
```

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Issue when compiling kernel
{: .prompt-tip }
<!-- markdownlint-restore -->

**Issue 1**
![](/assets/img/posts/build_rk3128_android5/kernel1.png)

Fix:
```
$ cd /workspace/RK3128/AOSP/kernel
$ sed -i 's/YYLTYPE yylloc/extern YYLTYPE yylloc/g' scripts/dtc/dtc-lexer.lex.c
$ sed -i 's/YYLTYPE yylloc/extern YYLTYPE yylloc/g' scripts/dtc/dtc-lexer.lex.c_shipped
```

#### 5.3. Android
```
$ cd /workspace/RK3128/AOSP
$ lunch rk312x-userdebug
$ make -j16 
```
![](/assets/img/posts/build_rk3128_android5/android1.png)

```
$ ./mkimage.sh
```
![](/assets/img/posts/build_rk3128_android5/android2.png)

```
$ ./FFTools/mkupdate/mkupdate.sh -l rk312x-userdebug 
```
![](/assets/img/posts/build_rk3128_android5/android3.png)

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Issue when compiling android
{: .prompt-tip }
<!-- markdownlint-restore -->

**Issue 1**
![](/assets/img/posts/build_rk3128_android5/android4.png)

Fix:
```
$ export LC_ALL=C
```

**Issue 2**
![](/assets/img/posts/build_rk3128_android5/android5.png)

Fix: refer to [https://stackoverflow.com/questions/36048358/building-android-from-sources-unsupported-reloc-43](https://stackoverflow.com/questions/36048358/building-android-from-sources-unsupported-reloc-43), add the following line to `build/core/clang/HOST_x86_common.mk`
```
-B$($(clang_2nd_arch_prefix)HOST_TOOLCHAIN_FOR_CLANG)/x86_64-linux/bin
```
![](/assets/img/posts/build_rk3128_android5/android6.png)

Then:
```
$ make clean
$ make -j16
```

**Issue 3**
![](/assets/img/posts/build_rk3128_android5/android7.png)

Fix:
```
$ make update-api
$ make -j16
```

**Issue 4**
![](/assets/img/posts/build_rk3128_android5/android8.png)

Fix: refer to [https://blog.csdn.net/qiushan_/article/details/125022672](https://blog.csdn.net/qiushan_/article/details/125022672)
```
$ rm -rf out/target/product/rk312x/obj/GYP/shared_intermediates/blink/core
```

Then comment the following line in `external/chromium_org/third_party/WebKit/Source/build/scripts/rule_bison.py`
```
=> comment line os.unlink(outputHTmp) 
```

**Other issues**: refer to [https://blog.csdn.net/qq_33750826/article/details/133344344](https://blog.csdn.net/qq_33750826/article/details/133344344)

## Flash image to RK3128 board
### 1. Environment
**OS**: Windows 11

**DriverAssistant**: version 5.1.1

**AndroidTool (RKDevtool)**: version 2.93

### 2. Install DriverAssistant and AndroidTool on your PC
Download **RKDevTool_Release_v2.93.zip** from [RKDevTool_Release_v2.93.zip](https://drive.google.com/file/d/118HsE-N5YvqLfDAE8c9noVY0qWIsGhR2/view)

Extract zip file

#### 2.1. DriverAssistant
Move to folder **DriverAssitant_v5.1.1 → DriverInstall.exe → Install Driver**
![](/assets/img/posts/build_rk3128_android5/flash1.png)

#### 2.2. RKDevTool
Move to folder **RKDevTool_Release_v2.93 → RKDevTool.exe**
![](/assets/img/posts/build_rk3128_android5/flash2.png)

### 3. Enter MASKROM mode of RK3128
For more information about MASKROM mode, refer to [https://wiki.t-firefly.com/en/Firefly-RK3128/MaskRom_Mode.html](https://wiki.t-firefly.com/en/Firefly-RK3128/MaskRom_Mode.html)

Here are steps to enter MASKROM mode:
* Connect mini usb of RK3128 to PC.
* Use metal tweezers connect CLK and GND or the board.
![](/assets/img/posts/build_rk3128_android5/flash3.png)

* Click RESET button.
* Then when you see that RKDevTool recognize the board as MASKROM device → release metal tweezers.
![](/assets/img/posts/build_rk3128_android5/flash4.png)

Now the board is in MASKROM mode and be able to be flashed.

### 4. Flash RK3128 image
Use your output image above or my prebuilt image [Image-rk3128](https://drive.google.com/drive/folders/14cechxqP3BLGchugJuXWe8u8ZAJl35oj)

There are 2 ways to load RK3128 image:

#### 4.1. Flash the total .img
The total .img is **Fireprime_Android5.1.1_Public_241114.img**

On RKDevTool: **Upgrade Firmware → Firmware (choose to image file) → Upgrade**

![](/assets/img/posts/build_rk3128_android5/flash5.png)

Above is the result when flash successfully.

#### 4.2. Flash separate .img
There are many images and related files beside to total .img
![](/assets/img/posts/build_rk3128_android5/flash6.png)

In **parameter.txt**, there is some information about address partition for each image. I based on that information to fill in RKDevTool.
![](/assets/img/posts/build_rk3128_android5/flash7.png)

Check all available images and files like image above → Now boot.

## Boot RK3128 board
After successfully flashing image to RK3128 board, the board will automatically reboot several times, then boot to Android 5.1. On the first boot, it takes about 3-5 minutes, so be patient. From the second boot onwards it only takes about 20 seconds.

![](/assets/img/posts/build_rk3128_android5/boot.png)

OK that’s all, hope you learn something from my blog !