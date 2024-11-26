# My IMX6ULZ demo board

- SOC: MCIMX6Z0DVM09AB
- DDR3L: D2516ECMDXGJD-U (4gb)
- eMMC: EMMC08G-MV28-01J22 on USDHC1/4bit
- QSPI: S25FL256SDPMFV001 (32MB)
- PMIC: discrete! Two RT8070Z
- WIFI: ATWILC3000 module
- SD card slot on USDHC2/4bit/no detect pin


## DTS structure

- Console: uart1/ with rts/dts
- usart2: available without rts/dts
- sdhc1: emmc, 4bit, no CD, Reset:GPIO1_IO09
- sdhc2: sd slot, 4bit, no CD or Volt
- QSPI: 4bit, CS: A_SS0_B
- audio?


## eMMC image map

Address     What    Size        In 512b Blocks
0x0         MBR     0x200       1
0x400       U-Boot  0x80000     0x400
0x1000      zImage  0x800000    0x4000
0x5000      dtb     0x40000     0x200
0x6000      env     0x2000      0x10
0x8000      rootfs  0x4000000   0x20000     Ext2 partition starts at 16MB


The board boots from uSDHC1. To enable this:

# Fuse settings from u-boot:

`fuse prog 0 5 00002060`
This configures eMMC boot (BOOT_CFG1)

`fuse prog 0 6 00000010`
This sets BT_FUSE_SEL to enable fuse reading

BOOT_MODE pins now can be 00 or 10 (internal) but not 01 (Serial boot)

TODO: Do a recovery fuse to enable QSPI or uSDHC2 as a backup


# MMC setup:

```
mmc bootbus 0 1 1 0
mmc partconf 0 0 7 0
```
- Enables user area as a boot area, disables boot areas
- This only needs to be run once.


# Partition setup:

This is primarily for rootfs. ROM loads u-boot.imx directly from 0x400 and all partitions exclude this region

```
env set partitions 'name=rootfs,start=16M,size=-,id=0x83'
mbr write mmc 0 ${partitions}
```

If GPT is preferred over MBR:

```
env set partitions 'uuid_disk=EE5C545B-2234-4CEA-AA8A-2B7FF8C1A1F0;name=bootloader,size=32M,bootable,uuid=1F7FD4B6-E20D-446C-B3E9-C2819557E2A4;name=rootfs,size=-,uuid=6B2D7C13-EC4A-4D4C-A362-AE8B0F423554'
gpt write mmc 0 $partitions
gpt set-bootable mmc 0 bootloader
part set mmc 0 efi
```


# U-Boot

Make sure only SD/EMMC is enabled under *Boot media* in U-boot config. This sets BOOT_MODE to sd in the imximage.cfg file which fixes the offset of u-boot.bin within u-boot.imx. If you enable other boot media options as well, the BOOT_MODE setting will default to qspi etc and the offset might be wrong.

Enable u-boot.imx as an output in buildroot config (not U-Boot's config).

The imx file includes IVT and DCD data from the file *uboot-2024.10/board/freescale/mx6sllevk/imximage.cfg*

Although the defaults of DCD worked for my board, it is best to go through the NXP's DDR stress test tool (Windows only) and their great spreadsheet for DDR settings *MX6UL_ULL_ULZ_MMDC_DDR3_register_programming_aid_v1.1.xls* 

This spreadsheet outputs registry settings to work with your DDR setup, but not in imximage.cfg's DCD format. You'll need to manually copy all reg entries and update imximage.cfg

The DDR stress test tool also updates a few calibration registries. Update them in imximage.cfg as well.


# Flashing

Boot the board in serial download mode and get it to run u-boot from memory:
You'll need to connect the board using usb and have boot jumpers configured for serial download mode.
You'll also need UART connected. My board (and dts) has uart1 as console output at 115200

`sudo uuu u-boot.imx`

U-Boot will stop at some stage. Run fastboot:

`fastboot -l 0x82000000 usb 0`

On the host machine, upload the u-boot file throught the fastboot protocol over USB:

`sudo uuu fb: download -f u-boot.imx`

Back in U-Boot, CTRL-C to exit fastboot and now write the uploaded image to the correct location, 1KB or 2 blocks from the start of the storage:

`mmc write 0x82000000 0x2 0x500`

Upload the kernel image the same way and put it 2MBytes in to give U-Boot enough space, but still in the hidden section before the first partition. We copy 8MB (0x4000 blocks)

In U-Boot:
`fastboot -l 0x82000000 -s 0x800000 usb 0`

From host:
`sudo uuu FB: download -f zImage`

U-Boot:
`mmc write 0x82000000 0x1000 0x4000`

Upload the devicetree 10MB in and 128KB max size:

U-Boot: `fastboot -l 0x82000000 usb 0`

Host: `sudo uuu FB: download -f ghazans-imx6ulz.dtb`

U-Boot: `mmc write 0x82000000 0x5000 0x100`


Upload the Buildroot ext4 rootfs:

U-Boot: `fastboot -l 0x82000000 -s 0x8000000 usb 0`
Host: `sudo uuu FB: download -f rootfs.ext2`
U-Boot: `mmc write 0x82000000 0x8000 0x20000`
(Only works for 64MB images(0x20000), adjust this size according to your image)

Reboot


# Boot

Load kernel: `mmc read ${loadaddr} 0x1000 0x4000`
Load dtb: `mmc read ${fdt_addr} 0x5000 0x100`
Bootargs: `env set bootargs 'console=ttymxc0,115200 root=/dev/mmcblk0p1 rootwait rw'`
Boot: `bootz ${loadaddr} - ${fdt_addr}`

# To automate boot

```
env set bootcmd 'mmc read ${loadaddr} 0x1000 0x4000;mmc read ${fdt_addr} 0x5000 0x100;bootz ${loadaddr} - ${fdt_addr}'
env set mmcdev 0
env set mmcpart 0
env save
```

Now it should boot correctly on start



# Using a Raspbian image

Download *2024-11-19-raspios-bookworm-armhf-lite.img.gz*

Unzip, mount as loopback device and extract the ext4 partition to another file

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
mount -t proc /proc
blkid /dev/mmcblk0p1
```

Copy the UUID to /etc/fstab, changing PARTUUID to just UUID

Add a password to the pi user:

`passwd pi`

Reboot and login
