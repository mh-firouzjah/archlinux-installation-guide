# XFCE Desktop

Some of the packages from xfce4 and xfce4-goodies are somehow broken for me and maybe
you also experience the same, I removed those packages (at least those were not really necessary) by:

```bash
# tumbler is broken but it's a dependency to ristretto so I removed both of them, also viewnior >>> ristretto
sudo pacman -Rns tumbler ristretto
```

- The desktop and sessions

  ```bash
  xfce4
  xfce4-goodies
  lightdm-gtk-greeter
  lightdm-gtk-greeter-settings
  gnome-keyring
  ```

- Themes

  ```bash
  gnome-themes-extra
  arc-gtk-theme
  arc-solid-gtk-theme
  arc-icon-theme
  papirus-icon-theme
  ```

- Image viewer

  ```bash
  viewnior
  ```

- LightDM Toggle External Monitor

```bash
nvim /etc/lightdm/lightdm.conf
---------------------
[Seat:*]
...
display-setup-script=/usr/bin/multiple-monitors.sh
```
