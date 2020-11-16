# Installing Manjaro Linux on Huawei MateBook X Pro 2020

## Preparation

* Download the latest version of Manjaro Architect.
* Create a bootable USB drive using Etcher.
* During the start up, hold the F2 key to enter the BIOS menu. Disable Secure Boot and exit saving changes.
* During the start up, hold the F12 key to enter the boot manager. Select the bootable USB drive. Boot Manjaro Architect and log in.
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
  * Set up LVM. Select Create VG and LV(s). Use matebook_vg as the volume group name. Select /dev/mapper/cryptroot as the physical volume. Create 1 logical volume. Use matebook_lv as the logical volume name.
  * Mount partitions. Select /dev/mapper/matebook_vg-matebook_lv as the ROOT partition. Format it as ext4. Keep noatime as the only mount option. Use swapfile instead of a SWAP partition. Specify 32G as the size. Finish mounting partitions. Select /dev/nvme0n1p1 as UEFI partition. Reformat it. Keep /boot/efi as the UEFI mountpoint.
  * Configure installer mirrorlist. Rank mirrors by speed. Keep only the stable branch selected. Select mirrors.
* Go back to the main menu and choose Install Desktop System.
  * Select Install Manjaro desktop. Select yay+base-devel linux-lts and linux-latest from the base package group. Select kde as the desktop environment. Do not add additional packages. Select the full version. Select auto-install proprietary drivers.
  * Select Install bootloader. Use grub. Set the bootloader as default.
  * Select Configure base. Generate FSTAB using the default Device UUID option. Set hostname to samuel-matebook. Set system language and system locale as en_GB.UTF-8. Set desktop keyboard layout to us. Set timezone to Europe/Prague and use utc. Set root password. Create new user. Choose bash as the default shell.
* Close the installer and shut down the machine.
```
systemctl poweroff
```

## Quick fixes

* Open System Settings.
  * In Display and Monitor > Display Configuration, set Global scale to 200%.
  * In Cursors, set the cursor size to 48.
  * In Startup and Shutdown > Login Screen (SDDM), synchronise the Plasma settings with SDDM.
  * In Input Devices > Touchpad, select Invert scroll direction.
* Set the panel height to 76.
* Restart the machine.

## Set up periodic TRIM

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