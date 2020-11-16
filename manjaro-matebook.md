# Installing Manjaro Linux on Huawei MateBook X Pro 2020

## Preparation

* [Download](https://manjaro.org/download/) the latest release of Manjaro Architect.
* Create a bootable USB drive using [Etcher](https://www.balena.io/etcher/).
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
  * Choose Configure base. Generate `fstab` using the default Device UUID option. Set the hostname, I used `samuel-matebook`. Set the system language and the system locale, I chose `en_GB.UTF-8`. Set the desktop keyboard layout. Set the timezone to and use UTC. Set the root password. Create a new user. Choose Bash as the default shell.
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

* Verify TRIM support.
```
lsblk --discard
```
* Try executing TRIM command.
```
sudo fstrim -v /
```
* Check the existence and status of fstrim timer service.
```
sudo systemctl status fstrim.timer
```
* Enable fstrim timer service.
```
sudo systemctl enable fstrim.timer
```