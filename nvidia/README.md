# Install Nvidia on Arch Linux


## List of required packages

- `nvidia-lts`
- `nvidia-utils`
- `nvidia-prime`
- `nvidia-settings`
- `xorg-server-devel`
- `opencl-nvidia`


## Configure kernel modules options

```bash
echo "options nomodeset i915.modeset=0 nouveau.modeset=0 nvidia-drm.modeset=1" > /etc/modprobe.d/nvidia.conf
```

## Configure initramfs modules for Early loading

edit the `/etc/mkinitcpio.conf` in `MODULES=` and add the following options to:

```sh
MODULES=(... nvidia nvidia_modeset nvidia_uvm nvidia_drm)
```

## Regenerate the initial RAM filesystem (initramfs) image

- `sudo mkinitcpio -P`

## - ***To avoid the possibility of forgetting to update initramfs after an NVIDIA driver upgrade, you may want to use a pacman hook:***

edit the `/etc/pacman.d/hooks/nvidia.hook` add the following lines:

```sh
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

*Make sure the Target package set in this hook is the one you have installed in steps above (e.g. `nvidia`, `nvidia-dkms`, `nvidia-lts` or `nvidia-ck-something`).*

### Power Consumption

follow: https://linrunner.de/tlp/faq/powercon.html#hybrid-graphics

aur: optimus-manager

also see [Fully power down discrete GPU](Fully%20power%20down%20discrete%20GPU.md)