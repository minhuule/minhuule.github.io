---
title: NFS Booting on BeagleBone Black
date: 2025-02-27 14:10:00 +0800
categories: [Linux Kernel]
tags: [linux kernel, beagle bone]
image: /assets/img/posts/nfs_booting_bbb/cover.png
---
Booting Beaglebone Black using NFS allows a machine to boot from an operating system stored on a network server rather than from uSD or EMMC.
## Requirement
Ubuntu 20.04 OS or Virtual Ubuntu 20.04 machine on Windows

## Setup network 
Verify ethernet port name, in my case it is enp0s8. 
* If you are using Virtual Machine, and your computer has ethernet physical port, you need to set up a network bridge adapter to your ethernet adapter
![setupethernet](/assets/img/posts/nfs_booting_bbb/ethernet.png)
* If you don't have physical ethernet, you can use usb ethernet instead
```
ifconfig
```
![Ethernet](/assets/img/posts/nfs_booting_bbb/ifconfig.png)

Config server network
```
sudo ifconfig enp0s8 192.168.4.16
```
> Sometimes you need to modify /etc/network/interfaces to set static for enp0s8*

## Install NFS server
> NFS server is used to store Rootfs

```
sudo apt-get install nfs-kernel-server
sudo /etc/init.d/nfs-kernel-server start # Start NFS server
```
Verify
```
service nfs-kernel-server status
```
![NFS](/assets/img/posts/nfs_booting_bbb/NFS_verify.png)

## Install TFTP server
> TFTP server is used to store Image and Device Tree*

```
sudo apt-get install tftpd-hpa
sudo ufw disable # Disable firewall
```
Verify
```
service tftpd-hpa status
```
![TFTP](/assets/img/posts/nfs_booting_bbb/TFTP_verify.png)

TFTP server located at **/srv/tftp**

## Setup image, device tree and rootfs

Download image file from [am335x-12-2-2023-10-07-4gb-microsd-iot](https://www.beagleboard.org/distros/am335x-12-2-2023-10-07-4gb-microsd-iot). Or you can use other versions on [distros](https://www.beagleboard.org/distros)

Extract image file to get .img file

Mount image file to folder, for example after mounting on Ubuntu, I get **rootfs** folder
> ![IMG](/assets/img/posts/nfs_booting_bbb/img.png)

Copy image & device tree to TFTP server (*Change kernel version if needed*)
```
cd <rootfs_folder_path>
sudo cp boot/vmlinuz-5.10.168-ti-r72 /srv/tftp/
sudo cp boot/dtbs/5.10.168-ti-r72/am335x-boneblack.dtb /srv/tftp/
sudo chmod 777 /srv/tftp/*
```
Copy rootfs to NFS server
```
sudo mkdir -p /nfs/bbb
sudo nano /etc/exports
```
> Add **/nfs/bbb *(rw,no_subtree_check,sync,no_root_squash)** at the end

![exports](/assets/img/posts/nfs_booting_bbb/exports.png)
```
sudo exportfs -a
showmount -e localhost # Verify NFS path
```
![showmount](/assets/img/posts/nfs_booting_bbb/showmount.png)
```
sudo cp -rf <rootfs_folder_path>/* /nfs/bbb/
sudo chmod 777 -R /nfs/bbb/ # Provide permission to all files/folders
```

## Boot on Windows using Teraterm

Install teraterm (Get from website or use **teraterm-5.0.exe**)

Connect BeagleBone UART cable

Open Teraterm -> choose your UART module

Select tab Setup -> Serial port -> change speed to 115200 -> New setting

Click Space after reset BeagleBone to access Uboot command
> ![uboot](/assets/img/posts/nfs_booting_bbb/uboot.png)

Select tab Control -> Macro -> point to **BBBTeratermBootScriptNFS.ttl**
> ![teraterm](/assets/img/posts/nfs_booting_bbb/bootteraterm.png)

## Boot on Ubuntu using minicom

Connect BeagleBone UART cable. If you are using Virtual Ubuntu Machine: select tab Devices -> USB -> choose your UART module
> ![minicomsetup](/assets/img/posts/nfs_booting_bbb/minicomsetup.png)

Connect UART module with any serial monitor on Ubuntu, for example Minicom

Click Space after reset BeagleBone to access Uboot command

Run each command in **BBBTeratermBootScriptNFS.ttl** in turn
```
1.  setenv gw_ip 192.168.4.1
2.  setenv server_ip 192.168.4.16
3.  setenv serverip 192.168.4.16
4.  setenv client_ip 192.168.4.25
5.  setenv ipaddr 192.168.4.25
6.  setenv root_dir '/nfs/bbb'
7.  setenv tftp_dir ''
8.  setenv bootfile vmlinuz-5.10.168-ti-r72
9.  setenv fdtfile am335x-boneblack.dtb
10. setenv nfsboot 'echo Booting from ${server_ip} ...; setenv nfsroot ${server_ip}:${root_dir}${nfs_options}; setenv ip ${client_ip}:${server_ip}:${gw_ip}:${netmask}::${device}:${autoconf}; setenv autoload no; setenv serverip ${server_ip}; setenv ipaddr ${client_ip}; tftp ${loadaddr} ${tftp_dir}${bootfile}; tftp ${fdtaddr} ${tftp_dir}/${fdtfile}; run nfsargs; bootz ${loadaddr} - ${fdtaddr}'
11. setenv bootcmd 'run nfsboot'
12. run nfsboot
```

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Login as root (username: `root`, password: `root`) to use sudo command
{: .prompt-tip }

> All the files mentioned in this post can be found in the GitHub: [NFS_Booting_BeagleBoneBlack](https://github.com/DucLee1509/NFS_Booting_BeagleBoneBlack)
{: .prompt-tip }
<!-- markdownlint-restore -->
