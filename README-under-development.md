# Arch Linux Systemd-boot BTRFS

## Preparation


- Download the iso at

    https://archlinux.org/download/

- Validation check

  ```bash
  gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig
  ```

  - Or

    ```bash
    pacman-key -v archlinux-version-x86_64.iso.sig
    ```

- Burn to CD/DVD or make a bootable USB

  ```bash
  dd bs=4M if=path/to/archlinux-version-x86_64.iso of=/dev/sdx conv=fsync oflag=direct status=progress
  ```


## Live Environment


- Connect to wi-fi (cable connection doesn't require user action)

  ```bash
  lwctl --passphrase PASSWORD station wlan0 connect WIFINAME
  ```

- Check internet connection

  ```bash
  ping -c 4 archlinux.org
  ```

- Ensure the system clock is accurate

  ```bash
  timedatectl
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
  ls /sys/firmware/efi/efivars
  ```

- Partitioning (recommended to mount `/var` on a spinning disk)

  ```sh
  fdisk -l
  fdisk /dev/nvme0n1

  g
  n
  enter
  enter
  +1G
  n
  enter
  enter
  enter
  t
  1
  1
  p
  w

  mkfs.fat -F 32 -n EFI /dev/nvme0n1p1
  mkfs.btrfs -f -L ROOT /dev/nvme0n1p2

  mount /dev/nvme0n1p2 /mnt

  btrfs su cr /mnt/@
  btrfs su cr /mnt/@root
  btrfs su cr /mnt/@home
  btrfs su cr /mnt/@swap

  btrfs su li /mnt

  umount /mnt

  mop="ssd,discard=async,noatime,compress-force=zstd:8,commit=120,space_cache=v2,autodefrag,max_inline=512k,inode_cache,subvol=@"

  mount -t btrfs -o ${mop} /dev/nvme0n1p2 /mnt
  mkdir -p /mnt/{boot,root,home,swap,hdd}
  mount -t btrfs -o ${mop}root /dev/nvme0n1p2 /mnt/root
  mount -t btrfs -o ${mop}home /dev/nvme0n1p2 /mnt/home
  mount /dev/nvme0n1p1 /mnt/boot
  mount -t ntfs3 -o rw,nls=utf8,noatime,windows_names,x-systemd.automount,x-systemd.idle-timeout=10min /dev/sda1 /mnt/hdd

  # swapfile
  mount -t btrfs -o defaults,subvol=@swap /dev/nvme0n1p2 /mnt/swap
  btrfs filesystem mkswapfile --size 16g --uuid clear /mnt/swap/swapfile
  swapon /mnt/swap/swapfile
```

- Install basic system

  ```bash
  pacstrap -K /mnt base base-devel llinux-lts linux-lts-headers inux-firmware intel-ucode ntfs-3g btrfs-progs duperemove neovim networkmanager efibootmgr efitools sbctl

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
  linux /vmlinuz-linux-lts
  initrd /intel-ucode.img
  initrd /initramfs-linux-lts.img
  options root=/dev/nvme0n1p2 rootfstype=btrfs rootflags=subvol=@ rw bgrt_disable add_efi_memmap

  nvim /boot/loader/entries/fallback.conf
  ---------------------
  title Fallback
  linux /vmlinuz-linux-lts
  initrd /intel-ucode.img
  initrd /initramfs-linux-lts-fallback.img
  options root=/dev/nvme0n1p2 rootfstype=btrfs rootflags=subvol=@ rw bgrt_disable add_efi_memmap
  ```

- Verify bootloader

  ```bash
  bootctl list
  ```

- Secure Boot

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

  # uncomment and edit `COMPRESSION_OPTIONS=` to save space for custom kernels
  COMPRESSION_OPTIONS=(-v -5 --long)

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

  sed -i '/^#\s*%wheel ALL=(ALL:ALL) ALL\s*$/s/^#\s*//' /etc/sudoers

  # for instance to have some user process running without any open session
  # $ loginctl enable-linger USERNAME
  ```

- Network Preparation

  ```bash
  $ systemctl enable NetworkManager.service
  $ systemctl enable NetworkManager-wait-online.service
  $ systemctl enable NetworkManager-dispatcher.service
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

## List of Packages

- Power Management

  ```bash
  powertop
  power-profiles-daemon # handles power profiles (e.g. balanced, power-saver, performance)
  $ systemctl enable power-profiles-daemon.service
  acpi
  tcl
  tk
  acpid # a flexible and extensible daemon for delivering ACPI events.
  $ systemctl enable acpid.service
  aur/laptop-mode-tools # considered by many to be the de facto utility for power saving
  xss-lock
  ```

- CPU Performance

  ```bash
  x86_energy_perf_policy # On Intel processors, x86_energy_perf_policy can also be used to configure Turbo Boost
  $ x86_energy_perf_policy --turbo-enable 0
  cpupower # a set of userspace utilities designed to assist with CPU frequency scaling
  $ systemctl enable cpupower.service
  thermald # a Linux daemon used to prevent the overheating of Intel CPUs
  $ systemctl enable thermald.service
  turbostat # can display the frequency, power consumption, idle status and other statistics of the modern Intel and AMD CPUs
  lm_sensors # Linux monitoring sensors
  i2c-tools # DIMM temperature sensors
  irqbalance # distribute hardware interrupts across processors on a multiprocessor system in order to increase performance
  systemctl enable irqbalance.service
  ```

### Network Packages

- Firewall

  ```bash
  ufw
  $ systemctl enable ufw
  $ ufw default deny incoming
  $ ufw default allow outgoing
  $ ufw enable
  ```

- Network Manager

  ```bash
  modemmanager
  usb_modeswitch # Activating switchable USB devices on Linux
  $ systemctl enable ModemManager.service
  rp-pppoe
  ```

  - VPN Support

    ```bash
    networkmanager-openconnect # for OpenConnect
    networkmanager-openvpn # for OpenVPN
    networkmanager-pptp # for PPTP Client
    networkmanager-strongswan # for strongSwan
    networkmanager-vpnc
    networkmanager-fortisslvpn-git # AUR
    networkmanager-iodine-git # AUR
    networkmanager-libreswan # AUR
    networkmanager-l2tp
    networkmanager-ssh-git # AUR
    network-manager-sstp
    ```

- Anonymizing overlay network

  ```bash
  tor
  torsocks
  ```

- Ncrypted communication sessions over a computer network

  ```bash
  openssh
  ```

- Download Manager

  ```bash
  wget # A network utility to retrieve files from the Web
  ```

- Text WebBrowser

  ```bash
  ELinks # Advanced and well-established feature-rich text mode web browser with mouse wheel scroll support, frames and tables, extensible with Lua & Guile (links fork).
  ```

<!--
- !! Avahi Daemom

  - disabling systemd-resolved may require some other replacement

  ```bash
  $ systemctl disable systemd-resolved.service
  avahi # a free Zero-configuration networking (zeroconf) implementation, including a system for multicast DNS/DNS-SD service discovery
  nss-mdns # Hostname resolution
  ## edit the file /etc/nsswitch.conf and change the hosts line to include mdns_minimal [NOTFOUND=return] before resolve and dns
  ## hosts: mymachines mdns_minimal [NOTFOUND=return] resolve [!UNAVAIL=return] files myhostname dns
  $ ufw allow 5353
  cp /usr/share/doc/avahi/ssh.service /etc/avahi/services/
  $ systemctl enable avahi-daemon.service
  ```
-->

- Developement packages

  ```bash
  git
  github-cli
  ipython
  ```

- System maintenance

  ```bash
  pacman-contrib # this brings `checkupdates` a safe way to check for upgrades
  archlinux-contrib # this brings `checkservices` hecks for processes to be restarted
  htop # Simple, ncurses interactive process viewer
  btop # Htop but more lightweight with more features
  inxi # A script to get system information
  neofetch # A fast, highly customizable system info script that supports displaying images with w3m
  hwinfo # Powerful hardware detection tool come from openSUSE
  reflector # service will run reflector with the parameters specified in `/etc/xdg/reflector/reflector.conf`
  $ systemctl enable reflector.service reflector.timer
  testdisk # provides both TestDisk and PhotoRec
  e2fsprogs
  smartmontools # Self-Monitoring, Analysis, and Reporting Technology through which devices monitor, store, and analyze the health of their operation
  $ systemctl enable smartd.service
  libatasmart
  libnotify
  procps-ng
  dust # disk usage display
  fwupd
  $ systemctl enable systemd-oomd.service
  profile-sync-daemon # is a tiny pseudo-daemon designed to manage browser profile(s) in tmpfs
  $ psd
  $ systemctl --user enable psd
   util-linux # provides fstrim.service and fstrim.timer systemd unit files
   $ systemctl enable fstrim.timer
   hdparm
   sdparm # are command line utilities to set and view hardware parameters of hard disk drives. hdparm can also be used as a simple benchmarking tool.


  ```

- Mounting Devices

  ```bash
  udisks2 # implements D-Bus interfaces used to query and manipulate storage devices
  $ systemctl enable udisks2.service
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
  ```

- Bluetooth

  ```bash
  bluez
  bluez-utils
  bluez-hid2hci
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
  unarchiver
  ```

- Sound

  ```bash
  alsa-utils # This contains (among other utilities) the alsamixer(1) and amixer(1) utilities
  alsa-plugins # to enable upmixing/downmixing and other advanced features
  sof-firmware # is required for some newer laptop models
  alsa-firmware # contains firmware that may be required for certain sound cards
  alsa-oss # If you want OSS applications to work with dmix
  alsa-plugins # if you are getting poor sound quality due to bad resampling
  pipewire # a new low-level multimedia framework. It aims to offer capture and playback for both audio and video with minimal latency and support for PulseAudio, JACK, ALSA and GStreamer-based applications
  $ systemctl enable pipewire.service
  wireplumber # more powerful Session manager and the current recommendation
  pipewire-audio
  pipewire-alsa
  pipewire-pulse
  $ systemctl --global enable pipewire-pulse.service
  pipewire-jack
  #pipewire-zeroconf
  #$ ufw allow 5353
  pipewire-jack-client
  pipewire-v4l2
  slurp
  flac
  wavpack
  lame
  libmad
  ```

- Manual pages

  ```bash
  man-db
  man-pages
  ```


### GUI

- Not for a specific DE

  ```bash
  xorg-server # the most popular display server among Linux users
  xorg-xwayland
  xorg-xeyes
  ```

- Monitor Gama & Color Adjustment

  ```bash
  xcalib # Monitor calibration loader
  xorg-xgamma # Alter a monitor's gamma correction
  acpilight
  xorg-xbacklight
  ddcutil
  autorandr # Automatically select a display configuration based on connected devices
```

- Intel Graphics

  ```bash
  xf86-video-intel
  #mesa
  mesa-amber # it's for gen7 and older but you may use mesa instead
  mesa-utils
  vulkan-intel
  intel-media-driver
  libva-intel-driver
  adriconf # GUI tool to configure Mesa drivers by setting options and writing them to the standard drirc file
  ```

- Nvidia Graphics

  ```bash
  xf86-video-nouveau
  #nvidia
  #nvidia-utils
  #nvidia-settings
  #nvidia-prime
  ```

- Input devices (Mouse/Keyboard)

  ```bash
  libinput
  xf86-input-libinput
  xorg-xinput # On-demand disabling and enabling of input sources
  xorg-xset
  xorg-xkill # Killing an application visually
  xclip
  ```

- Download Manager

  ```bash
  persepolis # Graphical front-end for aria2 download manager with lots of features
  ```

<!--
- File Sharing

  ```bash
  samba
  ```
-->

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
  mplayer # simply because pulls in a large number of codecs as dependencies, and also has codecs built in
  smplayer # because I love it :)
  smplayer-skins
  smplayer-themes
  ```

- Text Editor & Office

  ```bash
  code # Visual Studio Code Editor

  libreoffice-still # The office productivity suite compatible to the open and standardized ODF document format

  pdfslicer # Simple application to extract, merge, rotate and reorder pages of PDF documents
  ```


### KDE Applications

```bash
# desktop environemnt
plasma # you may need to exclude some pkgs e.g plasma-workspace-wallpapers, flatpak-kcm, plasma-sdk, plymouth-kcm
plasma-wayland-session
qt5-wayland
qt6-wayland
discover # KDE GUI Pkage Manager
packagekit-qt5 # to manage packages from Arch Linux repositories (not recommended, use at your own risk)
keysmith # OTP generation software by KDE
kwalletmanager # Tool to manage the passwords on your system
# kdenetwork-filesharing # Windows File and printer sharing for KDE. provides the Samba file sharing setup wizard
# smb4K # Advanced network neighborhood browser and Samba share mounting utility for KDE
filelight # Disk usage analyzer by KDE
sweeper # System cleaning utility for KDE
ksysguard # System monitor for KDE to monitor running processes and system performance.
plasma-systemmonitor # Advanced and customizable system monitor for KDE
kinfocenter # Centralized and convenient overview of system information for KDE
ksystemLog # System log viewer tool for KDE
kcron # Tool for KDE to run applications in the background at regular intervals
ktimer # Little tool for KDE to execute programs after some time
kshutdown # Graphical shutdown utility, which allows you to turn off or suspend a computer at a specified time
kate # Full-featured programmer's editor for the KDE desktop with MDI and a filesystem browser
kexi # Visual database applications creator tool by KDE
ghostwriter # Distraction-free Markdown editor
knotes # Program that lets you write the computer equivalent of sticky notes.
kcalc # Scientific calculator included in the KDE desktop
kget # Download manager for KDE
gwenview # Fast and easy to use image viewer for the KDE desktop
krita # Digital painting and illustration software included based on the KDE platform
kcolorChooser # Simple application to select the color from the screen or from a pallete
kdeconnect # Provides integration between devices
konsole # Terminal emulator included in the KDE desktop
ark # Archiving tool included in the KDE desktop
dolphin # File manager included in the KDE desktop
dolphin-plugins # uses udisksctl to either mount or unmount the iso in Dolphin
kompare # GUI front-end program for viewing and merging differences between source files with many options to customize the information
ffmpegthumbs # provides video thumbnailing plugin
kdegraphics-thumbnailers # provides PDF thumbnailing plugin, among others
qt5-imageformats # .webp, .tiff, .tga, .jp2 files
kdesdk-thumbnailers # Plugins for the thumbnailing system
taglib # Audio files
kfind # Search tool for KDE to find files by name, type or content
maliit-keyboard # Virtual keyboard useful for KDE Plasma-Wayland
kdf # Displays information about hard disks and other storage devices
aur/isoimagewriter # Tool to write a .iso file to a USB disk
ktouch # Program to learn and practice touch typing
kio-extras # MTP support is included in
bluedevil # KDE's Bluetooth tool
systemdgenie # a graphical frontend for systemctl
phonon-qt5-gstreamer # Phonon is the multimedia API provided by KDE and is the standard abstraction for handling multimedia streams within KDE software and also used by several Qt applications
kio-admin # provides a safe way to edit files as root
kio-gdrive # provides transparent KIO access to Google Drive
noto-fonts
noto-fonts-emoji
breeze-gtk # a GTK theme designed to mimic the appearance of Plasma's Breeze theme
cifs-utils # to make it look to Plasma like if the SMB share was just a normal local folder
powerdevil # for an integrated Plasma power managing service
sddm-kcm # KDE Configuration Module for SDDM
kde-gtk-config # GTK2 and GTK3 Configurator for KDE
aur/kcm-polkit-kde-git # Set of configuration modules which allows administrator to change PolicyKit settings
aur/systemd-kcm # systemd control module for KDE
plasma-browser-integration
xdg-desktop-portal # To use remote input functionality on a Plasma Wayland session/Desktop integration portals for sandboxed apps
xdg-desktop-portal-kde
libappindicator-gtk2 # icons in system tray
libappindicator-gtk3 # an attempt to get clear icons
spectacle # screenshot app
plasma5-applets-thermal-monitor # KDE Plasma applet for monitoring CPU, GPU and other available temperature sensors
plasma-disks # Hard disk health monitoring for KDE Plasma
partitionmanager # Utility to help you manage the disks, partitions, and file systems on your computer
qt5-virtualkeyboard
Okular # Universal document viewer for KDE
qt5ct # Icons
packagekit-qt5 # to install any context menu plugins
```

## Extra Works

```bash
# SDDM: Touchpad Tapping
nvim /etc/X11/xorg.conf.d/30-touchpad.conf
---------------------
Section "InputClass"
    Identifier "touchpad"
    Driver "libinput"
    MatchIsTouchpad "on"
    Option "Tapping" "on"
    Option "TappingButtonMap" "lmr"
EndSection

# ACPID: Disabling ordinary key events
nvim /etc/acpi/events/buttons
---------------------
event=button/(up|down|left|right|kpenter)
action=<drop>

# Advanced Sound
nvim /etc/modprobe.d/alsa-base.conf
---------------------
options snd_mia index=0
options snd_hda_intel index=1

nvim /etc/modules-load.d/seq-pcm-mixer.conf
---------------------
snd-seq-oss
snd-pcm-oss
snd-mixer-oss

---------------------
# By default, audio power saving is turned off by most drivers. It can be enabled by setting the power_save parameter; a time (in seconds) to go into idle mode. To idle the audio card after one second, create the following file for Intel soundcards:
nvim /etc/modprobe.d/audio_powersave.conf
---------------------
options snd_hda_intel power_save=1

# 1- It is also possible to further reduce the audio power requirements by disabling the HDMI audio output, which can done by blacklisting the appropriate kernel modules (e.g. snd_hda_codec_hdmi in case of Intel hardware).
# 2- also If you will not use integrated web camera then blacklist the uvcvideo module.
nvim /etc/modprobe.d/blacklist.conf
---------------------
blacklist snd_hda_codec_hdmi
blacklist uvcvideo

# Temporarily enable or disable the webcam:
$ modprobe uvcvideo
$ modprobe -r uvcvideo

---------------------

```

## Further Reading

https://wiki.archlinux.org/title/Core_utilities#find_alternatives
