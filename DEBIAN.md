# Debian image

Note: We can build these debian images through qemu, or just use Raspbian and rip out the rpi packages.
The Raspbian path is documented in the RASPBIAN.md file

## Dev environment: Jessie 8 (aarch64)
This version is useful in qemu because it comes with aarch64 (runs fast on Macbook) while still providing gcc-4.x for older kernels
Maybe it can be turned into UEFI VMware image too

```
qemu-img create -f qcow debian8-arm64.img 10G
wget http://ftp.ru.debian.org/debian/dists/jessie/main/installer-arm64/20150107/images/netboot/debian-installer/arm64/initrd.gz
wget http://ftp.ru.debian.org/debian/dists/jessie/main/installer-arm64/20150107/images/netboot/debian-installer/arm64/linux

qemu-system-aarch64 -machine virt -cpu cortex-a57 -nographic -smp 1 -m 512 -kernel linux -initrd initrd.gz -append root=/dev/ram console=ttyAMA0 -global virtio-blk-device.scsi=off -device virtio-scsi-device,id=scsi -drive file=debian8-arm64.img,id=rootimg,cache=unsafe,if=none -device scsi-hd,drive=rootimg -netdev user,id=unet -device virtio-net-device,netdev=unet -net user
```
Ref: https://gist.github.com/philipz/04a9a165f8ce561f7ddd


## Squeeze 6
For armv5 armel in QEMU, use:

https://people.debian.org/~aurel32/qemu/armel/
```
qemu-system-arm -M versatilepb -kernel vmlinuz-2.6.32-5-versatile -initrd initrd.img-2.6.32-5-versatile -hda debian_squeeze_armel_standard.qcow2
```
Updates gcc-4.6 to gcc-4.8
Compiles 2.6.32 nicely
Might need to run 
`echo 'APT::Get::AllowUnauthenticated "true";' > /target/etc/apt/apt.conf.d/99bypass`



## Lenny 5

Create a raw 1GB empty image: hda.img

Download netboot kernel and initrd and run:
```
wget http://archive.debian.org/debian/dists/lenny/main/installer-armel/current/images/versatile/netboot/vmlinuz-2.6.26-2-versatile
wget http://archive.debian.org/debian/dists/lenny/main/installer-armel/current/images/versatile/netboot/initrd.gz
qemu-system-arm -M versatilepb -kernel vmlinuz-2.6.26-2-versatile -hda hda.img -initrd initrd.gz -append "root=/dev/sda1 console=ttyAMA0" -m 256 -serial stdio
```
Instructions say copy out the initrd from within /boot after install to use
GCC: ?
Compiles 2.6.27?

Another mirror: http://ftp.de.debian.org/debian-archive/debian/dists/lenny/

Errors out the first try, do a shell and add this file:
`echo 'APT::Get::AllowUnauthenticated "true";' > /target/etc/apt/apt.conf.d/99bypass`
Reboot and try again without wiping out filesystem
Works second try.
Normally we get the vmlinuz and initrd from /boot but the wheezy version works well:
```
wget https://people.debian.org/~aurel32/qemu/armel/vmlinuz-2.6.32-5-versatile
wget https://people.debian.org/~aurel32/qemu/armel/initrd.img-2.6.32-5-versatile
qemu-system-arm -M versatilepb -kernel vmlinuz-2.6.32-5-versatile -hda hda.img -initrd initrd.img-2.6.32-5-versatile -append "root=/dev/sda1 console=ttyAMA0" -m 256 -serial stdio
```



## Etch 4


