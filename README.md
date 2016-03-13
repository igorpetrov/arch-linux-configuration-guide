# Arch Linux Configuration Guide

My configuration guide for Arch Linux.
I create it to remember all things to do to configure system after install.
Initial conditions assuming things set up:
- Arch Linux itself
- X Window Server and Gnome DE
- user with sudo rights and home folder
Notice that some of instructions are incompatible to KDE, XFCE and other DEs.
*Beware to just copy and paste commands and revise to Arch Wiki in relation to unique parameters of your system.*

## Add new repositories to pacman
- archlinuxfr
- multilib

## Install yaourt
...

## Install new packages
My set of packages listed below.

Official:
```bash
$ sudo pacman -S linux-headers firefox thunderbird pidgin skype gnome-tweak-tool dropbox nautilus-dropbox gimp sublime-text-dev vlc vim
```

AUR:
```bash
$ yaourt -S google-chrome chrome-gnome-shell-git slack-desktop gimp-plugin-saveforweb yandex-browser-beta
```

## Install Gnome shell extensions from [https://extensions.gnome.org](https://extensions.gnome.org) (interactive install should be support in Chrome by previuosly installed `chrome-gnome-shell-git`).
My list of extensions:
- Arch linux updates indicator
- Caffeine
- Dash to dock
- No topleft hot corner
- Pidgin IM integration
- Skype integration
- Topicons plus

## Setup proper resolution for GDM login screen
Copy file monitors.xml (symlink will not work):
$ sudo cp ${HOME}/.config/monitors.xml /var/lib/gdm/.config/monitors.xml

Then edit /etc/gdm/custom.conf (because of bug of Wayland which ignores monitors.xml):
WaylandEnable=false

Configure X server access permission:
$ xhost +SI:localuser:gdm

I use it to enable tap-to-click for my touchpad but actually I don't know what else this setting do. Read Arch Wiki: https://wiki.archlinux.org/index.php/GDM#Enabling_tap-to-click

Sources:
https://bbs.archlinux.org/viewtopic.php?id=196219
https://bugzilla.redhat.com/show_bug.cgi?id=1184617

## Swapiness
Edit (or create) /etc/sysctl.d/99-sysctl.conf:
vm.swappiness=10

Sources:
https://wiki.archlinux.org/index.php/Swap#Swappiness

## Enable TRIM operation for SSD
Enable TRIM via timer (will activate weekly):
$ sudo systemctl enable fstrim.timer

Sources:
https://wiki.archlinux.org/index.php/Solid_State_Drives#Apply_periodic_TRIM_via_fstrim

## Improve fonts rendering
Install missing fonts:
$ sudo pacman -S ttf-bitstream-vera ttf-inconsolata ttf-ubuntu-font-family ttf-dejavu ttf-freefont ttf-linux-libertine ttf-liberation

Disable bitmaps fonts:
$ sudo ln -s /etc/fonts/conf.avail/70-no-bitmaps.conf /etc/fonts/conf.d

Edit /etc/pacman.conf:
[multilib] 
Include = /etc/pacman.d/mirrorlist

[infinality-bundle]
Server = http://bohoomil.com/repo/$arch

[infinality-bundle-multilib]
Server = http://bohoomil.com/repo/multilib/$arch

Sign infinality repository:
$ sudo pacman-key -r 962DDE58
$ sudo pacman-key --lsign-key 962DDE58

Install infinality:
$ sudo pacman -Syy infinality-bundle
$ sudo pacman -Syy infinality-bundle-multilib

Then reboot.

My fonts congiguration for Gnome Tweak Tool:
Window Titles | Ubuntu Bold 11
Interface     | Ubuntu Regular 11
Documents     | Sans Regular 11
Monospace     | DejaVu Sans Mono Book 13

Sources:
https://wiki.archlinux.org/index.php/Infinality
http://www.techrapid.co.uk/linux/arch-linux/improve-font-rendering-on-arch-linux/

## Alternate way to improve fonts rendering
Place code in .fonts.conf
... (code)

Sources:
http://kaslnetwork.com/articles/making-ugly-fonts-pretty-in-arch-linux/

## Setup Virtualbox
Install packages:
$ sudo pacman -S virtualbox virtualbox-guest-iso linux-headers
$ yaourt virtualbox-ext-oracle

Generate Virtualbox kernel modules:
$ sudo dkms autoinstall

(?) Enable dkms.service to recompile Virtualbox kernel modules each time dkms has been upgraded:
$ sudo systemctl enable dkms.service

Edit (or create) /etc/modules-load.d/virtualbox.conf:
vboxdrv
vboxnetadp
vboxnetflt
vboxpci

Then reboot or load Virtualbox modules in current session:
$ modprobe vboxdrv vboxnetadp vboxnetflt vboxpci

When virtual machine is running (assuming it's Windows) choose "Devices" and them "Add Guest additions ISO". Inside guest OS install Guest addidtions when promted.

Sources:
https://wiki.archlinux.org/index.php/VirtualBox
