# Arch Linux Configuration Guide

My configuration guide for Arch Linux.

I'd created it just to remember things to do to configure system after fresh install.

Initial conditions assuming this things already set up:

- Arch Linux
- Gnome DE
- user with sudo rights and home folder

## Install yaourt

Uncomment following strings in `/etc/pacman.conf`:

```
[archlinuxfr]
SigLevel = Never
Server = http://repo.archlinux.fr/$arch
```

Sync package database:

```bash
$ sudo pacman -Syu
```
Install package:

```bash
$ sudo pacman -S yaourt
```

## Install new packages

Official:

```bash
$ sudo pacman -S linux-headers chromium firefox thunderbird pidgin skypeforlinux-bin gnome-tweak-tool dropbox nautilus-dropbox gimp sublime-text-dev vlc vim dosfstools
```

AUR:

```bash
$ yaourt -S chrome-gnome-shell-git chromium-pepper-flash chromium-widevine vivaldi vivaldi-ffmpeg-codecs slack-desktop gimp-plugin-saveforweb yandex-browser-beta telegram-desktop-bin jre downgrade
```

## Install Gnome shell extensions

Install Gnome shell extensions from [https://extensions.gnome.org](https://extensions.gnome.org). Interactive on-site installation provided by `chrome-gnome-shell-git`.

## Setup default applications

```bash
$ yaourt -S gnome-defaults-list
```

Copy and edit MIME-list:

```bash
$ cp /etc/gnome/defaults.list ~/.local/share/applications/mimeapps.list
```

Sources:

[https://wiki.archlinux.org/index.php/default_applications](https://wiki.archlinux.org/index.php/default_applications)

## Configure GDM login screen

(?) Copy `monitors.xml`:

```bash
$ sudo cp ~/.config/monitors.xml /var/lib/gdm/.config/monitors.xml
```

Then enter `gdm` user shell:

```bash
$ sudo su gdm -s /bin/bash
$ gsettings set org.gnome.desktop.interface gtk-theme GTK3_THEME
$ gsettings set org.gnome.desktop.interface icon-theme ICON_THEME
$ gsettings set org.gnome.desktop.interface cursor-theme CURSOR_THEME
$ gsettings set org.gnome.desktop.background picture-uri 'file://FILE'
```

Sources:

[https://bbs.archlinux.org/viewtopic.php?id=196219](https://bbs.archlinux.org/viewtopic.php?id=196219)

[https://bugzilla.redhat.com/show_bug.cgi?id=1184617](https://bugzilla.redhat.com/show_bug.cgi?id=1184617)

[https://wiki.archlinux.org/index.php/GDM#Enabling_tap-to-click](https://wiki.archlinux.org/index.php/GDM#Enabling_tap-to-click)

[http://askubuntu.com/questions/52463/how-to-set-gtk-style-and-background-in-gdm3](http://askubuntu.com/questions/52463/how-to-set-gtk-style-and-background-in-gdm3)

## Swapiness

Edit (or create) `/etc/sysctl.d/99-sysctl.conf`:

```
vm.swappiness=10
```

Sources:

[https://wiki.archlinux.org/index.php/Swap#Swappiness](https://wiki.archlinux.org/index.php/Swap#Swappiness)

## Automate TRIM operation for SSD

Enable TRIM via timer (will activate weekly):

```bash
$ sudo systemctl enable fstrim.timer
```

Sources:

[https://wiki.archlinux.org/index.php/Solid_State_Drives#Apply_periodic_TRIM_via_fstrim](https://wiki.archlinux.org/index.php/Solid_State_Drives#Apply_periodic_TRIM_via_fstrim)

## Setup Virtualbox

Install packages:

```bash
$ sudo pacman -S virtualbox virtualbox-guest-iso linux-headers
$ yaourt virtualbox-ext-oracle
```

Generate Virtualbox kernel modules:
```bash
$ sudo dkms autoinstall
```

(?) Enable dkms.service to recompile Virtualbox kernel modules each time dkms has been upgraded:
```bash
$ sudo systemctl enable dkms.service
```

**dkms.service seems not to be available anymore, maybe now it running not via systemd.*

Create `/etc/modules-load.d/virtualbox.conf`:

```
vboxdrv
vboxnetadp
vboxnetflt
vboxpci
```

This will load Virtualbox modules on system boot.

Then reboot or load Virtualbox modules in current kernel session:

```bash
$ sudo modprobe vboxdrv vboxnetadp vboxnetflt vboxpci
```

When virtual machine will be running choose "Devices" and then "Insert Guest additions CD image". Inside guest OS (assuming it's Windows) install Guest addidtions when promted.

To be able to use USB ports inside guest OS add current user to `vboxusers` group:

```bash
$ sudo gpasswd -a $USER vboxusers
```

Sources:

[https://wiki.archlinux.org/index.php/VirtualBox](https://wiki.archlinux.org/index.php/VirtualBox)

## Setup printers

Install packages:

```bash
$ sudo pacman -S cups ghostscript gsfonts system-config-printer
```

Start and enable CUPS service:

```bash
$ sudo systemctl start org.cups.cupsd.service
$ sudo systemctl enable org.cups.cupsd.service
```

Then configure printers via Gnome settings.

Sources:

[https://wiki.archlinux.org/index.php/CUPS](https://wiki.archlinux.org/index.php/CUPS)

[https://wiki.archlinux.org/index.php/CUPS/Printer-specific_problems](https://wiki.archlinux.org/index.php/CUPS/Printer-specific_problems)

[https://bbs.archlinux.org/viewtopic.php?id=200334](https://bbs.archlinux.org/viewtopic.php?id=200334)

## Setup Flash in Yandex browser

If `google-chrome` installed then Flash will work out of the box. If not, additional configuration required.

Install packages:

```bash
$ yaourt -S chromium-pepper-flash
```

Edit `/opt/yandex/browser-beta/yandex-browser-beta` (insert before `$@` in last line):

```bash
--ppapi-flash-path=/usr/lib/PepperFlash/libpepflashplayer.so
```

Sources:

[https://bbs.archlinux.org/viewtopic.php?id=191773](https://bbs.archlinux.org/viewtopic.php?id=191773)

## Setup Intel CPU microcode

Install packages:

```bash
$ sudo pacman -S intel-ucode
```

Then add new entry for `systemd-boot` (`/boot/loader/entries/arch.conf`):

```
initrd  /intel-ucode.img
```

Sources:
[https://wiki.archlinux.org/index.php/microcode](https://wiki.archlinux.org/index.php/microcode)

## Configure systemd log

Retain log for last seven days:

```bash
$ sudo journalctl --vacuum-time=7d
```

Sources:

[http://unix.stackexchange.com/a/194058](http://unix.stackexchange.com/a/194058)
