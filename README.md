# Arch Linux Installaion Guide

In this content I'll explain how to install Arch Linux (64-bit) using systemd-boot and Ext4 or BTRFS filesystem and booting in UEFI mode

## Prepare an installation medium

- Acquire an installation ISO image

  Download the iso image from the official website at <https://archlinux.org/download/>

- Verify signature

  - Download the ISO PGP signature under Checksums in the Download Page at <https://archlinux.org/download/#checksums>

  - Verifying it with

    ```bash
    gpg --keyserver-options auto-key-retrieve --verify archlinux-VERSION-x86_64.iso.sig
    ```

  - Alternatively, from an existing Arch Linux installation run

    ```bash
    pacman-key -v archlinux-VERSION-x86_64.iso.sig
    ```

- Burn the image to CD/DVD or to make a bootable USB run

  ```bash
  dd bs=4M if=path/to/archlinux-VERSION-x86_64.iso of=/dev/sdx conv=fsync oflag=direct status=progress
  ```

## Boot the live environment

- ***Note**: Arch Linux installation images do not support Secure Boot. You will need to disable Secure Boot to boot the installation medium. If desired, Secure Boot can be set up after completing the installation.*

- Connect to the internet:

  - Ethernet — plug in the cable

  - Wi-Fi — authenticate to the wireless network using iwctl

   ```bash
   lwctl --passphrase PASSWORD station wlan0 connect WIFINAME
   ```

- The connection may be verified with `ping`

  ```bash
  ping -c 4 archlinux.org
  ```

- Update the system clock

  ```bash
  timedatectl set-ntp true
  ```

- Update pacman keys

  ```bash
  pacman -Sy
  ```

- Refresh mirrorlist

  ```bash
  reflector --ipv6 -p https -c NL,DE,FR,GB -a 24 --sort rate -f 30 --save /etc/pacman.d/mirrorlist
  ```

- Verify system uefi/bios

  ```bash
  ls /sys/firmware/efi/efivars # any output indicates uefi boot
  ```

## Partition the disks

- Since we are using UEFI boot, we need to create a GPT partition table and also we need an efi partition. it's easier to consider `/boot` for that efi partition.

- You can use 1 GiB size for `/boot` to be on the safe side.
  for example adding nvidia mudole to `mkinitcpio` will increase ~=50-60mb to each initramfs and it's fallback.

- Since `/var` is frequently read or written, it is recommended that you consider the location of this partition on a spinning disk.

  - With that being said, newer SSDs, especially those with higher capacities, have a longer lifespan (support lots of writes and erases e.g 20g/dey for near 10 years).

- If you want to go with `BTRFS`, consider creating subvolumes for other directories that contain data you do not want to include in snapshots and rollbacks of the `@` subvolume, such as `/var/cache`, `/var/spool`, `/var/tmp`, `/var/lib/machines` (systemd-nspawn), `/var/lib/docker` (Docker), `/var/lib/postgres` (PostgreSQL), and other data directories under `/var/lib/`. It is up to you if you want to follow the flat layout or create nested subvolumes. On the other hand, the pacman database in `/var/lib/pacman` ***must*** stay on the root subvolume (`@`).

  | partition | size | fs-type | mount | point | partition |
  | --- | --- | --- | --- | --- | --- |
  |efi | 1 GiB          | EFI system     | /mnt/boot | /dev/efi_partition
  |root| remaining space| Linux file sys | /mnt      | /dev/root_partition

  ```bash
  $ fdisk -l

  $ fdisk -f /dev/nvme0n1

  # Welcome to fdisk (util-linux X.X.X)
  # Changes will remain in memory only, until you decide to write them
  # Be careful before using the write command

  Command (m for help): g
  # Created a new GPT disklabel (GUID: XXX)

  Command (m for help):  n
  Partition number (1-128, default 1): enter
  First sector (2048-15441886, default 2048): enter
  Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-15441886, default 15439871): +1G
  # Created a new partition 1 of type 'Linux filesystem' and of size 1 GiB

  Command (m for help): n
  Partition number (3-128, default 3): enter
  First sector (9439232-15441886, default 9439232): enter
  Last sector, +/-sectors or +/-size{K,M,G,T,P} (9439232-15441886, default 15439871): enter
  # Created a new partition 3 of type 'Linux filesystem' and of size 2.9 GiB

  Command (m for help): p
  #
  # Device          Start     End    Sectors  Size  Type
  # /dev/nvme0n1p1     2048  1050623 1048576  512M  Linux filesystem
  # /dev/nvme0n1p2  9439232 15439871 6000640  2.9G  Linux filesystem

  Command (m for help): t
  Partition number (1-2, default 2): 1
  Partition type or alias (type L to list all): 1
  #Changed type of partition 'Linux filesystem' to 'EFI System'

  Command (m for help): p
  #
  # Device           Start      End  Sectors  Size  Type
  # /dev/nvme0n1p1     2048  1050623 1048576  1G    EFI System
  # /dev/nvme0n1p2  9439232 15439871 6000640  29G  Linux filesystem

  Command (m for help): w
  # The partition table has been altered
  ```

- Format EFI partition

    ```bash
    mkfs.fat -F 32 -n EFI /dev/nvme0n1p1
    ```

- Ext4 Partitioning

    ```bash
    mkfs.ext4 -c -e remount-ro -L ROOT -O dir_index,extent,filetype,flex_bg,has_journal,sparse_super,uninit_bg -E lazy_itable_init,discard /dev/nvme0n1p2

    mount -t ext4 -o defaults,noatime,discard,journal_checksum,commit=120,min_batch_time=200,auto_da_alloc,i_version /dev/nvme0n1p2 /mnt
    ```

- BTRFS Partitioning

    ```bash
    mkfs.btrfs -f -L ROOT /dev/nvme0n1p2

    mount /dev/nvme0n1p2 /mnt

    btrfs su cr /mnt/@            # root directory
    btrfs su cr /mnt/@home        # home directory of non-root user
    btrfs su cr /mnt/@var_cache
    btrfs su cr /mnt/@var_log
    btrfs su cr /mnt/@var_spool
    btrfs su cr /mnt/@var_tmp
    btrfs su cr /mnt/@var_lib_machines
    btrfs su cr /mnt/@var_lib_docker
    btrfs su cr /mnt/@var_lib_postgres
    btrfs su cr /mnt/@tmp         # main Temporary files location
    btrfs su cr /mnt/@snapshots   # snapper will store BTRFS snapshots here, if not using snapper this partition is not needed
    btrfs su cr /mnt/@swap        # location of swapfile

    btrfs su li /mnt

    umount /mnt

    mop="defaults,discard=async,noatime,compress=zstd:8,max_inline=512k,inode_cache,commit=120,subvol_cache,subvol=@"

    mount -t btrfs -o ${mop} /dev/nvme0n1p2 /mnt
    mkdir -p /mnt/{boot,home,var...,tmp,swap,.snapshots,hdd}
    mount -t btrfs -o ${mop}home /dev/nvme0n1p2 /mnt/home
    mount -t btrfs -o ${mop}var... /dev/nvme0n1p2 /mnt/var...
    mount -t btrfs -o ${mop}tmp /dev/nvme0n1p2 /mnt/tmp
    mount -t btrfs -o ${mop}snapshots /dev/nvme0n1p2 /mnt/snapshots

    # swapfile
    mount -t btrfs -o defaults,subvol=@swap /dev/nvme0n1p2 /mnt/swap
    btrfs filesystem mkswapfile --size 16g --uuid clear /mnt/swap/swapfile
    swapon /mnt/swap/swapfile
    ```

- EFI and other partitions

    ```bash
    # noauto - will prevent automount and regarding that we have to mount the partition manually
    # users - will cause any user have read/write access but the drive is owned by root
    mount /dev/nvme0n1p1 /mnt/boot
    mount -t ntfs-3g -o defaults,noauto,noatime,discard,nofail,users,windows_names,utf8,nls=utf8,x-systemd.idle-timeout=10min /dev/sda1 /mnt/hdd
    ```

## Installation

- Basic System

  ```bash
  pacstrap -K /mnt base base-devel linux-lts linux-lts-headers linux-firmware intel-ucode ntfs-3g e2fsprogs btrfs-progs neovim networkmanager efibootmgr efitools sbctl # grub

  genfstab -U /mnt >> /mnt/etc/fstab

  cat /mnt/etc/fstab | grep /swap/swapfile none swap defaults 0 0
  ```

- Use refshed mirrorlist

  ```bash
  cp /etc/pacman.d/mirrorlist /mnt/etc/pacman.d/mirrorlist
  ```

- Chroot to new system

  ```bash
  arch-chroot /mnt
  ```

- Set local time zone

  ```bash
  ln -sf /usr/share/zoneinfo/Asia/Tehran /etc/localtime

  hwclock --systohc
  ```

- Generate locale

  ```bash
  sed -i '/en_US.UTF-8/s/^#\s*//g' /etc/locale.gen

  locale-gen

  echo "LANG=en_US.UTF-8" > /etc/locale.conf
  ```

- Configure network

  ```bash
  hostname=my_bluetooth_name

  echo "${hostname}" >> /etc/hostname
  echo "127.0.0.1 localhost" >> /etc/hosts
  echo "::1  localhost" >> /etc/hosts
  echo "127.0.1.1 ${hostname}.localdomain ${hostname}" >> /etc/hosts
  ```

- Configure TTY Fonts

  This will prevent systemd-vconsole-setup.service logging error message too

  ```bash
  echo "FONT=LatArCyrHeb-14" > /etc/vconsole.conf
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

- To set a loader edit `/boot/loader/loader.conf` and define a default entry

  ```bash
  nvim /boot/loader/loader.conf
  ---------------------
  default arch-lts.conf
  timeout 0
  editor no
  console-mode max
  ```

- Define default and fallback bootloaders

  ```bash
  nvim /boot/loader/entries/arch-lts.conf
  ---------------------
  title Arch Linux LTS
  linux vmlinuz-linux-lts
  initrd intel-ucode.img
  initrd initramfs-linux-lts.img
  ```

- For Ext4

    ```bash
    echo "options root=/dev/nvme0n1p2 rootfstype=ext4 add_efi_memmap" >> arch-lts.conf
    ```

- For BTRFS

    ```bash
    echo "options root=/dev/nvme0n1p2 rootfstype=btrfs rootflags=subvol=@ add_efi_memmap" >> arch-lts.conf
    ```

- Generate a fallback option

    ```bash
    cd /boot/loader/entries
    sed '/title/s/Arch Linux LTS/Fallback/' arch-lts.conf > fallback.conf
    sed -i '/initrd/s/lts.img/lts-fallback.img/' fallback.conf
    ```

- Verify boot-loaders

  ```bash
  bootctl list
  ```

- Secure Boot

  Secure Boot is in Setup Mode when the Platform Key is removed. To put firmware in Setup Mode, enter firmware setup utility and find an option to delete or clear certificates.

  - ***Note**: sbctl does not work with all hardware. How well it will work depends on the manufacturer.*

  ```bash
  sbctl create-keys
  sbctl enroll-keys -m
  sbctl status
  sbctl verify
  sbctl sign -s /boot/vmlinuz-linux
  sbctl sign -s /boot/EFI/BOOT/BOOTX64.EFI
  ...
  sbctl status
  systemctl enable systemd-boot-update.service
  sbctl sign -s -o /usr/lib/systemd/boot/efi/systemd-bootx64.efi.signed /usr/lib/systemd/boot/efi/systemd-bootx64.efi
  ```

- Finish systemd required configs

  ```bash
  nvim /etc/mkinitcpio.conf
  ---------------------
  # in `HOOKS=` replace `udev` with `systemd`
  HOOKS=(base systemd[udev] fsck ...)
  ```

  Then run

  ```bash
  mkinitcpio -P
  ```

</details>

- Configure mkinitcpio

  ```bash
  nvim /etc/mkinitcpio.conf
  ---------------------
  # in MODULES add the following based on the list bellow:
  # BTRFS -> btrfs
  # Ext4 -> ext4
  # intel GPU -> i915
  # AMD GPU -> amdgpu
  # Nvidia GPU -> nvidia nvidia_modeset nvidia_uvm nvidia_drm
  # Nouveau Driver -> nouveau
  MODULES = (...)

  # uncomment and edit `COMPRESSION_OPTIONS=` to save space for custom kernels
  COMPRESSION_OPTIONS=(-v -9 --long)

  # add this line at the end
  MODULES_DECOMPRESS="yes"
  ```

  Then run

  ```bash
  mkinitcpio -P
  ```

- Have boot messages stay on tty1

  ```bash
  nvim /etc/systemd/system/getty@tty1.service.d/noclear.conf
  ---------------------
  [Service]
  TTYVTDisallocate=no
  ```

## Manage Accounts

- Set root password

  ```bash
  passwd
  ```

- Create non-root user

  ```bash
  useradd -m -G wheel,realtime -s /bin/bash username
  passwd username

  EDITOR=nvim visudo
  - Or
  sed -i '/^#\s*%wheel ALL=(ALL:ALL) ALL\s*$/s/^#\s*//' /etc/sudoers

  # for instance to have some user process running without any open session
  # loginctl enable-linger USERNAME
  ```

- Network Preparation

  ```bash
  systemctl enable NetworkManager
  ```

  - then to connect to Wi-Fi run

    ```bash
    nmcli dev wifi connect WIFI_NAME password "network-password"
    ```

- Exit chroot environment

  ```bash
  exit
  ```

- Unmount all partitions

  ```bash
  umount -R /mnt
  ```

- Reboot and Remove the live media

  ```bash
  reboot
  ```

## Additional Packages

Although I’ll explain my suggested packages later, for convenience let’s use Pacman’s
feature to install the packages listed in a text file.

- **You have to generatesthe list to include only the packages you want.*

- Packages from AUR are prefixed with `aur/`, you can use any pacman wrapper of your choice to install them.

- I recommend `pikaur`, it uses the same command as pacman's - obviously the command starts with pikaur
*It's safer to use pikaur to install packages once logged-in with non-root user

```bash
pacman -S --needed - < pkglist.txt
```

- Optional - Install a Pacman Wrapper (AUR helper)

  ```bash
  # non-root user
  sudo pacman -S --needed base-devel git
  cd /tmp
  git clone https://aur.archlinux.org/pikaur.git
  cd pikaur
  makepkg -fsri
  pikaur -S - < aur-pkglist.txt
  ```

### Packages

- Power Management

  ```bash
  powertop
  tcl
  tk
  acpi
  acpid # a flexible and extensible daemon for delivering ACPI events.
  aur/laptop-mode-tools # considered by many to be the de facto utility for power saving
  xss-lock
  $ systemctl enable acpid.service
  ```

- CPU Performance

  ```bash
  x86_energy_perf_policy # On Intel processors, x86_energy_perf_policy can also be used to configure Turbo Boost
  cpupower # a set of userspace utilities designed to assist with CPU frequency scaling
  thermald # a Linux daemon used to prevent the overheating of Intel CPUs
  turbostat # can display the frequency, power consumption, idle status and other statistics of the modern Intel and AMD CPUs
  lm_sensors # Linux monitoring sensors
  i2c-tools # DIMM temperature sensors
  irqbalance # distribute hardware interrupts across processors on a multiprocessor system in order to increase performance
  power-profiles-daemon # handles power profiles (e.g. balanced, power-saver, performance)

  $ systemctl enable cpupower.service
  $ systemctl enable thermald.service
  $ systemctl enable irqbalance.service
  $ systemctl enable power-profiles-daemon.service

  # Note: careful about the following lines
  $ x86_energy_perf_policy --turbo-enable 0
  $ powerprofilesctl set power-saver
  $ nvim /boot/loader/entries/arch-lts.conf
  ---------------------
  options ... cpufreq.default_governor=powersave # performance - balance - powersave
  ```

- Firewall

  ```bash
  ufw
  $ systemctl enable ufw
  $ ufw default deny incoming
  $ ufw default allow outgoing
  $ ufw allow 5353 # to allow zeroconf
  $ ufw enable
  $ ufw logging off
  ```

- Network

  ```bash
  ethtool # Utility for controlling network drivers and hardware
  hblock # Adblocker that creates a hosts file from multiple sources
  ifplugd # A daemon which brings up/down network interfaces upon cable insertion/removal.
  inetutils # A collection of common network programs
  modemmanager # Mobile broadband modem management service
  net-tools # Configuration tools for Linux networking
  network-manager-applet # GUI tool for DEs except KDE, Gnome
  nmap # Utility for network discovery and security auditing
  ntp # Network Time Protocol reference implementation
  openresolv # resolv.conf management framework (resolvconf)
  rsync # A fast and versatile file copying tool for remote and local files
  sshfs # FUSE client based on the SSH File Transfer Protocol
  traceroute # Tracks the route taken by packets over an IP network
  usb_modeswitch # Activating switchable USB devices on Linux
  $ systemctl enable ModemManager.service
  $ sudo hblock
  ```

  - Network Manager VPN Support

    ```bash
    networkmanager-openconnect # for OpenConnect
    networkmanager-openvpn # for OpenVPN
    networkmanager-pptp # for PPTP Client
    networkmanager-strongswan # for strongSwan
    networkmanager-vpnc
    aur/networkmanager-fortisslvpn-git
    aur/networkmanager-iodine-git
    aur/networkmanager-libreswan
    networkmanager-l2tp
    aur/networkmanager-ssh-git
    network-manager-sstp
    ```

- Anonymizing overlay network

  ```bash
  tor
  torsocks
  $ systemctl enable tor.service
  ```

- Ncrypted communication sessions over a computer network

  ```bash
  openssh
  openssl
  ```

- Download Manager

  ```bash
  wget # A network utility to retrieve files from the Web
  curl # command line tool and library for transferring data with URLs
  ```

- Developer Tools

  ```bash
  neovim
  git
  github-cli
  ipython
  pyenv # Easily switch between multiple versions of Python
  ```

- System maintenance

  ```bash
  perl-file-mimeinfo # Determine file type, includes mimeopen and mimetype
  pacman-contrib # this brings `checkupdates` a safe way to check for upgrades
  archlinux-contrib # this brings `checkservices` hecks for processes to be restarted
  htop # Simple, ncurses interactive process viewer
  bat # colorful cat :)
  cpio # tool to copy files into or out of a cpio or tar archive
  cronie # Daemon that runs specified programs at scheduled times and related tools
  enchant # A wrapper library for generic spell checking
  eza # A modern replacement for ls (community fork of exa)
  inxi # A script to get system information
  neofetch # A fast, highly customizable system info script that supports displaying images with w3m
  hwinfo # Powerful hardware detection tool come from openSUSE
  reflector # service will run reflector with the parameters specified in `/etc/xdg/reflector/reflector.conf`
  expect # A tool for automating interactive applications
  dust # directories disk usage display
  duf # Disk Usage/Free Utility
  hdparm
  sdparm # are command line utilities to set and view hardware parameters of hard disk drives. hdparm can also be used as a simple benchmarking tool
  fwupd
  bind
  freeipmi
  hddtemp
  tree
  usbutils
  vulkan-tools
  xorg-xdpyinfo
  xorg-xdriinfo
  gpm # A mouse server for the console and xterm
  hwdata # hardware identification databases
  iniparser # A free stand-alone ini file parsing library written in portable ANSI C
  kernel-modules-hook # Keeps your system fully functional after a kernel upgrade
  libaio # The Linux-native asynchronous I/O facility (aio) library
  libtraceevent # Linux kernel trace event library
  libtracefs # Linux kernel trace file system library
  liburcu # LGPLv2.1 userspace RCU (read-copy-update) library
  libuv # Multi-platform support library with a focus on asynchronous I/O
  libyuv # Library for YUV scaling
  linux-api-headers # Kernel headers sanitized for use in userspace
  logrotate # Rotates system logs automatically
  lsb-release # LSB version query program
  ndctl # Utility library for managing the libnvdimm (non-volatile memory device) sub-system in the Linux kernel
  pkgfile # a pacman .files metadata explorer
  plocate # Alternative to locate, faster and compatible with mlocate's database.
  realtime-privileges # Realtime privileges for users
  rtkit # Realtime Policy and Watchdog Daemon
  sg3_utils # Generic SCSI utilities
  starship # The cross-shell prompt for astronauts
  sysfsutils # System Utilities Based on Sysfs
  systemd-sysvcompat # sysvinit compat for systemd
  tmux # Terminal multiplexer
  tree # A directory listing program displaying a depth indented list of files
  util-linux # provides fstrim.service and fstrim.timer systemd unit files
  profile-sync-daemon # is a tiny pseudo-daemon designed to manage browser profile(s) in tmpfs
  $ psd
  $ systemctl --user enable psd
  $ systemctl enable reflector.service reflector.timer
  $ systemctl enable fstrim.timer
  $ systemctl enable systemd-oomd.service
  ```

- Filesystem Utils

  ```bash
  dosfstools # DOS filesystem utilities
  ecryptfs-utils # Enterprise-class stacked cryptographic filesystem for Linux
  testdisk # provides both TestDisk and PhotoRec
  nfs-utils # Support programs for Network File Systems
  e2fsprogs
  exfatprogs
  f2fs-tools
  fatresize
  fscrypt
  libatasmart
  libnotify
  procps-ng
  xfsprogs
  mtools
  smartmontools # Self-Monitoring, Analysis, and Reporting Technology through which devices monitor, store, and analyze the health of their operation
  $ systemctl enable smartd.service
  ```

- Mounting Devices

  ```bash
  udisks2 # implements D-Bus interfaces used to query and manipulate storage devices
  udiskie # automounter with optional notifications, tray icon and support for password protected LUKS devices
  gvfs # virtual filesystem implementation for GIO
  gvfs-mtp # media players and mobile devices that use MTP
  gvfs-smb
  gvfs-gphoto2 # digital cameras and mobile devices that use PTP
  gvfs-afc # Apple mobile devices
  gvfs-gphoto2 # for having access at least to the photos
  libmtp # is a library MTP implementation, which also comes with some example command-line tools
  mtpfs # based on libmtp, it is a FUSE filesystem that supports reading and writing from any MTP device
  android-udev # This package contains per manufacturer/device udev rules for MTP devices, making it easier to use ADB or MTP
  android-tools # The Android Debug Bridge (ADB) is a command-line tool that can be used to install, uninstall and debug apps, transfer files and access the device's shell
  scrcpy # display and control your Android device
  gparted # partition manager (kde has its own partition manager!)
  gpart # for recovering corrupt partition tables
  xorg-xhost # authorization from wayland
  $ systemctl enable udisks2.service
  ```

- Bluetooth & Network

  ```bash
  bluez
  bluez-utils
  bluez-hid2hci
  bluez-tools
  blueman # for DEs except KDE, Gnome
  $ systemctl enable bluetooth.service
  ```

- Archiving

  ```bash
  unzip # support the use of zipped .zip files
  zip
  unrar
  p7zip
  gzip
  pigz # Parallel implementation of the gzip file compressor
  unarchiver
  xarchiver # GUI tool
  ```

- Sound

  ```bash
  alsa-firmware # contains firmware that may be required for certain sound cards
  alsa-lib # An alternative implementation of Linux sound support
  alsa-oss # If you want OSS applications to work with dmix
  alsa-plugins # if you are getting poor sound quality due to bad resampling + to enable upmixing/downmixing and other advanced features
  alsa-topology-conf # ALSA topology configuration files
  alsa-ucm-conf # ALSA Use Case Manager configuration (and topologies)
  alsa-utils # This contains (among other utilities) the alsamixer(1) and amixer(1) utilities
  sof-firmware # is required for some newer laptop models
  pipewire # a new low-level multimedia framework. It aims to offer capture and playback for both audio and video with minimal latency and support for PulseAudio, JACK, ALSA and GStreamer-based applications
  wireplumber # more powerful Session manager and the current recommendation
  upower
  pipewire-audio
  pipewire-alsa
  pipewire-pulse
  pipewire-jack
  pipewire-zeroconf
  pipewire-v4l2
  slurp
  flac
  wavpack
  lame
  libmad
  libavif # Library for encoding and decoding .avif files
  libcanberra # A small and lightweight implementation of the XDG Sound Theme Specification
  libmanette # Simple GObject game controller library
  pcaudiolib # Portable C Audio Library
  pipewire-x11-bell # Low-latency audio/video router and processor - X11 bell
  tinycompress # ALSA compressed device interface
  $ systemctl --global enable pipewire pipewire-pulse upower
  $ ufw allow 5353
  ```

- Manual pages

  ```bash
  man-db
  man-pages
  ```

- Shell

  ```bash
  bash-completion
  zsh # A very advanced and programmable command interpreter (shell) for UNIX
  zsh-autosuggestions
  zsh-completions
  zsh-history-substring-search
  zsh-syntax-highlighting
  zsh-theme-powerlevel10k
  ```

- Advanced Swapping

  - **Zswap** works in conjunction with regular swap while a **zram** based swap device
  does not require a backing swap device and may work standalone
  (if no swap on hard disk is required, i.e. on SSD or kind of flash memory).

  - **Zswap** is a compressed swap cache in RAM and works as a type of proxy
  for regular swap (in this context also called backing swap device).
  Zswap gets filled up first and evicts pages from compressed cache on an LRU basis
  to the backing swap device when the compressed pool reaches its size limit.
  This not only speeds up swap usage but also reduces hits on backing swap device (i.e. SSD).

  - **Zram** based swap on the other hand works like regular swap
  (but compressed in RAM) without the opportunity to evict pages.
  So it gets filled up gradually until it’s full.
  After that, the next (but probably slower) swap in order (i.e. on hard disk) fills up.
  This way, it is possible to have stored less frequently used memory pages within
  the faster zram based swap, while newer frequently used memory pages get swapped to slower hard disk.

  ```bash
  zram-generator # Systemd unit generator for zram devices

  $ echo "blacklist zswap" > /etc/modprobe.d/disable-zswap.conf

  $ nvim /etc/systemd/zram-generator.conf
  ---------------------
  [zram0]
  zram-size = ram / 2
  compression-algorithm = zstd
  swap-priority = 100
  fs-type = swap

  $ nvim /etc/sysctl.d/99-vm-zram-parameters.conf
  ---------------------
  vm.swappiness = 180
  vm.watermark_boost_factor = 0
  vm.watermark_scale_factor = 125
  vm.page-cluster = 0
  ```

- Intel CPU and Graphics

  ```bash
  adriconf # GUI tool to configure Mesa drivers by setting options and writing them to the standard drirc file
  intel-gpu-tools
  intel-media-driver
  libva
  libva-intel-driver
  libva-mesa-driver
  libva-utils
  libvdpau
  libvdpau-va-gl
  mesa-amber # it's for intel cpu gen7 and older but you may use mesa instead
  mesa-demos
  mesa-utils
  mesa-vdpau
  v4l-utils
  vdpauinfo
  vulkan-icd-loader
  vulkan-intel
  vulkan-mesa-layers
  xf86-video-intel
  ```

- Nvidia Graphics

  - Option One - nouveau:

    ```bash
    xf86-video-nouveau
    xorg-server-devel
    ```

  - Option Two - nvidia:

    ```bash
    dkms
    # nvidia -> kernel
    # nvidia-lts -> kernel-lts
    # nvidia-dkms -> kernel-zen or other kernels
    nvidia-lts
    nvidia-utils
    nvidia-settings
    nvidia-prime
    xorg-server-devel
    xapp
    aur/optimus-manager

    $ echo "options nomodeset i915.modeset=0 nouveau.modeset=0 nvidia-drm.modeset=1" > /etc/modprobe.d/nvidia.conf

    $ nvim /boot/loader/entries/arch-lts.conf
    ---------------------
    options ... loglevel=3 nvidia_drm.modeset=1 video=HDMI-A-1:1920x1080@60D cpufreq.default_governor=powersave

    $ nvim /etc/mkinitcpio.conf
    ---------------------
    # in `MODULES=` and add the following options to:
    MODULES=(... nvidia nvidia_modeset nvidia_uvm nvidia_drm)

    # in HOOKS remove kms

    $ mkinitcpio -P
    $ systemctl enable nvidia-hibernate nvidia-resume  nvidia-suspend
    ```

    *To avoid the possibility of forgetting to update initramfs after an NVIDIA driver upgrade, you may want to use a pacman hook:*
    *Make sure the Target package set in this hook is the one you have installed in steps above (e.g. `nvidia`, `nvidia-dkms`, `nvidia-lts` or `nvidia-ck-something`).*

    ```bash
    # If a different kernel is used change the Target=linux-lts also in the Exec line
    # If a different nvidia package is used change the Target=nvidia-lts

    $ nvim /etc/pacman.d/hooks/nvidia.hook
    ---------------------
    [Trigger]
    Operation=Install
    Operation=Upgrade
    Operation=Remove
    Type=Package
    Target=nvidia-lts
    Target=linux-lts

    [Action]
    Description=Update NVIDIA module in initcpio
    Depends=mkinitcpio
    When=PostTransaction
    NeedsTargets
    Exec=/bin/sh -c 'while read -r trg; do case $trg in linux-lts) exit 0; esac; done; /usr/bin/mkinitcpio -P'
    ```

    - To use optimus-manager for the first time, call `prime-offload` then do e.g `optimus-manager --switch MODE`

- Multimedia codecs & player

  ```bash
  ffmpeg
  gst-libav
  gst-plugin-pipewire
  gst-plugins-bad
  gst-plugins-base
  gst-plugins-base-libs
  gst-plugins-good
  gst-plugins-ugly
  gstreamer
  gstreamer-vaapi
  mplayer # because pulls in a large number of codecs as dependencies, and also has codecs built in
  ```

  - SMPlayers

    ```bash
    smplayer
    smplayer-skins
    smplayer-themes
    phonon-qt5-gstreamer # Phonon GStreamer backend for Qt5
    ```

- Text Editor & Office

  ```bash
  code # Code OSS - The Open Source build of Visual Studio Code (vscode) editor
  aur/code-marketplace # Enable vscode marketplace in Code OSS
  # Or
  aur/visual-studio-code-bin # Visual Studio Code (vscode)

  libreoffice-fresh # The office productivity suite compatible to the open and standardized ODF document format
  pdfslicer # Simple application to extract, merge, rotate and reorder pages of PDF documents
  ```

- Fonts

  ```bash
  awesome-terminal-fonts # icon package
  harfbuzz-icu # OpenType text shaping engine - ICU integration
  terminus-font # Monospace bitmap font (for X11 and console)
  ttf-cascadia-code-nerd # Patched font Cascadia Code (Caskaydia) from nerd fonts library
  ttf-hack-nerd # Patched font Hack from nerd fonts library
  noto-fonts
  noto-fonts-emoji
  woff2 # Web Open Font Format 2 reference implementation
  ```

### ACPID

acpid generates events for some ordinary key presses, such as arrow keys. This results in event/handler spam, visible in system logs or top. Events for these buttons can be dropped in the configuration file:

```bash
nvim /etc/acpi/events/buttons
---------------------
event=button/(up|down|left|right|kpenter)
action=<drop>
```

### Advanced Sound

```bash
nvim /etc/modprobe.d/alsa-base.conf
---------------------
options snd_mia index=0
options snd_hda_intel index=1


nvim /etc/modules-load.d/seq-pcm-mixer.conf
---------------------
snd-seq-oss
snd-pcm-oss
snd-mixer-oss
```

By default, audio power saving is turned off by most drivers. It can be enabled by setting the power_save parameter; a time (in seconds) to go into idle mode. To idle the audio card after one second, create the following file for Intel soundcards

```bash
nvim /etc/modprobe.d/audio_powersave.conf
---------------------
options snd_hda_intel power_save=1
```

It is also possible to further reduce the audio power requirements by disabling the HDMI audio output, which can done by blacklisting the appropriate kernel modules (e.g. snd_hda_codec_hdmi in case of Intel hardware).

If you will not use integrated web camera then blacklist the uvcvideo module.

```bash
nvim /etc/modprobe.d/blacklist.conf
---------------------
blacklist snd_hda_codec_hdmi
blacklist uvcvideo
```

- Temporarily enable or disable the webcam

    ```bash
    modprobe uvcvideo
    modprobe -r uvcvideo
    ```

### Toggle external monitor

This script toggles between an external monitor (specified by $extern) and a default monitor (specified by $intern), so that only one monitor is active at a time.

The default monitor should be connected when running the script, which is always true for a laptop.

- Note: To leave the default monitor enabled when an external monitor is connected, in the else clause comment the first line and uncomment the next line.

```bash
nvim /usr/bin/multiple-monitors.sh
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

then run 

```bash
sudo chmod +x /usr/bin/multiple-monitors.sh  
```

### Optional Firmwares

```bash
aur/mkinitcpio-firmware # The meta-package that contains most optional firmwares
```

- that includes firmwares listed bellow including `linux-firmware-mellanox` too.

```bash
linux-firmware
linux-firmware-qlogic
linux-firmware-bnx2x
linux-firmware-liquidio
linux-firmware-nfp

aur/ast-firmware
aur/aic94xx-firmware
aur/wd719x-firmware
aur/upd72020x-fw
```

### Desktops

- [KDE/Plasma](./plasma.md)
- [XFCE](./xfce.md)

## Further Reading

<https://wiki.archlinux.org/title/Core_utilities#find_alternatives>
