## Setting up dev environment using an older Ubuntu image and qemu

You need a kernel, initrd and rootfs(create):



Download kernel and initrd from netboot section of 14.04:
https://ports.ubuntu.com/ubuntu-ports/dists/trusty-updates/main/installer-armhf/current/images/generic/netboot/

```
wget https://ports.ubuntu.com/ubuntu-ports/dists/trusty-updates/main/installer-armhf/current/images/generic/netboot/vmlinuz
wget https://ports.ubuntu.com/ubuntu-ports/dists/trusty-updates/main/installer-armhf/current/images/generic/netboot/initrd.gz
```

Or the Wandaboard-specific kernel:
https://ports.ubuntu.com/ubuntu-ports/dists/trusty-updates/main/installer-armhf/current/images/generic/netboot/wandboard-quad/uImage



Download root image:
https://cloud-images.ubuntu.com/releases/trusty/release/ubuntu-14.04-server-cloudimg-armhf-disk1.img








dumpimage -T kernel -p 0 uImage -o kernel



Download a tar image from:
https://cdimage.ubuntu.com/ubuntu-base/releases/14.04/release/

Create a disk image with an ext4 partition:

```
dd if=/dev/zero of=ubuntu14.img count=256 bs=1M
fdisk ./ubuntu14.img # Create one partition of type 83 all defaults
losetup -f -P ubuntu14.img
losetup -a # Displays the mapped lo device
mkfs.ext4 /dev/loop0p1
mount /dev/loop0p1 /mnt
cd /mnt
tar xvfz ~/ubuntu-base-14.04.6-base-armhf.tar.gz
cd ~
umount /mnt
```



Boot using qemu:
```
qemu-system-arm  -kernel vmlinuz -initrd initrd.gz -hda ubuntu14.img -append "root=/dev/hda"
qemu-system-arm -m 512M -M sabrelite  -kernel kernel -initrd initrd.gz -hda ubuntu14.img -append "root=/dev/ram console=ttymxc0,115200"
```

Download gcc-4.9 (needed for kernel 2.6.32)

Cross-compile the kernel














## Mirrors


http://apt-repo.cs.uchicago.edu/ubuntu-ports/dists/trusty/main/binary-armhf/ (7.8MB)
https://ftp.osuosl.org/pub/ros-ubuntu/dists/trusty/main/binary-armhf/ (2.1MB)
http://packages.ros.org/ros/ubuntu/dists/trusty/main/binary-armhf/ (2.1MB)

1.2MB mirrors:
https://jp.mirror.coganng.com/ubuntu-ports/dists/trusty/main/binary-armhf/
https://mirror.coganng.com/ubuntu-ports/dists/trusty/main/binary-armhf/
https://mirrors.ocf.berkeley.edu/ubuntu-ports/dists/trusty/main/binary-armhf/
https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/dists/trusty/main/binary-armhf/

