# Arch Linux

https://wiki.archlinux.org/title/Installation_guide



#### Set the console keyboard layout and font
```sh
localectl list-keymaps
```
```sh
loadkeys us
```
```sh
setfont ter-132n
```

#### Connect to a network - iwctl
```sh
iwctl
```
```sh
device list
```
```sh
 station {name} scan
```
```sh
station {name} get-networks
```
```sh
station {name} connect SSID
```
```sh
station {name} show
```
```sh
exit
```

#### Update the system clock
```sh
timedatectl
```

#### Partition the disks
```sh
cfdisk /dev/{device}
```
create partition:

partition 1 - 1G EFI (type)

partition 2 - All storage LVM (type)


#### Drive preparation - dm-crypt
```sh
cryptsetup luksFormat /dev/{partition}
```
```sh
cryptsetup open /dev/sda1 cryptlvm
```

#### Drive preparation - LVM
```sh
pvcreate /dev/mapper/cryptlvm —> create physical volume
```
```sh
vgcreate {MyVolGroup} /dev/mapper/cryptlvm —> create volume group
```
```sh
lvcreate -L 8G -n swap {MyVolGroup}
```
```sh
lvcreate -L 32G -n root {MyVolGroup}
```
```sh
lvcreate -l 100%FREE -n home {MyVolGroup}
```
```sh
lvdisplay
```


#### Format partitions
```sh
mkfs.ext4 /dev/{MyVolGroup}/root
```
```sh
mkfs.ext4 /dev/{MyVolGroup}/home
```
```sh
mkswap /dev/{MyVolGroup}/swap
```
```sh
mkfs.fat -F32 /dev/{device}
```

#### Mount file systems
```sh
mount /dev/{MyVolGroup}/root /mnt
```
```sh
mount --mkdir /dev/{device} /mnt/boot —> Mount the partition to /mnt/boot
```
```sh
mount --mkdir /dev/{MyVolGroup}/home /mnt/home —> Mount the partition to /mnt/home
```
```sh
swapon /dev/{MyVolGroup}/swap
```

#### Install essential packages
```sh
pacstrap -K /mnt base linux-lts linux-firmware intel-ucode
```

### Configure the system


#### Fstab
```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

#### chroot
```sh
arch-chroot /mnt
```

#### Time
```sh
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```
```sh
hwclock --systohc
```

#### Localization
```sh
locale-gen
```
```sh
vim /etc/locale.conf
```
add:
LANG=en_US.UTF-8

#### Network configuration
```sh
/etc/hostname
```
set your hostname


#### Initramfs
```sh
vim /etc/mkinitcpio.conf
```
add: HOOKS=(base systemd lvm2 sd-encrypt consolefont autodetect microcode modconf kms keyboard keymap block filesystems fsck)
```sh
mkinitcpio -P
```

#### Root password
```sh
passwd
```
set password

#### Bootloader
```sh
bootctl install
```
```sh
vim /boot/loader/entries/arch.conf
```
add:  

title   Arch Linux
linux   /vmlinuz-linux-lts
initrd  /initramfs-linux.img

options rd.luks.name=40fcd665-4ef9-44d6-ba2c-e2b46d4d8867=storage root=/dev/storage/root rw

blkid —> shows UUID

```sh
vim /boot/loader/loader.conf
```
add:

beep yes
timeout 5
monitor no
console-mode max


#### install Network
```sh
systemctl enable systemd-networkd.service
```
```sh
systemctl enable systemd-resolved.service
```
```sh
vim /etc/systemd/network/20-wired.network
```

[Match]
Name=enp1s0

[Network]
DHCP=yes
```sh
pacman -Syu iwd
```
```sh
systemctl enable iwd.service
```
#### Set font
```sh
vim /etc/vconsole.conf
```
add:

FONT=ter-132n


#### Create user
```sh
useradd -m -g users -G wheel -s /bin/bash {user}
```
passwd {user}
```sh
EDITOR=vim visudo.
```
uncomment the text below:

%wheel ALL=(ALL:ALL) ALL
```sh
passwd {user}
```
set password

#### install packages

pacman -S lvm2 sudo terminus-font


Reboot!


## Laptop specific settings
```sh
vim /etc/systemd/logind.conf
```
uncomment:

HandlePowerKey=poweroff

HandleLidSwitch=suspend


## ***** Still investigating *****
