# Fully power down discrete GPU

You may want to turn off the high-performance graphics processor to save battery power.

## Using BIOS/UEFI

Some laptop manufacturers provide a toggle in the BIOS or UEFI to fully deactivate the dedicated card.

## Using udev rules

Ensure any display manager config for NVIDIA is removed.

Blacklist the nouveau drivers by creating

```bash
/etc/modprobe.d/blacklist-nouveau.conf
blacklist nouveau
options nouveau modeset=0
```

Then create

```bash
/etc/udev/rules.d/00-remove-nvidia.rules
# Remove NVIDIA USB xHCI Host Controller devices, if present
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x0c0330", ATTR{power/control}="auto", ATTR{remove}="1"

# Remove NVIDIA USB Type-C UCSI devices, if present
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x0c8000", ATTR{power/control}="auto", ATTR{remove}="1"

# Remove NVIDIA Audio devices, if present
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x040300", ATTR{power/control}="auto", ATTR{remove}="1"

# Remove NVIDIA VGA/3D controller devices
ACTION=="add", SUBSYSTEM=="pci", ATTR{vendor}=="0x10de", ATTR{class}=="0x03[0-9]*", ATTR{power/control}="auto", ATTR{remove}="1"
```

Reboot and run `lspci` to see if your NVIDIA GPU is still listed.
