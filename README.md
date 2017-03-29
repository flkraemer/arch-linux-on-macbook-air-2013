# Description

This step-by-step guide describes the install of Arch Linux on a late 2013 Macbook Air 6,2 and how to get audio, video and media hotkeys running. It it relaitvely straightforward and the machine is running very well with Arch. The only thing that doesn't work is the camera. Please find various useful config files for i3wm, Xorg and termite in this repo.

Credit mostly goes to the sources below. I just compiled and sorted out what is needed today.

+ https://github.com/pandeiro/arch-on-air#devsda6---rest-of-space-linux-filesystem-root
+ http://panks.me/posts/2013/06/arch-linux-installation-with-os-x-on-macbook-air-dual-boot/
+ https://wiki.archlinux.org/index.php/Mac

# Hardware

late 2013 Macbook Air 6,2


# Partitions

```
$cgdisk /dev/sda
```

    EFI	/dev/sda1        200MB              HFS+ 
    (or apparently FAT works too, this partition can be made via a gparted live ISO btw) 
    Boot	/dev/sda2    256MB	            Linux Filesystem
    Root	/dev/sda3    <remainign size> 	Linux Filesystem

```
$mkfs.ext4 /dev/sda5
```

```
$mkfs.ext4 /dev/sda6
```

```
$mount /dev/sda6 /mnt
```

```
$mkdir /mnt/boot && mount /dev/sda5 /mnt/boot
```

# Swapfile

```
$dd if=/dev/zero of=/mnt/swapfile bs=1M count=512
```


```
$chmod 600 /mnt/swapfile
```

```
$mkswap /mnt/swapfile
```

# Install
```
$pacstrap /mnt base base-devel
```

```
$genfstab -U -p /mnt >> /mnt/etc/fstab
```

# Optimize FSTAB for SSD
```
$nano /mnt/etc/fstab
```

  /dev/sda6 /     ext4 defaults,noatime,discard,data=writeback 0 1
  /dev/sda5 /boot ext4 defaults,relatime,stripe=4              0 2
  /swapfile none  swap defaults                                0 0

# Configure System
```
$arch-chroot /mnt /bin/bash
```

```
$passwd
```

```
$echo myhostname > /etc/hostname
```

```
$ln -s /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime
```

```
$hwclock --systohc --utc
```

```
$useradd -m -g users -G wheel -s /bin/bash myusername
```

```
$passwd myusername
```

# Install and grant sudo
```
$pacman -S sudo
```

```
$echo "%wheel ALL=(ALL) ALL" > /etc/sudoers.d/10-grant-wheel-group
```

# Locale
```
$nano /etc/locale.gen
```

```
$locale-gen
```

```
$echo LANG=en_US.UTF8 > /etc/locale.conf
```

```
$export LANG=en_US.UTF-8
```

```
$localectl status
```

```
$localectl list-keymaps
```

```
$/etc/vconsole.conf
```

KEYMAP=de-latin1

# MKINITCPIO hooks
Insert “keyboard” after “autodetect” if it’s not already there.
```
$nano /etc/mkinitcpio.conf
```

Then run it:
```
$mkinitcpio -p linux
```

# Install Grub (to use Apples EFI bootloader)
```
$pacman -S grub-efi-x86_64
```

```
$nano /etc/default/grub
```

A special parameter must be set to avoid system (CPU/IO) hangs related to ATA:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet rootflags=data=writeback libata.force=1:noncq"
```

also add

```
#fix broken grub.cfg gen
GRUB_DISABLE_SUBMENU=y
```

```
$grub-mkconfig -o boot/grub/grub.cfg
```

```
$grub-mkstandalone -o boot.efi -d usr/lib/grub/x86_64-efi -O x86_64-efi --compress=xz boot/grub/grub.cfg
```
Copy boot.efi (generated in the command above) to a USB stick for use later in OS X.

# Exit
```
$exit # exit chroot
```

```
$reboot
```

# Back in OSX/MacOS (Boot from install media -> Terminal)
```
$cd /Volumes/disk0s4
```

```
$mkdir System mach_kernel
```

```
$cd System
```

```
$mkdir Library
```

```
$cd Library
```

```
$mkdir CoreServices
```

```
$cd CoreServices
```

```
$touch SystemVersion.plist
```

```
$nano SystemVersion.plist
```

```
<xml version="1.0" encoding="utf-8"?>
<plist version="1.0">
<dict>
    <key>ProductBuildVersion</key>
    <string></string>
    <key>ProductName</key>
    <string>Linux</string>
    <key>ProductVersion</key>
    <string>Arch Linux</string>
</dict>
</plist>
```

Copy boot.efi from your USB stick to this CoreServices directory. The tree should look like this:

    |___mach_kernel
    |___System
         |
         |___Library
                |
                |___CoreServices
                        |
                        |___SystemVersion.plist
                        |___boot.efi

# Bless boot device

```
$sudo bless --device /dev/disk0s4 --setBoot

```

# Disable System Integrity Protection
```
$csrutil disable
```

# LAN
Thunderbolt to Gigabit works out of the box for live iso
DHCP: "dhcpcd ens9"
automate via: ifplugd (settings /etc/ifplugd/ifplugd.conf)

# WIFI
```
$modprobe wl
```

```
$sudo pacman -S dkms
```

```
$sudo systemctl enable dkms.service
```

```
$sudo pacman -S dialog
```

```
$sudo wifi-menu -o
```

Potentially not needed, check if it works first:
```
$enable wpa_supplicant
```
(check for conflicting network services ie dhcpcd)

# Audio
```
$sudo pacman -S alsa-utils
```

```
$sudo pacman -S pulseaudio
```

# Keyboard Hotkeys
```
$yaourt -S pommed-light
```

```
$systemctl enable pommed.service
```


# XORG-Server
```
$sudo pacman -S xf86-video-intel
```

```
$sudo pacman -S xf86-video-vesa
```

/etc/X11/xorg.config.d/ 
[config files in this repo]

~.xinitrc [config files in this repo]

# i3 Window Manager

```
~.config/i3/ [confog files attached]
```
