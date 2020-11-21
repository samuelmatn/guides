# Install Manjaro Linux on Huawei MateBook X Pro 2020

This guide describes the installation of Manjaro Linux with LVM on LUKS drive encryption. KDE Plasma is used as the desktop environment. Tested on a machine with Intel Core i7-10510U and NVIDIA GeForce MX250.
* Works immediately:
  * Wi-Fi and Bluetooth
  * webcam and microphone
  * hybrid graphics
  * HDMI external display through Huawei MateDock 2 (with sound)
  * battery charging thresholds (no GUI)
  * touchscreen (mostly useless in X11)
* Works after configuration:
  * HiDPI scaling
  * SSD TRIM
  * hibernation
  * 4.0 surround sound
  * multitouch gestures
* Doesn't work:
  * fingerprint reader
  * microphone toggle indicator

## Preparation

* Download the latest release of Manjaro Architect.
* Create a bootable USB drive using Etcher.
* During the start up, hold the F2 key to enter the BIOS menu. Disable Secure Boot and exit saving changes.
* During the start up, hold the F12 key to enter the boot manager. Select the bootable USB drive. After Manjaro Architect boots up, log in.
* Connect to Wi-Fi.
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

* Start the machine.
* Open System Settings.
  * In Display and Monitor > Display Configuration, set Global scale to 200%.
  * In Cursors, set the cursor size to 48.
  * In Startup and Shutdown > Login Screen (SDDM), synchronise the Plasma settings with SDDM.
  * In Input Devices > Touchpad, enable Invert scroll direction.
* Set the panel height to 76.
* Restart the machine.

## Enable periodic TRIM

* Edit GRUB configuration. Pass the `allow-discards` option to the kernel.
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

## Enable hibernation

* Edit `mkinitcpio.conf`. Add the `resume` hook as the last one.
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
* Edit GRUB configuration. Specify `matebook_vg-matebook_lv` as the resume device and the physical offset as the resume offset.
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

## Set up 4.0 surround speakers

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
* Edit `mkinitcpio.conf`. Add `hda-jack-retask.fw` to the image.
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
* Open System Settings. In Audio > Advanced, choose Analog Surround 4.0 Output + Analog Stereo Input as the profile.

## Set up multitouch gestures (optional)

* Install `libinput-gestures` and `xdotool`.
```
sudo pacman -S libinput-gestures xdotool
```
* Create `libinput-gestures.conf`. Configure swipe left/right to move between desktops, swipe up to show Desktop Grid and swipe down to toggle present windows.
```
nano ~/.config/libinput-gestures.conf
```
```
gesture swipe left _internal --wrap ws_up
gesture swipe right _internal --wrap ws_down
gesture swipe up xdotool key ctrl+F8
gesture swipe down xdotool key ctrl+F9
```
* Launch `libinput-gestures` manually and set up autostart.
```
libinput-gestures-setup start
libinput-gestures-setup autostart
```

## Remap hotkeys (optional)

* Install `evtest`.
```
sudo pacman -S evtest
```
* Find the event handler for Huawei WMI hotkeys. 
```
cat /proc/bus/input/devices
```
* Find the values of the hotkeys to be remapped.
```
sudo evtest /dev/input/<EVENT_HANDLER>
```
* Create `90-custom-hotkeys.hwdb`. Remap the microphone toggle to previous song, the Wi-Fi toggle to play/pause and the PC Manager launcher to next song.
```
sudo nano /etc/udev/hwdb.d/90-custom-hotkeys.hwdb
```
```
evdev:name:Huawei WMI hotkeys:*
 KEYBOARD_KEY_287=previoussong
 KEYBOARD_KEY_289=playpause
 KEYBOARD_KEY_28a=nextsong
```
* Update the hardware database. Reapply the rules in the device manager.
```
sudo systemd-hwdb update
sudo udevadm trigger
```

## Disable middle-click paste (optional)

* Open Clipboard configuration. In General, disable Prevent empty clipboard.
* Install `xbindkeys`, `xsel` and `xdotool`.
```
sudo pacman -S --needed xbindkeys xsel xdotool
```
* Create `.xbindkeysrc` and disable middle-click paste.
```
nano ~/.xbindkeysrc
```
```
"echo -n | xsel -n -i; pkill xbindkeys; xdotool click 2; xbindkeys"
b:2 + Release
```
* Launch `xbindkeys` manually.
```
xbindkeys
```
* Open System Settings. In Startup and Shutdown > Autostart, add `xbindkeys` as an application.

## Change the battery charging thresholds (optional)
* Edit `charge_control_thresholds`. Set up to start charging at 75% (or below) and stop charging at 80% (or above).
```
sudo nano /sys/devices/platform/huawei-wmi/charge_control_thresholds
```
```
75 80
```

## Final steps (optional)

* Install Breeze GTK theme.
```
sudo pacman -S breeze-gtk
```
* Uninstall Steam.
```
sudo pacman -Rs steam-manjaro
```
* Uninstall Yakuake.
```
sudo pacman -Rs yakuake 
```
* Uninstall Manjaro Hello.
```
sudo pacman -Rs manjaro-hello
```
* Uninstall K3b.
```
sudo pacman -Rs k3b
```
* Uninstall Konverstaion.
```
sudo pacman -Rs konversation
```
* Uninstall Cantata and MPD.
```
sudo pacman -Rs cantata mpd
```

## Use VMware Workstation to run Windows 10 (optional)

* Install Linux kernel headers.
```
sudo pacman -S linux-lts-headers linux-latest-headers
```
* Install VMware Workstation.
```
pamac build vmware-workstation
```
* Insert the VMware kernel modules.
```
sudo modprobe -a vmw_vmci vmmon
```
* Enable and manually start the VMware services.
```
sudo systemctl enable vmware-networks.service  vmware-usbarbitrator.service
sudo systemctl start vmware-networks.service  vmware-usbarbitrator.service
```
* Edit the VMware Workstation preferences file. Add the option to completely hide the toolbar in full-screen mode.
```
nano ~/.vmware/preferences
```
```
pref.vmplayer.fullscreen.nobar = "TRUE"
```
* Launch VMware Player and create a new virtual machine located in `~/VMware/Windows 10`. Close VMware Player.
* Export the Microsoft Data Management table to a file.
```
sudo cat /sys/firmware/acpi/tables/MSDM > ~/VMware/MSDM.bin
```
* Edit the virtual machine configuration file. Add the option to reflect the host BIOS management information. Add the exported table to the guest ACPI tables.
```
nano "~/VMware/Windows 10/Windows 10.vmx"
```
```
SMBIOS.reflectHost = "TRUE"
acpi.addtable.filename = "~/VMware/MSDM.bin"
```
* Launch VMware Player and install Windows 10 inside the virtual machine. Install VMware Tools.

## References

* [Encrypted Manjaro Installation using Manjaro Architect](https://forum.manjaro.org/t/encrypted-manjaro-installation-using-manjaro-architect/2709) - Manjaro Linux Forum
* [Arch Adventures: Hibernating to swap file](https://vadosware.io/2015/10/15/arch-adventures-hibernating-to-swap-file/) - VADOSWARE
* [Fix Huawei MateBook X Pro Speakers on Linux](https://github.com/hg8/arch-matebook-x-pro-2019/blob/master/guide-fix-matebook-x-pro-speakers-linux.md) - hg8 on GitHub
* [Linux keymapping with udev hwdb](https://yulistic.gitlab.io/2017/12/linux-keymapping-with-udev-hwdb/) - Yulistic
* [Remapping ThinkPad Keys with udev hwdb](https://blog.zhanghai.me/remapping-keys-with-udev-hwdb/) - Hai Zhang's blog
* [How can I turn off “middle mouse button paste” functionality in all programs?](https://unix.stackexchange.com/questions/24330/how-can-i-turn-off-middle-mouse-button-paste-functionality-in-all-programs) - Unix & Linux Stack Exchange
* [Huawei WMI laptop extras linux driver](https://github.com/aymanbagabas/Huawei-WMI) - Ayman Bagabas on GitHub
* [Can I completely hide toolbar in VMware Workstation?](https://superuser.com/questions/246644/can-i-completely-hide-toolbar-in-vmware-workstation) - Super User Stack Exchange
* [How to edit BIOS information for a virtual machine in VMWare?](https://superuser.com/questions/199906/how-to-edit-bios-information-for-a-virtual-machine-in-vmware) - Super User Stack Exchange
* [Replace ACPI Tables in VMware BIOS](https://communities.vmware.com/t5/ESXi-Discussions/Replace-ACPI-Tables-in-VMware-BIOS/td-p/426629) - VMware Technology Network