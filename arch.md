# arch

## Display Manager

### SDDM

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
