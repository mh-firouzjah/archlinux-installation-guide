# Archlinux Installation Guide

Here I explain a step-by-step guide to install Archlinux.
- this document is heavily inspired by
[freecodecamp - how-to-prepare-for-installing-arch](https://www.freecodecamp.org/news/how-to-install-arch-linux/#how-to-prepare-your-computer-for-installing-arch-linux)

## Pre Installation Step

### Bootable ISO

Download the ISO from https://archlinux.org/download/.

Burn it to a CD/DVD or crate a bootable USB flash drive using `dd` or
if you prefer a GUI application I suggest `Fedora Media Writer` from
https://fedoraproject.org/en/workstation/download/.

```bash
of=/dev/sdx # the usb drive identified by lsblk
dd bs=4M if=path/to/archlinux.iso of=/dev/sdx conv=fsync oflag=direct status=progress
```

The first change that you'll have to make is disabling secure boot in your UEFI configuration.

The second thing that you should disable is only relevant
if you're installing Arch Linux alongside Windows.
There is a Windows feature called fast startup that reduces
the boot time of your computer by partially hibernating it.

### Live Environment

Assuming that you have a bootable USB drive and
your computer is configured properly,
you'll have to boot from the USB drive.
The process of booting from a USB drive differs from machine to machine.

To verify the boot mode, execute the following command.
If you're in UEFI mode then, it will list out a bunch of files on your screen:

```bash
ls /sys/firmware/efi/efivars
```

If you're using a wired network then you should have a working internet connection
from the get go, But if you have a wireless connection, things can get a bit tricky.

```bash
iwctl --passphrase PASSWORD station wlan0 connect wi-fi-name
```

Check if connection was successful:

```bash
ping -c 4 archlinux.org
```

Enable auto adjustment of date & time over network

```bash
timedatectl set-ntp true
```

Update pacman mirrorlist

```bash
cp /etc/pacman.d/mirrorlist  /etc/pacman.d/mirrorlist.bac

pacman -Sy reflector

reflector --ipv6 -c NL,DE,FR,GB -p https -a 24 -l 30 --sort rate -f 20 --save /etc/pacman.d/mirrorlist
# --country NL,DE,FR,GB --protocol https --age 24 --latest 30 --sort rate --fastest 20

```

### Partition the disk

According the Archlinux wiki a good example could be as the table bellow:

 - efi should be 1 GiB and mounted on /boot for systemd-boot and
 could be 300 MiB and mounted on /boot/efi for grub.

| Partition | Size      | File System type  | Mount Point   | Disk Drive          |
| :---:     | :---:     | :---:             | :---:         | :---:               |
| efi       | 1 GiB     | EFI system        | /mnt/boot     | /dev/efi_partition  |
| efi       | 300 MiB   | EFI system        | /mnt/boot/efi | /dev/efi_partition  |
| root      | remaining | Linux file system | /mnt          | /dev/root_partition |

For example using fdisk would be like this:

```bash
fdisk /dev/nvme0n1

Command (m for help): g

Command (m for help): n
Partition number (1-128, default 1): enter
First sector (2048-15441886, default 2048): enter
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-15441886, default 15439871): +1G

Command (m for help): n
Partition number (3-128, default 3): enter
First sector (9439232-15441886, default 9439232): enter
Last sector, +/-sectors or +/-size{K,M,G,T,P} (9439232-15441886, default 15439871): enter

Command (m for help): t
Partition number (1-2, default 2): 1
Partition type or alias (type L to list all): 1

Command (m for help): w
```

Now format the partitions you've just created

```bash
mkfs.fat -F 32 -n LABEL /dev/nvme0n1p1
mkfs.btrfs -f -L LABEL /dev/nvme0n1p2
```

Prepare for BTRFS installation

```bash
mount /dev/nvme0n1p2 /mnt

btrfs su cr /mnt/@
btrfs su cr /mnt/@root
btrfs su cr /mnt/@home

# check the result
btrfs su li /mnt

umount /mnt
```

### Mount the partitions

```bash
options=ssd,discard,noatime,compress-force=zstd:8,commit=130,space_cache=v2,autodefrag,max_inline=512k,inode_cache,subvol=@

mount -o ${options} /dev/nvme0n1p2 /mnt

mkdir -p /mnt/{boot/efi,hdd,home,root}

mount -o ${options}root /dev/nvme0n1p2 /mnt/root

mount -o ${options}home /dev/nvme0n1p2 /mnt/home

# for systemd-boot
- mount /dev/nvme0n1p1 /mnt/boot
# for grub
- mount /dev/nvme0n1p1 /mnt/boot/efi

mount -t ntfs-3g -o rw,nls=utf8,noatime,windows_names,permissions,allow_others /dev/sda1 /mnt/hdd

# check the result
lsblk
```

### Install the base system then Login to newly installed system using arch-chroot

```bash
pacman -Sy

pacstrap -K /mnt base base-devel linux-lts linux-firmware intel-ucode ntfs-3g btrfs-progs vim 

genfstab -U /mnt >> /mnt/etc/fstab

cp /etc/pacman.d/mirrorlist /mnt/etc/pacman.d/mirrorlist

arch-chroot /mnt
```

### Configure the newly installed system form inside

```bash
# set timezone
ln -sf /usr/share/zoneinfo/Asia/Tehran /etc/localtime

# adjust time
hwclock --systohc

# enable date & time adjustment over network
timedatectl set-ntp true

# generate locale
sed -i '/en_US.UTF-8/s/^#\s*//' /etc/locale.gen

locale-gen

# generate language locale.conf
echo "LANGE=en_US.UTF-8" > /etc/locale.conf

# set consolefont
echo "FONT=LatArCyrHeb-14" > /etc/vconsole.conf

# set hostname
hostname=my_bluetooth_name

echo "${hostname}" > /etc/hostname

# set hosts
echo "127.0.0.1 localhost" > /etc/hosts
echo "::1  localhost" >> /etc/hosts
echo "127.0.1.1 ${hostname}.localdomain ${hostname}" >> /etc/hosts

# set the root password
passwd

# create non-root user
useradd -m -G wheel -s /bin/bash USERNAME
passwd USERNAME

# in order to give new user sudo access
# uncomment (remove the # at line start) %wheel ALL=(ALL) ALL
visudo
```

## Install bootloader

<details>
  <summary>Grub</summary>

  Install required packages:

  ```bash
  pacman -S grub efibootmgr
  ```

  Now, we'll use the grub-install command to
  install GRUB in the newly mounted EFI system partition:

  ```bash
  grub-install --target=x86_64-efi --bootloader-id=grub
  ```

  - If you're installing alongside other operating systems:</summary>

    - you'll also need the os-prober package:

      ```bash
      pacman -S os-prober
      ```

    - and you have to uncomment `#GRUB_DISABLE_OS_PROBER=false` from `/etc/default/grub`:

      ```bash
      vim /etc/default/grub
      ---------------------
      remove '#' from the following line
      #GRUB_DISABLE_OS_PROBER=false
      ```

  Now execute the following command to generate the configuration file:

  ```bash
  grub-mkconfig -o /boot/grub/grub.cfg
  ```
</details>

<details>
  <summary>systemd-boot</summary>

  ```bash
  bootctl install --efi-boot-option-description="Arch Linux LTS"
  ```

  To set a loader edit `/boot/loader/loader.conf` and define a default entry

  ```bash
  vim /boot/loader/loader.conf
  ---------------------
  default arch-lts.conf
  timeout 0
  editor no
  console-mode max
  ```

  Define default and fallback bootloaders

  ```bash
  vim /boot/loader/entries/arch-lts.conf
  ---------------------
  title Arch Linux LTS
  linux /vmlinuz-linux-lts
  initrd /intel-ucode.img
  initrd /initramfs-linux-lts.img
  options root=/dev/nvme0n1p2 rootfstype=btrfs rootflags=subvol=@ rw quiet add_efi_memmap

  vim /boot/loader/entries/fallback.conf
  ---------------------
  title Fallback
  linux /vmlinuz-linux-lts
  initrd /intel-ucode.img
  initrd /initramfs-linux-lts-fallback.img
  options root=/dev/nvme0n1p2 rootfstype=btrfs rootflags=subvol=@ rw quiet add_efi_memmap
  ```

  Check if boot entries are recognized and there's no error

  ```bash
  bootctl list
  ```

  Because we use systemd-boot,
  edit `/etc/mkinitcpio.conf` at `HOOKS=` replace `udev` hook with `systemd`,
  then regenerate initramfs as well

  ```bash
  vim /etc/mkinitcpio.conf
  ---------------------
  HOOKS=(base systemd[udev] fsck ...)

  # regenerate initramfs
  mkinitcpio -P
  ```

</details>


## Enable Network & Firewall

I'm gonna use UFW which is very simple firewall

```bash
pacman -S ufw networkmanager
systemctl enable ufw NetworkManager

ufw default deny incoming
ufw default allow outgoing
ufw enable
```

## Better pacman

```bash
vim /etc/pacman.conf
---------------------
Color
VerbosePkgLists
ILoveCandy
ParallelDownloads = 3
```

## Install Graphics Driver

Installing graphics drivers on Arch Linux is very straightforward.
You just install the packages required by your graphics processing unit and call it a day.

```bash
# for amd discreet and integrated graphics processing unit
pacman -S xf86-video-amdgpu

# for nvidia graphics processing unit
pacman -S nvidia-lts nvidia-prime nvidia-settings xorg-server-devel opencl-nvidia

# for intel integrated graphics processing unit
pacman -S xf86-video-intel
```

## Install Desktop

Archlinux lets you select among a bunch of desktops such as Plasma, Gnome, Lxqt and many more
also beside these full featured matore desktops there are a lot of window managers available
e.g qtile, iw3, bspwm, ...

I like KDE Plasma so I will install that following these comands

- During the installation,
you'll get multiple choices for `ttf-font`, `pipwire-session-manager`,
and `phonon-qt5-backend` packages. Make sure to pick these ones:
  - noto-fonts
  - phonon-qt5-gstreamer
  - wireplumber

```bash
pacman -S plasma plasma-wayland-session

systemctl enable sddm
```

## Finalize The Installation

Now that you've installed Arch Linux and
gone through all necessary configuration steps,
you can reboot to your newly installed system.
To do so:

```bash
# first come out of the Arch-Chroot environment
exit

# unmount the root partition to make sure there are no pending operations
umount -R /mnt

# reboot the machine
reboot
```

## Post Install

### ZRAM instead of ZSWAP

```bash
pacman -S zram-generator

echo "blacklist zswap" > /etc/modprobe.d/disable-zswap.conf

vim /etc/systemd/zram-generator.conf
---------------------
[zram0]
zram-size = ram / 2
compression-algorithm = zstd
swap-priority = 100
fs-type = swap

vim /etc/sysctl.d/99-vm-zram-parameters.conf
---------------------
vm.swappiness = 180
vm.watermark_boost_factor = 0
vm.watermark_scale_factor = 125
vm.page-cluster = 0
```

### Prevent acpid spamming the journal logs because of arrow keys events

```bash
vim /etc/acpi/events/buttons
---------------------
event=button/(up|down|left|right|kpenter)
action=<drop>
```

### Safe gard google-chrome using some flags

```bash
vim ~/.config/chrome-flags.conf
---------------------
#--incognito
--process-per-site
--js-flags="--noexpose_wasm --no-experimental-wasm-anyref --jitless -clear-free-memory --single-threaded-gc"
```

### Enable profile-sync-daemon

After login with non-root user execute the following commands:

```bash
psd
# First time running psd so please edit ~/.config/psd/psd.conf to your liking and run again
systemctl --user enable psd
# Created symlink ~/.config/systemd/user/default.target.wants/psd.service → /usr/lib/systemd/user/psd.service
systemctl --user start psd
```

### Force SDDM to use external monitor

```bash
vim /usr/share/sddm/scripts/Xsetup
---------------------
#!/bin/sh

# run xrandr to see which monitors you have
intern=eDP1
extern=HDMI1

if xrandr | grep "$extern disconnected"; then
    xrandr --output "$extern" --off --output "$intern" --auto
else
    xrandr --output "$intern" --off --output "$extern" --auto
    ## To leave the default monitor enabled when an external monitor is connected, replace the else clause with
    # xrandr --output "$intern" --primary --auto --output "$extern" --right-of "$intern" --auto
fi
```

### Enable touchpad on SDDM

```bash
vim /etc/X11/xorg.conf.d/20-touchpad.conf
---------------------
Section "InputClass"
        Identifier "libinput touchpad catchall"
        MatchIsTouchpad "true"
        MatchDevicePath "/dev/input/event*"
        Driver "libinput"

        Option "Tapping" "true"
        Option "NaturalScrolling" "true"
        Option "MiddleEmulation" "true"
        Option "DisableWhileTyping" "true"
EndSection
```