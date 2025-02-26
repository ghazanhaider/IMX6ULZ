# Using a Raspbian image

The Raspbian armhf image is nicely packaged and very usable on armhf boards.
It works great for my SAMA5D21 and iMX6 boards.
It comes in ext4 format so it likely isn't great for unmanaged NAND chips or SD cards. Use a buildroot image instead or try to migrate the contents to a UBIFS image.
It works for eMMC (tested here) and SSD drives mounted through USB

The goal is to have a functioning debian image to run on these boards and be able to compile packages like the linux kernel directly on the board.
The lite image takes up around 2GB and can compile the latest kernel on a 4GB eMMC storage.

We also load the kernel and devicetree from /boot, in the Raspberry PI style to make kernel updates easier. Therefore only the u-boot.imx image needs to be outside of this filesystem.



Download *2024-11-19-raspios-bookworm-armhf-lite.img.gz* from the Raspbian site.

Unzip, mount as loopback device and extract the ext4 partition to another file, called raspios.img here

MacOS:
```
hdiutil attach -imagekey diskimage-class=CRawDiskImage -nomount 2024-11-19-raspios-bookworm-armhf-lite.img
dd if=/dev/disk4s2 of=raspios.img bs=1M
hdiutil detach /dev/disk4
```

Linux (untested):
```
losetup -Pf 2024-11-19-raspios-bookworm-armhf-lite.img
dd if=/dev/loop0p2 of=raspios.img bs=1M
losetup -D
```


## Fixup image contents

Mount this partition image in a linux host:
`losetup -fP raspios.img`
`mount /dev/loop0 /mnt/1`


## List of changes to make

A scripted way of doing all of these is at the end

### Allow pi user to login

Remove pi password in /etc/shadow:
`pi::20130:0:99999:7:::`

### USB Gadget

Copy any kernel modules to /lib/modules and ensure g_multi.ko exists in there
Add `g_multi` to /etc/modules
Add `options g_multi iSerialNumber=123456 file=/file.img removable=y ro=0 stall=0` to /etc/modprobe.d/g_multi.conf
Add a 1MB file /file.img: `dd if=/dev/zero of=/file.img count=1k bs=1k`
Create a getty link to ttyGS0:
```
mkdir /etc/systemd/system/getty.target.wants
cd /etc/systemd/system/getty.target.wants
ln -s /lib/systemd/system/serial-getty@.service serial-getty@ttyGS0.service
```

### ATWILC1000 firmware

Download firmware files: https://github.com/linux4wilc/firmware
Extract the `atmel` folder and place in /lib/firmware/


### Scripted steps

Needs
- Mounted partition in /mnt/1
- Buildroot with kernel and modules created as IMXDIR = /imx6ulz
- Map this repo to: IMX_GIT = ~/IMX6ULZ
- FW_DIR = ~/linux-firmware-20250211


```
# Directories:
IMXDIR=/imx6ulz
IMX_GIT=~/IMX6ULZ
FW_DIR=~/linux-firmware-20250211

# Other variables:
GADGET_SN=33333
HOSTNAME=imx6ulz3

# Main code:
sed -i "s/^pi:\!:/pi::/" /mnt/1/etc/shadow
echo -e "proc /proc proc defaults 0 0\nUUID=bb15c8e6-d999-4838-be67-5ff200bffa46 / ext4 defaults,noatime 0 1" > /mnt/1/etc/fstab
echo "g_multi" >> /mnt/1/etc/modules
echo "options g_multi iSerialNumber=${GADGET_SN} file=/file.img removable=y ro=0 stall=0 host_addr=02:93:e1:61:b2:33 dev_addr=c2:a8:af:d3:32:33" > /mnt/1/etc/modprobe.d/g_multi.conf
echo ${HOSTNAME} > /etc/hostname
sed -i "s/raspberrypi/${HOSTNAME}/" /mnt/1/etc/hosts
dd if=/dev/zero of=/mnt/1/file.img count=1k bs=1k
cp /mnt/1/lib/systemd/system/serial-getty@.service /mnt/1/etc/systemd/system/serial-getty@ttyGS0.service
sed -i "s/# the entered username./# the entered username.\nEnvironment=TERM=screen/" /mnt/1/etc/systemd/system/serial-getty@ttyGS0.service
mkdir /mnt/1/etc/systemd/system/getty.target.wants
ln -sf /etc/systemd/system/serial-getty@ttyGS0.service /mnt/1/etc/systemd/system/getty.target.wants/serial-getty@ttyGS0.service
cp $IMX_GIT/br2_external/fs_overlay/etc/NetworkManager/system-connections/USB.nmconnection /mnt/1/etc/NetworkManager/system-connections/
echo -e "#Office:\n#stty cols 136 rows 35\n\n#Home:\nstty cols 173 rows 41\n" >> /mnt/1/home/pi/.bashrc
echo -e "\nescape ^Bb\n" >> /mnt/1/etc/screenrc
find /mnt/1/lib/firmware -mindepth 1 ! \( -name  "regulatory*" \) -delete
mkdir /mnt/1/lib/firmware/atmel
cp $FW_DIR/atmel/* /mnt/1/lib/firmware/atmel/
cp /$IMXDIR/images/zImage /mnt/1/boot/vmlinuz-6.13.3gh
ln -sf /boot/vmlinuz-6.13.3gh /mnt/1/boot/vmlinuz
cp /$IMXDIR/images/ghazans-imx6ulz.dtb /mnt/1/boot/ghazans-imx6ulz.dtb
cp -R $IMXDIR/target/lib/modules/6.13.3gh /mnt/1/lib/modules/
rm -f /mnt/1/etc/*/*/z50-raspi-firmware
rm -f /mnt/1/etc/initramfs/*/z50-raspi-firmware
rm -f /mnt/1/var/swap
cd /
umount /mnt/1
losetup -D
```


## Burn image and first boot

Fastboot will not work for large images. It needs a buffer size bigger than the file and with 512MB memory, the largest buffer we can use is around 500MB. 
So we use UMS in U-Boot which presents the whole eMMC as a USB Mass Storage device:

If ums.imx is already built:
`sudo uuu ums.imx`

Else:
`sudo uuu u-boot.imx`

In a serial console, run:
`ums 0 mmc 0`

Upload the ext4 partition image to the ext4 partition we created. It will take around 7 minutes.

`dd if=raspios.img of=/dev/sda1 bs=1K status=progress`

### If there are boot errors

In U-Boot, add `init=/bin/bash` to bypass systemd init on first boot:

`env set bootargs 'console=ttymxc0,115200 root=/dev/mmcblk0p1 rootwait rw init=/bin/bash'`

You end up in a shell.

```
mount -t proc none /proc
mount -t sysfs none /sys
mount -o remount,rw /
```

## If you can login and get a prompt

Before any package updates or installs, remove raspi-firmware:
`dpkg -P raspi-firmware`

Connect network, unless the usb0 interface is already connected and your host is passing through traffic

Set date to today-ish or apt just fails:

`date 022612122025`

Apt updates and upgrades

`sudo apt update`
`sudo apt upgrade`

Disable unnecessary services

```
systemctl stop apparmor.service
systemctl stop avahi-daemon.service
systemctl stop cryptdisks.service
systemctl stop dphys-swapfile.service
systemctl stop keyboard-setup.service
systemctl stop ModemManager.service
systemctl stop pigpiod.service
systemctl stop rpi-display-backlight.service
systemctl stop rpi-eeprom-update.service
systemctl stop triggerhappy.service
systemctl stop userconfig.service
systemctl disable apparmor.service
systemctl disable avahi-daemon.service
systemctl disable userconfig.service
systemctl disable triggerhappy.service
systemctl disable rpi-eeprom-update.service
systemctl disable rpi-display-backlight.service
systemctl disable pigpiod.service
systemctl disable ModemManager.service
systemctl disable keyboard-setup.service
systemctl disable dphys-swapfile.service
systemctl disable cryptdisks.service
```

Optional:
```
systemctl stop alsa-utils.service
systemctl stop bluetooth.service
systemctl stop hciuart.service
systemctl stop wpa_supplicant.service
systemctl disable bluetooth.service
systemctl disable hciuart.service
systemctl disable  wpa_supplicant.service
```

Find the biggest packages to remove if not needed:


```
apt install debian-goodies
dpigs -H -n 30
```

Packages to remove:

```
sudo dpkg -P firmware-atheros firmware-misc-nonfree rpi-eeprom firmware-libertas firmware-brcm80211 firmware-realtek bluez-firmware
sudo dpkg -P rpi-eeprom raspinfo raspi-utils raspi-config userconf-pi raspberrypi-sys-mods raspberrypi-net-mods pi-bluetooth rpi-update
sudo dpkg -P linux-headers-6.6.51+rpt-common-rpi linux-headers-6.6.51+rpt-rpi-v6 linux-headers-6.6.51+rpt-rpi-v7 linux-headers-6.6.51+rpt-rpi-v7l linux-image-6.6.51+rpt-rpi-v6 linux-image-6.6.51+rpt-rpi-v7 linux-image-6.6.51+rpt-rpi-v7l linux-image-6.6.51+rpt-rpi-v8 linux-image-rpi-v8 linux-kbuild-6.6.51+rpt
sudo dpkg -P linux-image-rpi-v6 linux-image-rpi-v7l linux-headers-rpi-v6 linux-headers-rpi-v7l linux-image-6.6.74+rpt-rpi-v6 linux-image-6.6.74+rpt-rpi-v7l linux-headers-6.6.74+rpt-rpi-v6 linux-headers-6.6.74+rpt-rpi-v7l
sudo dpkg -P v4l-utils libv4lconvert0 libv4l2rds0 libv4l-0
sudo dpkg -P bluez
```

Kernel compile tools:
```
sudo apt install build-essential
sudo apt install bison flex libncurses-dev
```

Slim down:
```
sudo apt autoclean
sudo apt clean
sudo apt autoremove
journalctl --flush --rotate --vacuum-time=1s
journalctl --user --flush --rotate --vacuum-time=1s
```

