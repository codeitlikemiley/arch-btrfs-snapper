# Special Arch Install With BTRFS on UEFI

1. set wifi password


```sh
wifi-menu
```

2. set mirror list

```sh
vim /etc/pacman.d/mirrorlist
# add this to /etc/pacman.d/mirrorlist
# generate your mirrorlist @ https://www.archlinux.org/mirrorlist/
Server = http://mirror.rise.ph/archlinux/$repo/os/$arch
```
3. Update Packages

```sh
pacman -Syu
```
4. Set Keyboard Layout

5. Partition SSD

```sh
fdisk -l /dev/nvme0n1
cfdisk /dev/nvme0n1
# DELETE ALL CURRENT PARTION SET new Partition as such
/dev/nvme0n1p1 ->type = EFI System
/dev/nvme0n1p2 ->type = Linux Filesystem
/dev/nvme0n1p3 ->type = Linux Filesystem
 /dev/nvme0n1p4 ->type = Linux Swap
```
6. format disk

```sh
mkfs.vfat -F 32 -n EFI /dev/nvme0n1p1
mkfs.btrfs -L ROOT /dev/nvme0n1p2
mkfs.xfs -L HOME /dev/nvme0n1p3
mkswap -L SWAP /dev/nvme0n1p4
```

7. set BTRFS on btrfs partition

```sh
mount /dev/nvme0n1p2 /mnt
btrfs sub cr /mnt/@
btrfs sub cr /mnt/@log
btrfs sub cr /mnt/@pkg
btrfs sub cr /mnt/@snapshots
```

8. Unmount
```sh
umount /mnt
```

9. Mount BTRFS
   
```sh
mount -o relatime,space_cache=v2,compress=lzo,subvol=@ /dev/nvme0n1p2 /mnt
mkdir -p /mnt/{boot/efi,home,var/log,var/cache/pacman/pkg,btrfs}
mount -o relatime,space_cache=v2,compress=lzo,subvol=@log /dev/nvme0n1p2 /mnt/var/log
mount -o relatime,space_cache=v2,compress=lzo,subvol=@pkg /dev/nvme0n1p2 /mnt/var/cache/pacman/pkg/
mount -o relatime,space_cache=v2,compress=lzo,subvolid=5 /dev/nvme0n1p2 /mnt/btrfs
mount /dev/nvme0n1p1 /mnt/boot/efi/
mount /dev/nvme0n1p3 /mnt/home/
swapon /dev/nvme0n1p4
```

10. Check partition

```sh
df -Th
# check memory
free -h
```

11.  Install Arch and Packages

```sh
acstrap /mnt base base-devel bash-completion vim dialog btrfs-progs xfsprogs dosfstools grub efibootmgr linux-lts linux-firmware man-db man-pages inetutils netctl intel-ucode snapper grub networkmanager
```

12. review this command if we need to use wifi-menu not part of installation process
    
```sh
pacman -S wpa_supplicant dialog iw
```

13. Generate FSTAB
    
```sh
genfstab -U /mnt >> /mnt/etc/fstab
# check the file
# remove the line for ROOT partition
cat /mnt/etc/fstab
```

14. GO INSIDE ARCH INSTALLED MACHINE (CHROOT)
    
```sh
arch-chroot /mnt
```

15. Set time zone
```sh
ln -sf /usr/share/zoneinfo/Asia/Manila /etc/localtime
```
16. set hwclock
    
```sh
hwclock --systohc
```

17. localization
    
```sh
vim /etc/locale.gen
```
18. Uncomment en_US.UTF-8 UTF-8 and other needed locales in /etc/locale.gen
    
```sh
vim /etc/locale.conf
```

19. add on /etc/locale.conf
    
```sh
LANG=en_US.UTF-8
```

20. set hostname
    
```sh
vim /etc/hostname
```
21.  Add hostname
```sh
dev
```
22. edit /etc/hosts
    
```sh
vim /etc/hosts
```
23. add this to /etc/hosts
```
127.0.0.1	localhost
::1		localhost
127.0.1.1	dev.localdomain	dev
```
24. create the grub folder

```sh
cd /boot
mkdir grub
# install grub
pacman -S grub
grub-mkconfig -o /boot/grub/grub.cfg
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=grub
```

25. change root pass
```sh
passwd
```
26. avoid error on journald.conf
```sh
vim /etc/systemd/journald.conf
```
27. Update Jourld.conf
```sh
Storage=volatile
```

# install network manager
systemctl enable NetworkManager


vim /etc/mkinitcpio.conf
mkinitcpio -p linux-lts



#exit on arch-chroot
exit
umount -R /mnt

# dont exit unless we install a way to connect to internet :(

)




useradd -m -g wheel uriah
passwd uriah
vim /etc/sudoers



After Installation
https://www.youtube.com/watch?v=TKdZiCTh3EM&feature=youtu.be&t=1218

Next Step

pacman -S snapper
snapper -c root create-config /
btrfs sub del /.snapshots/
mkdir /.snapshots

vim  /etc/fstab
add entry of .snapshots wuth subvol=@snapshots
mount /.snapshots/

df -Th
snapper list
No current snapshots

pacman -S grub-btrfs
open this file
vim /etc/grub.d/41_snapshots-btrfs

Set harmonized entries true on /etc/default/grub
GRUB_BTRFS_CREATE_ONLY_HARMONIZED_ENTRIES="true"
GRUB_BTRFS_LIMIT="10"


vim /etc/snapper/configs/root

NUMBER_CLEANUP="yes"
NUMBER_MIN_AGE="0"
NUMBER_LIMIT="12"
NUMBER_LIMIT_IMPORTANT="3"

TIMELINE_CREATE="no"

systemctl status cronie.service
this should be running


ONLY INSTALL THIS AFTER WE HAVE COMPLETED INSTALLING EVERYTHING
FOR DEVELOPMENT TO SAVE SPACE
sudo pacman -S snap-pac
Yay -y snap-pac-grub

there will be pre-and-post snapshots everytime you install  new pacman package



