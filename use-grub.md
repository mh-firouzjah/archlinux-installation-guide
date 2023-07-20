# Use GRUB2 as boot manager

take these steps so to be able to boot using grub

## Install necessary packages

```bash
sudo pacman -S grub efibootmgr shim os-prober
```

## Mount the EFI partition

```bash
sudo mkdir /boot/efi
sudo mount /dev/sdXY /boot/efi
# Replace /dev/sdXY with the actual device and partition number for your EFI partition
```

## Install GRUB to the EFI system partition (support secure boot)

```bash
sudo grub-install \
--target=x86_64-efi \
--efi-directory=/boot/efi \
--bootloader-id=GRUB \
--modules="normal test efi_gop efi_uga search echo linux all_video gfxmenu gfxterm_background gfxterm_menu gfxterm loadenv configfile tpm"
```

## Add required kernel modules

In order to add required kernel modules, edit `/etc/default/grub` and add modules for `GRUB_CMDLINE_LINUX`

```bash
GRUB_CMDLINE_LINUX="root=/dev/nvme0n1p2 rootfstype=btrfs rootflags=subvol=@ rw quiet nowatchdog add_efi_memmap acpi_osi=Linux acpi_backlight=vendor zswap.enabled=0"
```


## Detecting other operating systems

To have grub-mkconfig search for other installed systems and automatically add them to the menu, 
install the `os-prober` package and mount the partitions from which the other systems boot. 
Then re-run grub-mkconfig. If you get the following output: 
`Warning: os-prober will not be executed to detect other bootable partitions` then 
edit `/etc/default/grub` and add/uncomment:

```bash
GRUB_DISABLE_OS_PROBER=false
```

Then try again.

## Generate the GRUB configuration file

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

## Create a new EFI boot entry

Here in the following command `GRUB` is the `bootloader-id` from previous steps and
`ESP` is the mount point of the fat32 partition, which sould be `/boot/efi`.

```bash
sudo efibootmgr --disk /dev/sdX --part Y --create --label "Arch Linux LTS" \
--loader /ESP/GRUB/grubx64.efi \
# Replace /dev/sdX with the disk containing the EFI partition, Y with the partition number of the EFI partition, 
# and XXXX with the PARTUUID of the root partition.
# If you have an AMD CPU, replace \intel-ucode.img with \amd-ucode.img in the above command
```


## Reboot the system

```bash
sudo systemctl reboot
```
