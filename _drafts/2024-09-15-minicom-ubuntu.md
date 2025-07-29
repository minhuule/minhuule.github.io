---
title: How I use MINICOM to interact with board in Ubuntu
date: 2025-09-15 18:24:00 +0800
categories: [Tools]
tags: [minicom, vscode]
image: /assets/img/posts/minicom/cover.png
---
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
> Reboot the machine to apply the changes.

Connecting serial device to Ubuntu and confirm by the following commands:
```
groups

ls -l /dev/ttyUSB*
```
![](/assets/img/posts/minicom/confirm.png)

> **/dev/ttyUSB*** now have read/write permission automatically.

