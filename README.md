# README

This is a proof-of-concept for generating an initramfs that you can use to migrate a commodity Linux
to a more reliable embedded setup (such as Nerves).

This repo is built for a specific circumstance and needs adaptation to work in anything else:

- Raspberry Pi OS
- 1 boot partition at mmcblk0p1
- 1 root partition at mmcblk0p2 that needs to be shrunk
- you like the size 3G (otherwise change the `init` file)
- support for tryboot, should be on most current rpi4s, otherwise shouldn't do any damage

## Further requirements

Requires a kernel file packaged up as `nerves-kernel8.img`, it needs to support initramfs block devices. So if you grab `nerves_system_rpi4` and at the time of writing you need `main` because `CONFIG_BLK_DEV_INITRD=y` was just added.

I am including a compiled kernel that should work for convenience. You need to copy all files from `boot-firmware-files` to the `/boot/firmware` folder on your device.

- `tryboot.txt` read instead of `config.txt` if you restart your device with `reboot "0 tryboot"`
- `nerves-kernel8.img` kernel image that includes necessary drivers for the initramfs. Chosen by `tryboot.txt`.
- `rootfs.cpio.gz` the compiled version of this initramfs. Chosen by `tryboot.txt`.
- `cmdline2.txt` uses the initramfs as the root filesystem. Chosen by `tryboot.txt`.
- `config.txt` slightly modified, optional, just adds better logging.

## The procedure

Using the tryboot mechanism by running `sudo reboot "0 tryboot"` you tell your RPi to reboot and attempt one (1)
tryboot run. It will pick tryboot.txt over config.txt and determine the rest from there. We choose to load an
initramfs meaning that we put all of our Linux stuff into memory and can then run operations on the disk that
cannot be done while mounted, such as shrinking a partition. So we do. The script contents are in the file `configs/rpi4/init`.

When they've run the disk should be resized down to that hardcoded value of around 3GB. It then restarts back into RPi OS or whatever you were running. Hopefully no-one being the wiser.

Confirm disk size with `df -h`.
