# How to build a dev box for really old kernels

## Reasons

- For old chips like at91sam9260 we need armel/armv4/v5 kernel and distributions
- The Linux Device Drivers ed3 book uses 2.6.10.
- VMware Fusion does not run x86 distros
- We need an older distro with native gcc4 to compile kernel 2.6.x versions and maybe older
- Ideally it would not cross-compile; the whole distribution should be armel
- Optionally if we can find a binary toolchain of gcc 4.9 but compiled as armhf or better yet aarch64 that would work too with a faster VM on the macbook M2. Havent found one yet, Linaro toolchains from that timeframe are all x86/x64
- Most qemu targets fail for various reasons, even the raspberry pi, imx6ul and orange pi ones.
- Only versatilepb works well and boots well, but comes with cpu type arm1176, this works with older Raspbian images though


## Targets

- Debian Wheezy (kernel 3.x) or Squeeze (2.6.32)
- RHEL 6.0 (Can't find an armel image online)
- Ubuntu 14.04
Only armhf/armv7 is available: https://cloud-images.ubuntu.com/releases/trusty/release/
- Slackware 14.1 (Still armhf)


Amonst the 3, getting and building a Raspbian image from 2015 based on Wheezy was the easiest:
https://downloads.raspberrypi.org/raspbian/archive/2015-09-28-23:47/


We need kernel and devicetree that works with versatilepb machine.
Maybe one day the raspi machine will work and we can just use its kernel
https://github.com/dhruvvyas90/qemu-rpi-kernel



## Steps to build

- Download boot.tar.xz and root.tar.xz
- Create a 4GB image, MBR partition, first partition type 0xc size 60MB and second type 83 and ext4
- Extract the above files in the two partitions
- Edit /etc/fstab, change partitions to sda1 and sda2
- Edit /ectc/default/keyboard and change default keyboard layout from gb to us
- Newer distros add a feature in the ext4 filesystem that must be disabled for these older distros:
`tune2fs -O ^metadata_csum`


## Boot with qemu on a Macbook M2

```
qemu-system-aarch64 -cpu arm1176 -m 256M -M versatilepb -kernel kernel-qemu-4.14.79-stretch -dtb versatile-pb.dtb -hda rasbian.img -append 'console=ttyAMA0,115200 console=tty1 root=/dev/sda2 rootfstype=ext4 elevator=deadline rootwait' -serial stdio
```

- Kernel and dtb came from the qemu-rpi-kernel repo above






Linux4SAM has their own version:
https://github.com/linux4sam/linux-at91/tree/linux-2.6.39-at91
