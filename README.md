# Arch Linux Configuration Guide

My configuration guide for Arch Linux.

I'd created it just to remember things to do to configure system after fresh install.

Initial conditions assuming things set up:

- Arch Linux itself
- X Window Server and Gnome DE
- user with sudo rights and home folder

Note that some instructions are incompatible to KDE, XFCE and other DEs.

## Install yaourt

Edit `/etc/pacman.conf`:

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
$ sudo pacman -S linux-headers firefox thunderbird pidgin skype gnome-tweak-tool dropbox nautilus-dropbox gimp sublime-text-dev vlc vim
```

AUR:

```bash
$ yaourt -S google-chrome chrome-gnome-shell-git slack-desktop gimp-plugin-saveforweb yandex-browser-beta jre downgrade
```

## Install Gnome shell extensions

Install Gnome shell extensions from [https://extensions.gnome.org](https://extensions.gnome.org). In Chrome interactive installation provided by previously installed `chrome-gnome-shell-git`.

My list of extensions:

- Arch linux updates indicator
- Caffeine
- Dash to dock
- No topleft hot corner
- Pidgin IM integration
- Skype integration
- Topicons plus
- Freon

## Setup proper resolution for GDM login screen

Copy file `monitors.xml` (symlink will not work because of rights):

```bash
$ sudo cp ~/.config/monitors.xml /var/lib/gdm/.config/monitors.xml
```

Then edit `/etc/gdm/custom.conf` (because of Wayland bug which ignores `monitors.xml`):

```
WaylandEnable=false
```

Configure X Server access permission:

```
$ xhost +SI:localuser:gdm
```

*It shares some X settings with GDM such as tap-to-click for touchpad and suspend behavior. Arch Wiki: [https://wiki.archlinux.org/index.php/GDM#Enabling_tap-to-click](https://wiki.archlinux.org/index.php/GDM#Enabling_tap-to-click).*

Sources:

[https://bbs.archlinux.org/viewtopic.php?id=196219](https://bbs.archlinux.org/viewtopic.php?id=196219)

[https://bugzilla.redhat.com/show_bug.cgi?id=1184617](https://bugzilla.redhat.com/show_bug.cgi?id=1184617)

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

## Improve fonts rendering

Install missing fonts:

```bash
$ sudo pacman -S ttf-bitstream-vera ttf-inconsolata ttf-ubuntu-font-family ttf-dejavu ttf-freefont ttf-linux-libertine ttf-liberation
```

Disable bitmaps fonts:

```bash
$ sudo ln -s /etc/fonts/conf.avail/70-no-bitmaps.conf /etc/fonts/conf.d
```

Edit `/etc/pacman.conf`:

```
[multilib] 
Include = /etc/pacman.d/mirrorlist

[infinality-bundle]
Server = http://bohoomil.com/repo/$arch

[infinality-bundle-multilib]
Server = http://bohoomil.com/repo/multilib/$arch
```

Sign infinality repository:

```bash
$ sudo pacman-key -r 962DDE58
$ sudo pacman-key --lsign-key 962DDE58
```

Sync package database:

```bash
$ sudo pacman -Syu
```

Install infinality:

```bash
$ sudo pacman -S infinality-bundle infinality-bundle-multilib
```

Then reboot.

My fonts configuration for Gnome Tweak Tool:

| Setting       | Value                    |
|---------------|--------------------------|
| Window Titles | Ubuntu Bold 11           |
| Interface     | Ubuntu Regular 11        |
| Documents     | Sans Regular 11          |
| Monospace     | DejaVu Sans Mono Book 13 |

Sources:

[https://wiki.archlinux.org/index.php/Infinality](https://wiki.archlinux.org/index.php/Infinality)
[http://www.techrapid.co.uk/linux/arch-linux/improve-font-rendering-on-arch-linux/](http://www.techrapid.co.uk/linux/arch-linux/improve-font-rendering-on-arch-linux/)

## Alternate way to improve fonts rendering

Place file [.fonts.conf](.fonts.conf) in home directory, then reboot.

Sources:

[http://kaslnetwork.com/articles/making-ugly-fonts-pretty-in-arch-linux/](http://kaslnetwork.com/articles/making-ugly-fonts-pretty-in-arch-linux/)

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
$ modprobe vboxdrv vboxnetadp vboxnetflt vboxpci
```

When virtual machine will be running choose "Devices" and then "Insert Guest additions CD image". Inside guest OS (assuming it's Windows) install Guest addidtions when promted.

Sources:

[https://wiki.archlinux.org/index.php/VirtualBox](https://wiki.archlinux.org/index.php/VirtualBox)
