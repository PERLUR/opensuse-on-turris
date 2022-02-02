# opensuse-on-turris
## Documenting how to install and run openSUSE on Turris routers

I'm doing all of this through UART.

Attempting to boot openSuse Tumbleweed: http://download.opensuse.org/ports/armv7hl/tumbleweed/appliances/openSUSE-Tumbleweed-ARM-JeOS.armv7-rootfs.armv7l.tar.xz

On a seperate device insert your USB drive and download the above image.

Create a `btrfs` partition on USB `/dev/sda1` in my case.

Mounted it and extracted the above tar onto it.

On your Turris omnia, connect your UART and boot the device, if you connected your UART properly you should be able to `minicom -D /dev/ttyUSB0`
right from terminal.

We're going to need to grab the dtb(device tree blob file) from /boot dir on the Turris and symlink it.
You're looking for `/boot/armada-385-turris-omnia-phy.dtb`

On the Turris create a dir `usb`: `mkdir /usb/`
Mount the USB you have your openSuse Rootfs on by: `mount /dev/sda /usb`

copy the dtb files by: `cp /boot/armada-385-turris-omnia-phy.dtb /usb/boot` and `cp /boot/armada-385-turris-omnia-sfp.dtb /usb/boot`

Then create a symlink to `armada-385-turris-omnia-phy.dtb` by: `cd /usb/boot` and then `ln -s armada-385-turris-omnia-phy.dtb dtb`

This should give you a bootable USB wtih the proper device tree blob.

Now reboot the Turris Omnia with the USB plugged in and make sure you enter UBoot by pressing enter once you see: => that means you're in uboot prompt.
Bellow are uboot settings to boot off of USB. Now it will boot from the USB and hang up on `waiing on root device /dev/sda1`

One other strange thing is current uboot is missing `btrload` command which isn't crucial but intersting that it's missing in the current version of uboot on the Turris Omnia which is 2019.07 and `brtload` has been introduiced since 2013 so yeah. For now we substitude with `load`

**Commands to pass to uboot**

`setenv usbboot 'setenv bootargs "$bootargs cfg80211.freg=$regdomain"; usb start; load usb 0 0x01000000 boot/zImage @; load usb 0 0x02000000 boot/dtb @; bootz 0x01000000 - 0x02000000'`

`setenv bootargs 'earlyprintk console=ttyS0,115200 rootfstype=btrfs rootwait root=/dev/sda1 rootflags=subvol=@,commit=5 rw'`

`setenv bootcmd 'i2c dev 1; i2c read 0x2a 0x9 1 0x00FFFFF0; setexpr.b rescue *0x00FFFFF0; if test $rescue -ge 1; then echo BOOT RESCUE; run rescueboot; else echo BOOT eMMC FS; run usbboot; fi'`

`saveenv`
`boot`

If you have a fuck up at any time just `env default -a` and you'll be back to booting TurrisOS from emmc.

## Existing documentation
- [openSUSE Wiki - HCL:Turris Omnia](https://en.opensuse.org/HCL:Turris_Omnia)
- [openSUSE Wiki - HCL:Turris MOX](https://en.opensuse.org/HCL:Turris_Mox)

