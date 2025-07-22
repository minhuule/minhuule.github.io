```
sudo apt update
sudo apt install gawk wget git-core diffstat unzip texinfo gcc-multilib \
     build-essential chrpath socat cpio python3 python3-pip python3-pexpect \
     xz-utils debianutils iputils-ping
```
```
git clone -b dunfell https://github.com/yoctoproject/poky.git
git clone -b dunfell git://git.openembedded.org/meta-openembedded
git clone -b dunfell https://git.yoctoproject.org/meta-raspberrypi
```
```
# Output image type
IMAGE_FSTYPES = "rpi-sdimg tar.gz"

# Reduce memory to build
BB_NUMBER_THREADS = "2"
PARALLEL_MAKE = "-j2"
```
```
BBLAYERS ?= " \
  ${TOPDIR}/../poky/meta \
  ${TOPDIR}/../poky/meta-poky \
  ${TOPDIR}/../poky/meta-yocto-bsp \
  ${TOPDIR}/../meta-raspberrypi \
  ${TOPDIR}/../meta-openembedded/meta-oe \
  ${TOPDIR}/../meta-openembedded/meta-networking \
  ${TOPDIR}/../meta-openembedded/meta-python \
  "
```

```
bitbake core-image-base
```

## Build Kernel

https://github.com/raspberrypi/linux/tree/rpi-5.4.y

```
sudo apt install gcc-aarch64-linux-gnu
```

```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2711_defconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-
```

Output:
* `arch/arm64/boot/Image`
* `arch/arm64/boot/dts/broadcom/bcm2711-rpi-4-b.dtb`