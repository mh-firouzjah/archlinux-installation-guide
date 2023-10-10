# Arch Linux Installaion Guide

In this content I'll explain how to install Arch Linux (64-bit) using systemd-boot and Ext4 or BTRFS filesystem and booting in UEFI mode


## Prepare an installation medium

- Acquire an installation ISO image

  Download the iso image from the offical website at https://archlinux.org/download/

- Verify signature

  - Download the ISO PGP signature under Checksums in the Download Page at https://archlinux.org/download/#checksums

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

- Partition the disks 

  - Since we are using UEFI boot, we need to create a GPT partition table and also we need an efi partition. it's easier to consider `/boot` for that efi partition.

  - You can use 1 GiB size for `/boot` to be on the safe side.
  for example adding nvidia mudole to mkinitcpio will increase ~=50-60mb to each initramfs and it's fallback.

  - Since `/var` is frequently read or written, it is recommended that you consider the location of this partition on a spinning disk.

    - With that being said, newer SSDs, especially those with higher capacities, have a longer lifespan (support lots of writes and erases e.g 20g/dey for near 10 years).

  - If you want to go with BTRFS, consider creating subvolumes for other directories that contain data you do not want to include in snapshots and rollbacks of the `@` subvolume, such as `/var/cache`, `/var/spool`, `/var/tmp`, `/var/lib/machines` (systemd-nspawn), `/var/lib/docker` (Docker), `/var/lib/postgres` (PostgreSQL), and other data directories under `/var/lib/`. It is up to you if you want to follow the flat layout or create nested subvolumes. On the other hand, the pacman database in `/var/lib/pacman` **_must_** stay on the root subvolume (`@`).

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

  - Format EFI parition

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
    mount /dev/nvme0n1p1 /mnt/boot
    mount -t ntfs3 -o rw,nls=utf8,noatime,windows_names,x-systemd.automount,x-systemd.idle-timeout=10min /dev/sda1 /mnt/hdd
    ```

- Install basic system

  ```bash
  pacstrap -K /mnt base base-devel linux-lts linux-lts-headers linux-firmware intel-ucode ntfs-3g e2fsprogs btrfs-progs neovim networkmanager efibootmgr efitools sbctl

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

  echo "LANGE=en_US.UTF-8" > /etc/locale.conf
  ```

- Configure network

  ```bash
  hostname=my_bluetooth_name

  echo "${hostname}" >> /etc/hostname
  echo "127.0.0.1 localhost" >> /etc/hosts
  echo "::1  localhost" >> /etc/hosts
  echo "127.0.1.1 ${hostname}.localdomain ${hostname}" >> /etc/hosts
  ```

- Configure bootloader

  ```bash
  bootctl install --efi-boot-option-description="Arch Linux LTS"

  nvim /boot/loader/loader.conf
  ---------------------
  default arch-lts.conf
  timeout 0
  editor no
  console-mode max

  nvim /boot/loader/entries/arch-lts.conf
  ---------------------
  title Arch Linux LTS
  linux vmlinuz-linux-lts
  initrd intel-ucode.img
  initrd initramfs-linux-lts.img
  ```

  - For Ext4

    - *Note:* Because this loader entry will mount the root partition then the `fstab` line regarding the mount of root partition will remount that partition so I wrote all mount options here as `rootflags` and I removed that line from `/etc/fstab`, I don't know if it's a bad idea or not.

    ```bash
    echo "options root=/dev/nvme0n1p2 rootfstype=ext4 rootflags=rw,noatime,discard,journal_checksum,commit=120,min_batch_time=200,auto_da_alloc add_efi_memmap" >> arch-lts.conf
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

- Verify bootloaderes

  ```bash
  bootctl list
  ```

- Secure Boot

Secure Boot is in Setup Mode when the Platform Key is removed. To put firmware in Setup Mode, enter firmware setup utility and find an option to delete or clear certificates.

  - *__Note__: sbctl does not work with all hardware. How well it will work depends on the manufacturer.*

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
  # in MODULES add the following based on the list bellow:
  # BTRFS -> btrfs
  # Ext4 -> ext4
  # intel GPU -> i915
  # AMD GPU -> amdgpu
  # Nvidia GPU -> nvidia nvidia_modeset nvidia_uvm nvidia_drm
  # Nouveau Driver -> nouveau
  MODULES = (...)

  # in `HOOKS=` replace `udev` with `systemd`
  HOOKS=(base systemd[udev] fsck ...)

  # uncomment and edit `COMPRESSION_OPTIONS=` to save space for custom kernels
  COMPRESSION_OPTIONS=(-v -5 --long)
  
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
     nmcli dev wlan0 connect WIFI_NAME password "network-password"
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

Although I’ll explain my suggested packages later, for convenience let’s use Pacman’s feature to install the packages listed in a text file.

- **You may edit the list to include only the packages you want.*

- Packages from AUR are prefixed with `aur/`, but their listed in `aur-pkglist.txt` and not in `pkglist.txt` and ou can use any pacman wrapper of your choice to install them.

- I recmmend `pikaur`, it uses the same command as pacman's - obviously the command starts with pikaur ;)
*It's safer to use pikaur to install packages once loged-in with non-root user


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
  power-profiles-daemon # handles power profiles (e.g. balanced, power-saver, performance)
  tcl
  tk
  acpi
  acpid # a flexible and extensible daemon for delivering ACPI events.
  aur/laptop-mode-tools # considered by many to be the de facto utility for power saving
  xss-lock
  $ systemctl enable power-profiles-daemon.service
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
  $ x86_energy_perf_policy --turbo-enable 0
  $ systemctl enable cpupower.service
  $ systemctl enable thermald.service
  $ systemctl enable irqbalance.service
  ```


- Firewall

  ```bash
  ufw
  $ systemctl enable ufw
  $ ufw default deny incoming
  $ ufw default allow outgoing
  $ ufw allow 5353 # to allow zeroconf
  $ ufw enable
  ```


- Network Manager

  ```bash
  geoip # Non-DNS IP-to-country resolver C library & utils
  geoip-database # GeoIP legacy country database (based on GeoLite2 data created by MaxMind)
  gnu-netcat # GNU rewrite of netcat, the network piping application
  hblock # Adblocker that creates a hosts file from multiple sources
  modemmanager
  usb_modeswitch # Activating switchable USB devices on Linux
  wireless_tools
  wireless-regdb
  ifplugd # A daemon which brings up/down network interfaces upon cable insertion/removal.
  inetutils # A collection of common network programs
  libmaxminddb # MaxMindDB GeoIP2 database library
  libomxil-bellagio # An opensource implementation of the OpenMAX Integration Layer API
  libsoup # HTTP client/server library for GNOME
  net-tools # Configuration tools for Linux networking
  nmap # Utility for network discovery and security auditing
  ntp # Network Time Protocol reference implementation
  openresolv # resolv.conf management framework (resolvconf)
  rsync # A fast and versatile file copying tool for remote and local files
  socat # Multipurpose relay
  sshfs # FUSE client based on the SSH File Transfer Protocol
  thin-provisioning-tools # Suite of tools for manipulating the metadata of the dm-thin device-mapper target
  traceroute # Tracks the route taken by packets over an IP network
  nfs-utils # Support programs for Network File Systems
  perl-clone # Recursive copy of nested objects.
  perl-file-listing # parse directory listing
  perl-file-mimeinfo # Determine file type, includes mimeopen and mimetype
  perl-html-parser # Perl HTML parser class
  perl-html-tagset # Data tables useful in parsing HTML
  perl-http-cookies # HTTP cookie jars
  perl-http-daemon # Simple http server class
  perl-http-date # Date conversion routines
  perl-http-message # HTTP style messages
  perl-http-negotiate # Choose a variant to serve
  perl-io-html # Open an HTML file with automatic charset detection
  perl-libwww # The World-Wide Web library for Perl
  perl-lwp-mediatypes # Guess the media type of a file or a URL
  perl-net-http # Low-level HTTP connection (client)
  perl-try-tiny # Minimal try/catch with proper localization of $@
  perl-www-robotrules # Database of robots.txt-derived permissions
  perl-xml-parser # Expat-based XML parser module for perl
  perl-xml-writer # Module for writing XML documents
  rp-pppoe # Roaring Penguin's Point-to-Point Protocol over Ethernet client
  ethtool # Utility for controlling network drivers and hardware
  $ systemctl enable ModemManager.service
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
  bash-completion
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
  solid # Hardware integration and detection
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
  android-file-transfer
  mtpfs # based on libmtp, it is a FUSE filesystem that supports reading and writing from any MTP device
  android-udev # This package contains per manufacturer/device udev rules for MTP devices, making it easier to use ADB or MTP
  android-tools # The Android Debug Bridge (ADB) is a command-line tool that can be used to install, uninstall and debug apps, transfer files and access the device's shell
  scrcpy # display and control your Android device
  gparted # partition manager (kde has its own partition manager!)
  gpart # for recovering corrupt partition tables
  xorg-xhost # authorization from wayland
  $ systemctl enable udisks2.service
  ```


- Bluetooth

  ```bash
  bluez
  bluez-utils
  bluez-hid2hci
  bluez-tools
  $ systemctl enable bluetooth.service
  ```


- User Privilege

  ```bash
  polkit # for defining and handling the policy that allows unprivileged processes to speak to privileged processes
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
  $ systemctl --global enable pipewire.service pipewire-pulse.service
  $ ufw allow 5353
  ```


- Manual pages

  ```bash
  man-db
  man-pages
  ```


- Shell

  ```bash
  zsh # A very advanced and programmable command interpreter (shell) for UNIX
  zsh-autosuggestions
  zsh-completions
  zsh-history-substring-search
  zsh-syntax-highlighting
  zsh-theme-powerlevel10k
  ```


- Advanced Swapping

  - **Zswap** works in conjunction with regular swap while a **zram** based swap device does not require a backing swap device and may work standalone (if no swap on hard disk is required, i.e. on SSD or kind of flash memory).

  - **Zswap** is a compressed swap cache in RAM and works as a type of proxy for regular swap (in this context also called backing swap device). Zswap gets filled up first and evicts pages from compressed cache on an LRU basis to the backing swap device when the compressed pool reaches its size limit. This not only speeds up swap usage but also reduces hits on backing swap device (i.e. SSD).

  - **Zram** based swap on the other hand works like regular swap (but compressed in RAM) without the opportunity to evict pages. So it gets filled up gradually until it’s full. After that, the next (but probably slower) swap in order (i.e. on hard disk) fills up. This way, it is possible to have stored less frequently used memory pages within the faster zram based swap, while newer frequently used memory pages get swapped to slower hard disk.

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

### GUI Evironment


- Not for a specific DE

  ```bash
  xorg-server # the most popular display server among Linux users
  xorg-xwayland
  xorg-xeyes
  appmenu-gtk-module # to integrate application menus with the desktop environment's global menu bar
  wpebackend-fdo # Freedesktop.org backend for WPE WebKit
  xdg-dbus-proxy # Filtering proxy for D-Bus connections
  xdg-user-dirs # Manage user directories like ~/Desktop and ~/Music
  xdg-utils # Command line tools that assist applications with a variety of desktop integration tasks
  xorg-xinit # X.Org initialisation program
  xsel # Command-line program for getting and setting the contents of the X selection
  gtk3
  gtk2
  ```


- Monitor Gama & Color Adjustment

  ```bash
  xcalib # Monitor calibration loader
  xorg-xgamma # Alter a monitor's gamma correction
  acpilight
  xorg-xbacklight
  ddcutil
  ```


- Intel CPU and Graphics

  ```bash
  xf86-video-intel
  mesa-amber # it's for intel cpu gen7 and older but you may use mesa instead
  mesa-utils
  vulkan-intel
  intel-media-driver
  libva-intel-driver
  v4l-utils
  vdpauinfo
  vulkan-icd-loader
  vulkan-mesa-layers
  libva
  libva-mesa-driver
  libva-utils
  libvdpau
  libvdpau-va-gl
  mesa-demos
  intel-gpu-tools
  mesa-vdpau
  adriconf # GUI tool to configure Mesa drivers by setting options and writing them to the standard drirc file
  ```


- Nvidia Graphics

  - Option One - nouveau:

    ```bash
    xf86-video-nouveau
    xorg-server-devel
    ```

  - Option Two - nvidia:

    ```bash
    nvidia-lts
    nvidia-utils
    nvidia-settings
    nvidia-prime
    xorg-server-devel

    $ echo "options nomodeset i915.modeset=0 nouveau.modeset=0 nvidia-drm.modeset=1" > /etc/modprobe.d/nvidia.conf
    $ nvim /etc/mkinitcpio.conf
    ---------------------
    # in `MODULES=` and add the following options to:
    MODULES=(... nvidia nvidia_modeset nvidia_uvm nvidia_drm)

    $ mkinitcpio -P
    ```

    *To avoid the possibility of forgetting to update initramfs after an NVIDIA driver upgrade, you may want to use a pacman hook:*
    *Make sure the Target package set in this hook is the one you have installed in steps above (e.g. `nvidia`, `nvidia-dkms`, `nvidia-lts` or `nvidia-ck-something`).*


    ```bash
    $ nvim /etc/pacman.d/hooks/nvidia.hook
    ---------------------
    [Trigger]
    Operation=Install
    Operation=Upgrade
    Operation=Remove
    Type=Package
    Target=nvidia-lts
    Target=linux-lts
    # Change the `Target=linux` part above and in the Exec line if a different kernel is used
    # Change the `Target=nvidia` part above if a different nvidia is used

    [Action]
    Description=Update NVIDIA module in initcpio
    Depends=mkinitcpio
    When=PostTransaction
    NeedsTargets
    Exec=/bin/sh -c 'while read -r trg; do case $trg in linux-lts) exit 0; esac; done; /usr/bin/mkinitcpio -P'
    ```


- Input devices (Mouse/Keyboard)

  ```bash
  libinput # libinput is a library to handle input devices
  xf86-input-libinput # libinput is a library to handle input devices
  xorg-xinput # On-demand disabling and enabling of input sources
  xorg-xset
  xorg-xkill # Killing an application visually
  xclip
  ```

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
  smplayer # I love it :)
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

  libreoffice-still # The office productivity suite compatible to the open and standardized ODF document format
  pdfslicer # Simple application to extract, merge, rotate and reorder pages of PDF documents
  ```

- Fonts

  ```bash
  harfbuzz-icu # OpenType text shaping engine - ICU integration
  icu # International Components for Unicode library
  otf-ipafont # Japanese outline fonts by Information-technology Promotion Agency, Japan (IPA)
  terminus-font # Monospace bitmap font (for X11 and console)
  ttf-cascadia-code-nerd # Patched font Cascadia Code (Caskaydia) from nerd fonts library
  ttf-hack-nerd # Patched font Hack from nerd fonts library
  woff2 # Web Open Font Format 2 reference implementation
  ```


### KDE Applications

```bash
layer-shell-qt # Qt component to allow applications to make use of the Wayland wl-layer-shell protocol
ark # Archiving tool included in the KDE desktop
aur/isoimagewriter # Tool to write a .iso file to a USB disk
aur/kcm-polkit-kde-git # Set of configuration modules which allows administrator to change PolicyKit settings
aur/systemd-kcm # systemd control module for KDE
bluedevil # KDE's Bluetooth tool
breeze # Plasma's Breeze theme
breeze-gtk # a GTK theme designed to mimic the appearance of Plasma's Breeze theme
cifs-utils # to make it look to Plasma like if the SMB share was just a normal local folder
discover # KDE GUI Pkage Manager
dolphin # File manager included in the KDE desktop
dolphin-plugins # uses udisksctl to either mount or unmount the iso in Dolphin
drkonqi # The KDE crash handler
falkon # Cross-platform QtWebEngine browser
ffmpegthumbs # provides video thumbnailing plugin
filelight # Disk usage analyzer by KDE
ghostwriter # Distraction-free Markdown editor
gwenview # Fast and easy to use image viewer for the KDE desktop
kaccounts-providers # Online account providers for the KAccounts system
kactivitymanagerd # System service to manage user activities and track the usage patterns
kate # Full-featured programmer's editor for the KDE desktop with MDI and a filesystem browser
kcalc # Scientific calculator included in the KDE desktop
kcolorlhooser # Simple application to select the color from the screen or from a pallete
kcron # Tool for KDE to run applications in the background at regular intervals
kde-cli-tools # Tools based on KDE Frameworks 5 to better interact with the system
kde-gtk-config # GTK2 and GTK3 Configurator for KDE
kdeconnect # Provides integration between devices
kdecoration # Plugin based library to create window decorations
kdegraphics-thumbnailers # provides PDF thumbnailing plugin, among others
kdenetwork-filesharing # Windows File and printer sharing for KDE. provides the Samba file sharing setup wizard
kdeplasma-addons # All kind of addons to improve your Plasma experience
kdesdk-thumbnailers # Plugins for the thumbnailing system
kdf # Displays information about hard disks and other storage devices
kdialog # A utility for displaying dialog boxes from shell scripts
kexi # Visual database applications creator tool by KDE
keysmith # OTP generation software by KDE
kfind # Search tool for KDE to find files by name, type or content
kgamma5 # Adjust your monitor gamma settings
kget # Download manager for KDE
khotkeys
kimageformats # Image format plugins for Qt5
kinfocenter # Centralized and convenient overview of system information for KDE
kinit # Process launcher to speed up launching KDE applications
kio # Resource and network access abstraction
kio-admin # provides a safe way to edit files as root
kio-extras # MTP support is included in
kio-fuse # FUSE interface for KIO
kio-gdrive # provides transparent KIO access to Google Drive
kio-zeroconf # Network Monitor for DNS-SD services (Zeroconf)
kjournald # Framework for interacting with systemd-journald
kmenuedit # KDE menu editor
kompare # GUI front-end program for viewing and merging differences between source files with many options to customize the information
konsole # Terminal emulator included in the KDE desktop
kpat # Offers a selection of solitaire card games
kpipewire
krita # Digital painting and illustration software included based on the KDE platform
kscreen
kscreenlocker
kshutdown # Graphical shutdown utility, which allows you to turn off or suspend a computer at a specified time
ksshaskpass # ssh-add helper that uses kwallet and kpassworddialog
ksysguard # System monitor for KDE to monitor running processes and system performance.
ksystemlog # System log viewer tool for KDE
ksystemstats # A plugin based system monitoring daemon
ktimer # Little tool for KDE to execute programs after some time
ktouch # Program to learn and practice touch typing
kwallet-pam
kwalletmanager # Tool to manage the passwords on your system
kwayland-integration # Provides integration plugins for various KDE frameworks for the wayland windowing system
kwin
kwrited # KDE daemon listening for wall and write messages
layer-shell-qt # Qt component to allow applications to make use of the Wayland wl-layer-shell protocol
libappindicator-gtk2 # icons in system tray
libappindicator-gtk3 # an attempt to get clear icons
libkscreen
libksysguard
libqtxdg
markdownpart # KPart for rendering Markdown content (kate/okular markdown preview)
milou # A dedicated search application built on top of Baloo
noto-fonts
noto-fonts-emoji
okular # Universal document viewer for KDE
oxygen
oxygen-sounds
packagekit-qt5 # to manage packages from Arch Linux repositories (not recommended, use at your own risk)
partitionmanager # Utility to help you manage the disks, partitions, and file systems on your computer
phonon-qt5-gstreamer # Phonon is the multimedia API provided by KDE and is the standard abstraction for handling multimedia streams within KDE software
plasma-browser-integration
plasma-desktop
plasma-disks # Hard disk health monitoring for KDE Plasma
plasma-firewall
plasma-integration
plasma-nm
plasma-pa
plasma-vault
plasma-systemmonitor # Advanced and customizable system monitor for KDE
plasma-wayland-session
plasma-workspace
plasma5-applets-thermal-monitor # KDE Plasma applet for monitoring CPU, GPU and other available temperature sensors
polkit-kde-agent # Daemon providing a polkit authentication UI for KDE
powerdevil # for an integrated Plasma power managing service
qt5-imageformats # .webp, .tiff, .tga, .jp2 files
qt5-virtualkeyboard
qt5-wayland
qt5ct # Icons
qt6-imageformats # Plugins for additional image formats: TIFF, MNG, TGA, WBMP
qt6-shadertools # Provides functionality for the shader pipeline that allows Qt Quick to operate on Vulkan, Metal, and Direct3D, in addition to OpenG
qt6-wayland
qtxdg-tools # libqtxdg user tools
sddm # simple display manager
sddm-kcm # KDE Configuration Module for SDDM
spectacle # screenshot app
sweeper # System cleaning utility for KDE
systemdgenie # a graphical frontend for systemctl
systemsettings # KDE system manager for hardware, software, and workspaces
svgpart # A KPart for viewing SVGs (kate/okular svg preview)
taglib # Audio files
xdg-desktop-portal-kde # To use remote input functionality on a Plasma Wayland session/Desktop integration portals for sandboxed apps
# breeze-plymouth
# flatpak-kcm # Flatpak Permissions Management KCM for discover
# plasma-sdk
# plasma-thunderbolt
# plasma-welcome
# plasma-workspace-wallpapers
# plymouth-kcm
```

## Troubleshooting

### SDDM: Touchpad Tapping

```bash
nvim /etc/X11/xorg.conf.d/30-touchpad.conf
---------------------
Section "InputClass"
    Identifier "touchpad"
    Driver "libinput"
    MatchIsTouchpad "on"
    Option "Tapping" "on"
    Option "TappingButtonMap" "lmr"
EndSection
```


### ACPID:

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
nvim /usr/share/sddm/scripts/Xsetup
---------------------
intern=LVDS1
extern=VGA1

if xrandr | grep "$extern disconnected"; then
    xrandr --output "$extern" --off --output "$intern" --auto
else
    xrandr --output "$intern" --off --output "$extern" --auto
    # xrandr --output "$intern" --primary --auto --output "$extern" --right-of "$intern" --auto
fi
```

### Optional Firmwares

```bash
aur/mkinitcpio-firmware # The meta-package that contains most optional firmwares
```
that includes firmwares listed bellow including `linux-firmware-mellanox` too.

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

## Further Reading

https://wiki.archlinux.org/title/Core_utilities#find_alternatives
