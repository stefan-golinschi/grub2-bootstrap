# GRUB2 Bootstrap

This is an example on how to use GRUB2 as a bootstrap loader.
This allows you to burn ISO9660 files within a partition rather than to burn it on the entire disk, creating the possiblity to have a single flash drive with multiple ISOs to boot from. It's like [Ventoy](https://www.ventoy.net/en/index.html), but using a different approach, maybe more suitable for other use cases.

For now, this project covers only UEFI-compatible system compatibility. For legacy/BIOS systems, take a look here: [stefan-golinschi/microserver-bootstrap](https://github.com/stefan-golinschi/microserver-bootstrap).


## Disk Partitioning

Disk partitioning is done using `gdisk`, by creating an EFI system partition for the bootloader and some other partitions, depending on your specific requirements.

A sample partition layout is [layout.sfdisk](./layout.sfdisk), which can be easily imported using sfdisk like this:

```
sfdisk /dev/sdg < layout.sfdisk
```


## Disk formatting

```
sudo mkfs.vfat -F32 /dev/sdg1
```


## Installing the bootloader

If you use Fedora, you will need to install this package `grub2-efi-x64.x86_64`.

```
sudo mount /dev/sdg1 data
sudo grub2-install --target=x86_64-efi --efi-directory=./data/ --boot-directory=./data/boot --removable --force /dev/sdg

# TODO: For legacy/BIOS systems
# sudo grub2-install --target=i386-pc --boot-directory=./data/boot --removable /dev/sdg
# sudo grub2-install --target=i386-efi --efi-directory=./data/ --boot-directory=./data/boot --removable --force /dev/sdg
```


## Burning bootable ISOs

For example, we will use two alpine linux ISOs.

```
sudo dd if=alpine-standard-3.19.1-x86_64.iso of=/dev/sdg2 status=progress
sudo dd if=alpine-extended-3.18.3-x86_64.iso of=/dev/sdg3 status=progress
```

After this, make sure to update the [grub.cfg](./grub.cfg) file accordingly. For example, I used GRUB2's `search --label` function to set the root for each menuentry.

If you want to know the labels of the ISOs, you can use `file alpine-standard-3.19.1-x86_64.iso`, or you can do a `lsblk -o +LABEL` after you burn the ISOs to the flash drive.

Finally, copy the grub config file to the EFI ESP partition on the flash drive.

```
sudo cp grub.cfg data/boot/grub2/
```


## References

 * https://colinxu.wordpress.com/2018/12/29/create-a-universal-bootable-usb-drive-using-grub2
