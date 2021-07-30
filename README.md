# acpi-fixes

## General procedure

Refer to the great Arch Wiki https://wiki.archlinux.org/index.php/DSDT for a
general understanding. Check your own distribution's documentation if your boot
setup varies.

You will most likely need to re-do this after firmware upgrades. Remember to
boot the kernel *without* the override to get the updated vendor tables.

Also be aware that a firmware update might screw your EFI boot settings, so
keep a rescue stick at hand.


## Example procedure for an ACER Swift3 SF314-42

On a side note, this model uses an Insyde firmware which is currently only
upgradable via Windows. Though, it extracts /Insyde and /UpdateCapsule to the
ESP and boots it, which would allow for retrieval of the actual flash program.
It also reset the EFI boot entries for me.

### Dump
```
sudo acpidump -n DSDT -b
```
### Disassemble
```
iasl -d dsdt.dat
```
### Edit
```
$EDITOR dsdt.dsl
```
### Compile
```
iasl -tc dsdt.dsl
```
### Create kernel override preload image
```
mkdir -p kernel/firmware/acpi
mv *.aml kernel/firmware/acpi/
find kernel | cpio -H newc --create > acpi_override.cpio.img
sudo mv acpi_override.cpio.img /boot
sudo chown root: /boot/acpi_override.cpio.img
```
### Add to grub.cfg's initrd command before the actual initrd and make deep default
```
kernel /boot/kernel [..] mem_sleep_default=deep
initrd /boot/amd-ucode.img /boot/acpi_override.cpio.img /boot/initrd
```
### Reboot
### Check S3/deep is there
```
sudo dmesg | grep 'ACPI:\ (supports'
[    0.366204] ACPI: (supports S0 S3 S4 S5)
cat /sys/power/mem_sleep
s2idle [deep]
```
