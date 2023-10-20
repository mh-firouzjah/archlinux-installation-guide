# Plasma DE & KDE Applications

Here are Plasma group and a list of some other KDE applications which are not grouped onther Plasma

```bash
ark # Archiving tool included in the KDE desktop
aur/isoimagewriter # Tool to write a .iso file to a USB disk
aur/kcm-polkit-kde-git # Set of configuration modules which allows administrator to change PolicyKit settings
aur/systemd-kcm # systemd control module for KDE
dolphin # File manager included in the KDE desktop
dolphin-plugins # uses udisksctl to either mount or unmount the iso in Dolphin
drkonqi # The KDE crash handler
ffmpegthumbs # provides video thumbnailing plugin
filelight # Disk usage analyzer by KDE
ghostwriter # Distraction-free Markdown editor
gwenview # Fast and easy to use image viewer for the KDE desktop
kactivitymanagerd # System service to manage user activities and track the usage patterns
kate # Full-featured programmer's editor for the KDE desktop with MDI and a filesystem browser
kcalc # Scientific calculator included in the KDE desktop
kde-cli-tools # Tools based on KDE Frameworks 5 to better interact with the system
kde-gtk-config # GTK2 and GTK3 Configuration for KDE
kdecoration # Plugin based library to create window decorations
kdegraphics-thumbnailers # provides PDF thumbnailing plugin, among others
kdeplasma-addons # All kind of addons to improve your Plasma experience
kdesdk-thumbnailers # Plugins for the thumbnailing system
kdialog # A utility for displaying dialog boxes from shell scripts
kget # Download manager for KDE
kimageformats # Image format plugins for Qt5
kinit # Process launcher to speed up launching KDE applications
konsole # Terminal emulator included in the KDE desktop
kpat # Offers a selection of solitaire card games
ksysguard # System monitor for KDE to monitor running processes and system performance.
ksystemlog # System log viewer tool for KDE
ksystemstats # A plugin based system monitoring daemon
kwalletmanager # Tool to manage the passwords on your system
libappindicator-gtk2 # icons in system tray
libappindicator-gtk3 # an attempt to get clear icons
libqtxdg # Library providing freedesktop.org XDG specs implementations for Qt
markdownpart # KPart for rendering Markdown content (kate/okular markdown preview)
noto-fonts # 
noto-fonts-emoji # 
okular # Universal document viewer for KDE
packagekit-qt5 # lets discoover to manage packages from Arch Linux repositories (not recommended, use at your own risk)
partitionmanager # Utility to help you manage the disks, partitions, and file systems on your computer
phonon-qt5-gstreamer # Phonon is the multimedia API provided by KDE and is the standard abstraction for handling multimedia streams within KDE software
plasma # Plasma Package Group
plasma-wayland-session # 
plasma5-applets-thermal-monitor # KDE Plasma applet for monitoring CPU, GPU and other available temperature sensors
qt5-imageformats # Plugins for additional image formats: TIFF, MNG, TGA, WBMP
qtxdg-tools # libqtxdg user tools
solid # Hardware integration and detection
spectacle # screenshot app
svgpart # A KPart for viewing SVGs (kate/okular svg preview)
sweeper # System cleaning utility for KDE
systemdgenie # a graphical frontend for systemctl
taglib # Audio files
```

- Enable SDDM

  ```bash
  systemctl enable sddm
  ```

## Troubleshooting

- SDDM Touchpad Tapping

```bash
sudo nvim /etc/X11/xorg.conf.d/30-touchpad.conf
---------------------
Section "InputClass"
    Identifier "touchpad"
    Driver "libinput"
    MatchIsTouchpad "on"
    Option "Tapping" "on"
    Option "TappingButtonMap" "lmr"
EndSection
```

- SDDM Toggle External Monitor

  from the main file that script should be used here

```bash
sudo nvim /usr/share/sddm/scripts/Xsetup
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
