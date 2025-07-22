---
title: Build and Run AOSP 13 Emulator on Android Studio
date: 2024-11-25 19:28:00 +0800
categories: [Android Automotive]
tags: [aosp, raspberry pi 4, emulator, android studio]
image: /assets/img/posts/build_aosp13_emulator/cover.png
---
<div style="text-align: justify">
Do you want to grasp the knowledge of Android Automotive, but do not have an android device to boot and trial ? AOSP emulator is a good approach in this case. In this blog I will show you how to build and boot an AOSP version 13 emulator on Android Studio.
</div>

## Requirements
* Ubuntu 20.04
* 250 - 300 GB free disk space

## Install tools and packages
### 1. Repo
```
sudo apt update
sudo apt install curl python3
mkdir -p ~/bin
curl -o ~/bin/repo https://storage.googleapis.com/git-repo-downloads/repo
chmod a+x ~/bin/repo
echo 'export PATH=~/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```
Re-open terminal and confirm with:
```
repo --version
```

### 2. Packages
```
sudo apt-get install bison g++-multilib git gperf libxml2-utils make zlib1g-dev:i386 zip liblz4-tool libncurses5 libssl-dev bc flex curl python-is-python3 zlib1g-dev libelf-dev dwarves
```

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Above packages are nessesary packages, you may need to install more during building process.
{: .prompt-tip }
<!-- markdownlint-restore -->

## Build AOSP emulator image
Clone repo
```
repo init -u https://android.googlesource.com/platform/manifest -b android-13.0.0_r75 --depth=1
repo sync
```
* `repo init`: Download the manifest (an XML file describing what Git repositories your Android source tree is made of).
* `repo sync`: Read the manifest and clone all source codes.

```
source build/envsetup.sh
lunch sdk_car_x86-userdebug
export BUILD_EMULATOR_CLUSTER_DISPLAY=true
```
* `export BUILD_EMULATOR_CLUSTER_DISPLAY=true` is optional. This help building cluster screen.

```
make
make emu_img_zip
```

After building successfully, output will be stored at path like `out/target/product/emulator_x86/sdk-repo-linux-system-images-eng.labrnd.zip`

## Run AOSP emulator on Android Studio
Extract output **zip image** above to Android Studio emulator directory, usually stored at `C:\Users\<user-name>\AppData\Local\Android\Sdk\system-images\android-33\new_aaos_e
mulator\`
* `android-33`: for android 13, we need to use API level 33 (refer to [https://apilevels.com/](https://apilevels.com/))
* `new_aaos_emulator`: can be any name
* After extraction, you will have a directory inside `new_aaos_emulator`, in this case it is **x86**

Create package.xml in `C:\Users\<user-name>\AppData\Local\Android\Sdk\system-images\android-33\new_aaos_emulator\x86\package.xml` with the following content: [package.xml](https://gist.github.com/dpetrecki/76276a7efa1ce41333ac987f194cfaa7) and adjust its information like below:
![](/assets/img/posts/build_aosp13_emulator/packages.png)

In android studio, create device and map to the extracted image folder above:
* Device manager -> Add a new device -> Create Virtual Device
![](/assets/img/posts/build_aosp13_emulator/config1.png)

* Choose an automotive display -> Next
![](/assets/img/posts/build_aosp13_emulator/config2.png)

* On `x86 Images` tab, refresh to load your image, select it -> Next -> Finish
![](/assets/img/posts/build_aosp13_emulator/config3.png)

Now you can click start button or cold boot to run the emulator.
![](/assets/img/posts/build_aosp13_emulator/config4.png)

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Sometimes click start button will not show cluster screen, so I usually use Cold Boot.
{: .prompt-tip }
<!-- markdownlint-restore -->

## Result
Check out the result !
![](/assets/img/posts/build_aosp13_emulator/result.png)

Okay that's all. Give it a try. Thank you for your reading !

## Ref
* [https://developer.sony.com/open-source/aosp-on-xperia-open-devices/guides/aosp-build-instructions/build-aosp-android-13](https://developer.sony.com/open-source/aosp-on-xperia-open-devices/guides/aosp-build-instructions/build-aosp-android-13).
* [https://grapeup.com/blog/android-automotive-os-14-is-out-build-your-own-emulator-from-scratch](https://grapeup.com/blog/android-automotive-os-14-is-out-build-your-own-emulator-from-scratch)