# Using a Raspbian image

Download *2024-11-19-raspios-bookworm-armhf-lite.img.gz*

Unzip, mount as loopback device and extract the ext4 partition to another file, called raspios.img here


## Fixup image contents

Mount this partition image in a linux host:
`losetup -fP raspios.img`
`mounti /dev/loop0 /mnt/1`

Remove pi password in /etc/shadow:
`pi::20130:0:99999:7:::`

## USB Gadget

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

## ATWILC1000 firmware

Download firmware files: https://github.com/linux4wilc/firmware
Place them 


## Burn image and first boot

Fastboot will not work for large images. It needs a buffer size bigger than the file and with 512MB memory, the largest buffer we can use is around 500MB. 
So we use UMS in U-Boot which presents the whole eMMC as a USB Mass Storage device:

`ums 0 mmc 0`

Upload the ext4 partition image to the ext4 partition we created. It will take around 7 minutes.

`dd if=raspios.img of=/dev/sda1 bs=1K status=progress`

On first boot, Raspbian throws errors and hangs for not finding its DOS partition. It also has no password for its *pi* user.

So in U-Boot, add `init=/bin/bash` to bypass systemd init on first boot:

`env set bootargs 'console=ttymxc0,115200 root=/dev/mmcblk0p1 rootwait rw init=/bin/bash'`

You end up in a shell.

Edit */etc/fstab* and remove the vfat partition
Change the UUID of the root partition to actual:
```
proc            /proc           proc    defaults          0       0
UUID=bb15c8e6-d999-4838-be67-5ff200bffa46  /               ext4    defaults,noatime  0       1
```


```
mount -t proc /proc
blkid /dev/mmcblk0p1
```

Copy the UUID to /etc/fstab, changing PARTUUID to just UUID

Add a password to the pi user:

`passwd pi`

If youre working on the fs image offline, just remove the password entry in /etc/shadow to blank the password

Reboot and login



## TODO notes

fstab fix
pi password set
Copy kernel modules
Enable g_multi in /etc/modules and conf
stty in bash rc: stty  cols 173 rows 42
time/date update
Local IP config and dhcpd to setup usb0 automatically
services disable
/boot/firmware???

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

```
root@raspberrypi:~ # cat /etc/modprobe.d/g_multi.conf
options g_multi iSerialNumber=126 file=/file.img removable=y ro=0 stall=0


g_multi in /etc/modules


root@raspberrypi:~ # cat /etc/fstab
proc            /proc           proc    defaults          0       0
UUID=bb15c8e6-d999-4838-be67-5ff200bffa46  /               ext4    defaults,noatime  0       1



root@raspberrypi:~ # ls -l /etc/systemd/system/getty.target.wants/
total 0
lrwxrwxrwx 1 root root 41 Nov 25 21:28 serial-getty@ttyGS0.service -> /lib/systemd/system/serial-getty@.service


