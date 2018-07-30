# Build

Run `generate-images` from the checkout directory.

This generates a root fs tarball (`rootfs.tar.xz`) and a firmware file
(`flash-image.bin`) containing u-boot, ARM trusted firmware etc.


# Install

## Update the firmware image

This is necessary because the original firmware doesn't set up the PCI-E port
and USB ports correctly. You'll run into all sorts of problems without the
newly built firmware.

Since storage devices may or may not work with the original firmware, the safest
way to update the firmware is through TFTP. Install the `tftp-server` package,
copy `flash-image.bin` to `/srv/tftp` and run the tftp server
("`systemctl start tftpd`").

U-Boot uses the Ethernet port next to the USB port, so make sure that one is
connected.

Then, in u-boot (the prompt you get from the board's serial console), run
```
env default -a
env save
setenv ethact eth2 (where eth2 is the ethernet port you want to use)
setenv serverip 1.2.3.4 (where 1.2.3.4 is your TFTP server's IP address)
setenv ipaddr 1.2.3.5 (where 1.2.3.5 is the temporary IP address your board can take)
bubt
```

(Watch out for failed downloads, they're usually related to the IP address
not being set correctly. Set the variables bubt tells you about.)

`reset`

## Boot the OS...
Untar rootfs.tar.xz to a SD card, USB stick or SATA harddisk (whatever you want
to boot from)

## Autoboot
Tell u-boot what to do -- e.g. if you've installed the OS on a SATA harddisk, with
partition 1 swap, partition 2 /, partition 3 /home:

`setenv bootcmd 'scsi reset; ext4load scsi 0:2 $kernel_addr /boot/Image; ext4load scsi 0:2 $fdt_addr /boot/armada-8040-mcbin.dtb; setenv bootargs $console root=/dev/sda2 rw; booti $kernel_addr - $fdt_addr'`

Or if you've installed the OS on an SD card with / on partition 1:

`setenv bootcmd 'ext4load mmc 1:1 $kernel_addr /boot/Image; ext4load mmc 1:1 $fdt_addr /boot/armada-8040-mcbin.dtb; setenv bootargs $console root=/dev/mmcblk1p1 rw; booti $kernel_addr - $fdt_addr'`

If you ever need to return to the original autoboot setup (designed to boot
over the network):
`setenv bootcmd 'run get_images; run set_bootargs; booti $kernel_addr $ramfs_addr $fdt_addr'`
