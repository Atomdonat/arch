# arch

## Display Manager

### SDDM (Login Manager) with Breeze Dark Theme
#### Login with (FIDO2) Thumbdrive
- https://wiki.archlinux.org/title/Nitrokey 
- https://forums.gentoo.org/viewtopic-t-1168984.html
- https://www.reddit.com/r/archlinux/comments/18yo9gr/sddm_login_with_password_or_fingerprint_on_arch/

### KDE-Plasma / KWin

## AUR

- cd ~/aur

- git clone <package_url>

- cd package_folder

- makepkg # creates .pkg.tar.xz package_file

- sudo pacman -U <package_file>


## Troubleshootings

### Audio

- removing pipewire with Rncs also removes plasma(-meta)

- "sof-firmware" was missing first -> no soundcard & no audio devices

### SDDM

- if only sddm is installed (no DE), hitting enter on login wont do anything, because nothing is there

### Wine

- can't install/find packages wine and lib32-...

- in /etc/pacman.conf:
    '''
    [multilib]
    Include = /etc/pacman.d/mirrorlist
    '''
    was commented out

-> pacman -Syu wine worked

### Discord Screen Sharing
Problem: infinite loop of Wayland screen share window and Discord screen share window
Fix: 
	1. remove Discord `sudo pacman -Rs discord` 
	2. install WebCord using [flatpak](https://wiki.archlinux.org/title/Flatpak#List_repositories) `flatpak install WebCord`
(Same problem: https://discuss.kde.org/t/screensharing-menu-in-wayland-conflicts-with-same-menu-of-discord/14657; Solution:https://lemmy.ml/post/1557630)