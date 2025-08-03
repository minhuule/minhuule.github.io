---
title: How I use MINICOM to interact with linux board on Ubuntu
date: 2025-02-23 18:24:00 +0800
categories: [Tools]
tags: [minicom, vscode]
image: /assets/img/posts/minicom/cover.png
---
<div style="text-align: justify">
When working with linux boards, a reliable serial communication tool is essential. Minicom is a simple, lightweight, and powerful terminal emulator that runs in the command line on Linux systems. In this post, I’ll walk through how I use Minicom on Ubuntu to communicate with board effectively, including interacting, running scripts and viewing logs. Checkout my output:
</div>

{% include embed/youtube.html id='QspWPlQ91UM' %}

## Install and setup minicom
### 1. Install

```
sudo apt-get install minicom
```
![](/assets/img/posts/minicom/install.png)

### 2. Setup

```
sudo minicom –s
```
![](/assets/img/posts/minicom/setup.png)
Make sure you have above setup:
1. Press `a` to change default serial device -> Enter
2. Press `f` to change Hardware Flow Control to No. (This allow you to interact with board serial)
3. Enter -> Save setup as dfl -> Enter -> Exit from Minicom

### 3. Allow access /dev/ttyUSB* without "chmod" or "sudo"

This make sure that /dev/ttyUSB* automatically accessed by normal user, without `root`, `sudo` or `chmod` needed.

```
sudo usermod -aG dialout $USER
```

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Reboot the machine to apply the changes.
{: .prompt-tip }
<!-- markdownlint-restore -->

Connecting serial device to Ubuntu and confirm by the following commands:
```
groups

ls -l /dev/ttyUSB*
```
![](/assets/img/posts/minicom/confirm.png)

> **/dev/ttyUSB*** now have read/write permission automatically.

### 4. Connect serial device to minicom

```
minicom –D /dev/ttyUSB0
```

Or just use `minicom` for default serial device

Enter Uboot console of your device to confirm the interaction between board and minicom

![](/assets/img/posts/minicom/uboot.png)

Leave minicom by command: `Ctrl+A`, then `Z`, then `Q`, then `Yes`

## Run scripts in minicom

### 1. Create script

Minicom file name: `*.mscr`

Create a new script or modify a ttl (teraterm format) with the following syntax:
1. Replace `wait` with `expect`
2. Replace `sendln` with `send`
3. Add `\r` to the last command
4. Make sure there is a line expect `=>` at the beginning.

Here is an example of a NFS boot script for beaglebone:

![](/assets/img/posts/minicom/script.png)

### 2. Run script

Here I use NFS beagle bone boot script to demonstrate. There are 2 methods:

* Method 1: In ubuntu terminal: `minicom –S <path-to-.mscr-script>`. Enter board **U-boot** then hit **Enter** to trigger script.

* Method 2: In minicom session: `Ctrl+A`, then `Z`, then `G`. Press `c`, enter script name in the same folder with the folder you are running minicom command, then hit **Enter** a few times to run.

![](/assets/img/posts/minicom/boot.png)

## Log in minicom

Sometimes we need log to store output console. There are 2 methods:

* Method 1: In terminal: `minicom –C <path-to-log-file>`
  * For example: `minicom -C my_log.txt`

* Method 2: In minicom session: `Ctrl+A`, then `L`. Enter the log file name.

![](/assets/img/posts/minicom/log.png)

## Final Result

Combine all in vscode for convenient development.

![](/assets/img/posts/minicom/result.png)

Okay that's all. Hope it helps your kernel development. Thank you!
