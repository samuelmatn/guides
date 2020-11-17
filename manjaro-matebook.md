# Installing Manjaro Linux on Huawei MateBook X Pro 2020

## Preparation

* Download the latest release of Manjaro Architect.
* Create a bootable USB drive using Etcher.
* During the start up, hold the F2 key to enter the BIOS menu. Disable Secure Boot and exit saving changes.
* During the start up, hold the F12 key to enter the boot manager. Select the bootable USB drive. After Manjaro Architect boots up, log in.
* Connect to WiFi.
```
nmcli device wifi list
nmcli device wifi connect <SSID> password <PASSWORD>
```

## Installation

* Launch the installer. Choose English as the language.
* In the main menu, choose Prepare installation.
  * Set virtual console to `us`.
  * Partition the disk `/dev/nvme0n1`. Use automatic partitioning.
  * Set up LUKS encryption. Choose Automatic LUKS Encryption. Choose `/dev/nvme0n1p2`. Leave the default `cryptroot` as the block device name. Enter the encryption password.
  * Set up LVM. Choose Create VG and LV(s). Use `matebook_vg` as the volume group name. Select `/dev/mapper/cryptroot` as the physical volume. Create 1 logical volume. Use `matebook_lv` as the logical volume name.
  * Mount partitions. Select `/dev/mapper/matebook_vg-matebook_lv` as the root partition. Format it as ext4. Keep `noatime` as the only mount option. Use swap file instead of swap partition. Specify `32G` as its size. Finish mounting partitions. Choose `/dev/nvme0n1p1` as the UEFI partition and reformat it. Keep `/boot/efi` chosen as the UEFI mountpoint.
  * Configure installer mirrorlist. Rank mirrors by speed. Keep only the stable branch selected. Select the mirrors.
* Go back to the main menu and choose Install Desktop System.
  * Choose Install Manjaro desktop. Select `yay+base-devel`, `linux-lts` and `linux-latest` from the base package group. Select KDE as the desktop environment. Do not add additional packages. Choose the full version. Choose auto-install proprietary drivers.
  * Choose Install bootloader. Use GRUB. Set the bootloader as default.
  * Choose Configure base. Generate `fstab` using the default Device UUID option. Set the hostname. Set the system language and the system locale. Set the desktop keyboard layout. Set the timezone to and use UTC. Set the root password. Create a new user. Choose Bash as the default shell.
* Close the installer and shut down the machine. Remove the bootable USB drive.
```
systemctl poweroff
```

## Quick fixes

* Start the machine and log in.
* Open System Settings.
  * In Display and Monitor > Display Configuration, set Global scale to 200%.
  * In Cursors, set the cursor size to 48.
  * In Startup and Shutdown > Login Screen (SDDM), synchronise the Plasma settings with SDDM.
  * In Input Devices > Touchpad, enable Invert scroll direction.
* Set the panel height to 76.
* Restart the machine.

## Setting up periodic TRIM

* Edit `/etc/default/grub`. Pass the `allow-discards` option to the kernel.
```
sudo nano /etc/default/grub
```
```
GRUB_CMDLINE_LINUX="cryptdevice=UUID=<UUID>:cryptroot:allow-discards"
```
* Update the GRUB configuration file.
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
* Restart the machine.
* Verify TRIM support.
```
lsblk --discard
```
* Manually discard the unused blocks on the filesystem.
```
sudo fstrim -v /
```
* Check the existence and the status of the `fstrim` timer.
```
sudo systemctl status fstrim.timer
```
* Enable the `fstrim` timer.
```
sudo systemctl enable fstrim.timer
```

## Enabling hibernation

* Edit `/etc/mkinitcpio.conf`. Add the `resume` hook as the last one.
```
sudo nano /etc/mkinitcpio.conf
```
```
HOOKS=(base udev autodetect keymap modconf block encrypt lvm2 filesystems keyboard resume)
```
* Update `initramfs`.
```
sudo mkinitcpio -P
```
* Find the physical offset of the swap file. It corresponds to the logical offset 0.
```
sudo filefrag -v /swapfile
```
* Edit `/etc/default/grub`. Specify `matebook_vg-matebook_lv` as the resume device and the physical offset as the resume offset.
```
sudo nano /etc/default/grub
```
```
GRUB_CMDLINE_LINUX="resume=/dev/mapper/matebook_vg-matebook_lv resume_offset=<PHYSICAL_OFFSET> cryptdevice=UUID=<UUID>:cryptroot:allow-discards"
```
* Update the GRUB configuration file.
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
* Restart the machine.

## Setting up 4.0 surround speakers

* Install ALSA Tools.
```
sudo pacman -S alsa-tools
```
* Launch HDAJackRetask as root.
```
sudo hdajackretask
```
* On the top, select Realtek ALC256 codec. On the right, enable Show unconnected pins and Advanced overrides.
* Enable Override on Pin ID 0x14. Set the following values:
  * Connectivity: Internal
  * Location: Internal
  * Device: Speaker
  * Jack: Other Analog
  * Color: Unknown
  * Jack detection: Not present
  * Channel group: 1
  * Channel (in group): Front
* Enable Override on Pin ID 0x1b. Set the following values:
  * Connectivity: Internal
  * Location: Internal
  * Device: Speaker
  * Jack: Other Analog
  * Color: Unknown
  * Jack detection: Not present
  * Channel group: 1
  * Channel (in group): Back
* On the bottom right, click Install boot override.
* Edit `/etc/mkinitcpio.conf`. Add `hda-jack-retask.fw` to files.
```
sudo nano /etc/mkinitcpio.conf
```
```
FILES=(/crypto_keyfile.bin /usr/lib/firmware/hda-jack-retask.fw)
```
* Update `initramfs`.
```
sudo mkinitcpio -P
```
* Restart the machine.
* Open System Settings. In Audio > Advanced, choose Analog Surround 4.0 Output as the profile.

## Setting up multitouch gestures

* Install `libinput-gestures`.
```
sudo pacman -S libinput-gestures
```
* Create `libinput-gestures.conf`. Configure swipe left/right to move between desktops, swipe up to show desktop grid and swipe down to toggle present windows.
```
nano ~/.config/libinput-gestures.conf
```
```
gesture swipe left _internal --wrap ws_up
gesture swipe right _internal --wrap ws_down
gesture swipe up xdotool key ctrl+F8
gesture swipe down xdotool key ctrl+F9
```
* Launch `libinput-gestures` and let it launch automatically next time.
```
libinput-gestures-setup start
libinput-gestures-setup autostart
```