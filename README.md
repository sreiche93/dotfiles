#.files

This repository will give you an overview on:
- how to install Arch Linux on an encrypted SSD
- the needed configuration files to achieve this lock

It's mainly designed to document the procedure for myself and friends.
But you're welcome to try it out yourself. :)

##Directory structure
```
i3  - i3 is a tiling window manager
```

##Directory structure
```
i3:       ~/.config/i3/config
i3status: ~/.config/i3status/config
xinitrc:  ~/.xinitrc
```

##Installation
###1. Download the ISO-Image
Download the actual image [here](https://www.archlinux.org/download/).
Make sure to compare the checksums to ensure file integrity.

###2. Base install
The beginning is straight forward and does not differ from the official wiki.

*(This step does not apply if you don't use a qwertz-keyboard)*  
First you want to change your keyboard layout to make it compatible with the normal german qwertz keyboards.  
`# loadkeys de`


I decided to choose brtfs as the used file system. For more information about the difference file systems check the [relevant wiki page](https://wiki.archlinux.org/index.php/File_systems).

`# parted -a optimal /dev/sda mklabel gpt mkpart primary 0% 257MiB name 1 boot mkpart primary 257MiB 100% name 2 root
mkfs.btrfs -L boot /dev/sda1`

alternative split:
```
# parted -a optimal /dev/sda
(parted) mklabel gpt
(parted) mkpart primary 0% 257MiB name 1 boot
(parted) mkpart primary 257MiB 100% name 2 root
(parted) quit
# mkfs.btrf -L boot /dev/sda1
```

Disc encryption:
```
# modprobe dm-crypt
# cryptsetup -c aes-xts-plain64 -s 512 -h sha512 -i 5000 -y luksFormat /dev/sda2
# cryptsetup open /dev/mapper lvm
```

Prepare and mount the logical volumes.
```
# pvcreate /dev/mapper/lvm
# vgcreate /dev/mapper/lvm

# lvcreate -L 40GB -n root main
# lvcreate -L 8GB -n swap main
# lvcreate -l 100%FREE -n home main

# lvs
# mkfs.btrfs -L root /dev/mapper/main-root
# mkfs.btrfs -L home /dev/mapper/main-home
# mkswap /dev/mapper/main-swap
# swapon /dev/mapper/main-swap

# mount /dev/mapper/main-root /mnt
# mkdir /mnt/boot
# mkdir /mnt/home
# mount /dev/sda1 /mnt/boot
# mount /dev/mapper/main-home /mnt/home
```
From this part on the installation is pretty much the same compared to the official installation guide. 
```
# ip a
# dhcpcd enp5s0

# cd /etc/pacman.d
# cp mirrorlist mirrorlist.b
# rankmirrors -n 3 mirrorlist.b > mirrorlist

# pacstrap /mnt base base-devel
# genfstab -p /mnt >> /mnt/etc/fstab

# arch-chroot /mnt /bin/bash

# echo Yoga2 >> /etc/hostname
# ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime

# nano /etc/locale.gen
```

```
#en_SG ISO-8859-1
en_US.UTF-8 UTF-8
#en_US ISO-8859-1
```

```
# locale-gen
# echo "LANG=en_US.UTF-8" >> /etc/locale.conf
# echo "KEYMAP=de-latin1" >> /etc/vconsole.conf

# nano /etc/mkinitcpio.conf
```

```
HOOKS="base udev autodetect modconf block keyboard keymap encrypt lvm2 filesystems fsck shutdown"
```

```
# mkinitcpio -p linux
# passwd

# pacman -S gptfdisk syslinux
#syslinux-install_update -iam
# nano /boot/syslinux/syslinux.cfg
```
Delete everything below INITRD (of label arch) except reboot.  
Edit the append-line:
```
APPEND cryptdevice=/dev/sda2:main root=/dev/mapper/main-root rw
```

```
# exit
# umount -R /mnt
# reboot
```

###3. Configure the system
(This may not be the most elegant way but it works for me)  
Change the syslinux-config (`nano /boot/syslinux.cfg`):
 Set `TIMEOUT` to 1. This way you won't have to wait 5s while booting.

Now add a new user:
```
# useradd -m -g users -s /bin/bash sreiche93
# passwd sreiche93
```
Edit the sudoers file (`nano /etc/sudoers`) and remove the `#` in front of `%wheel ALL=(ALL) ALL`

```
gpasswd -a sebastian wheel
```

X-Installation:
```
# pacman -S xorg-server xorg-xinit xorg-utils xorg-server-utils xorg-drivers
# cp /etc/x11/xinit/xinitrc ~/.xinitrc
```

AUR:
```
# groupadd abs
# gpasswd -a srcodes abs
# mkdir -p /var/abs/local
# chown root:abs /var/abs/local
```

Now `logout` and log into the new created user.

i3+gaps:
```
# curl -O https://aur.archlinux.org/cgit/aur.git/snapshot/i3-gaps-git.tar.gz
# tar -xvzf i3-gaps-git.tar.gz
# cd i3-gaps-git
# makepkg -si

# git clone https://www.github.com/Airblader/i3 i3-gaps
# cd i3-gaps
# git checkout gaps && git pull
# make
# sudo make install
```
`# sudo pacman -S i3-status`

Now copy the .xinitrc

Useful applications:
```
sudo pacman -S ttf-dejavu
sudo pacman -S dmenu
sudo pacman -S vim
sudo pacman -S rxvt-unicode
sudo pacman -S firefox firefox-i18n-de
sudp pacman -S flashplugin icedtea-web
sudo pacman -S vlc
sudo pacman -S feh
sudo pacman -S zip unzip
sudo pacman -S libreoffice-still libreoffice-still-de
sudo pacman -S texlive-most texlive-lang texmaker
sudo pacman -S thunderbird
sudo pacman -S pcmanfm gvfs
sudo pacman -S alsa-utils pulseaudio pavucontrol
sudo pacman -S jupyter jupyter-notebook ipython2-notebook
sudo pacman -S python-pip python2-pip
sudo pacman -S ranger
sudo pacman -S iw wpa_supplicant
sudo pacman -S dialog
sudo pacman -S networkmanager network-manager-applet dhclient
sudo pacman -S gnome-keyring
sudo pacman -S gedit
sudo pacman -S python-pyqt4
sudo pacman -S git
sudo pacman -S cbatticon
sudo pacman -S nodejs npm
sudo pacman -S mupdf
sudo pacman -S rofi
sudo pacman -S chromium
sudo pacman -S ghc
sudo pacman -S lxrandr
sudo pacman -S i3lock
sudo pacman -S pidgin pidgin-otr
sudo pacman -S gtk3
```

From AUR:
```
yaourt -S lxterminal
yaourt -S anaconda
yaourt -S spotify
yaourt -S sublime-text
yaourt -S android-studio
yaourt -S android-sdk android-sdk-platform-tools android-sdk-build-tools (benötigt multilib)
yaourt -S glxinfo
```

Pip
```
pip install orange3
pip install unicodecsv#
```

Github
```
python-unicodecsv
```


###Additional resources
[Arch Installation Guide](https://wiki.archlinux.org/index.php/Installation_guide)  
[/r/unixporn](https://reddit.com/r/unixporn)
