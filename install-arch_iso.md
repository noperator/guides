Download latest ISO from [Arch Linux Downloads page](https://www.archlinux.org/download/), identify flash drive device path, and write ISO to flash drive.
```
lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 238.5G  0 disk
└─sda1   8:1    0 238.5G  0 part /
sdb      8:16   1  14.5G  0 disk
└─sdb1   8:17   1  14.5G  0 part

sudo dd bs=4M if=archlinux-2020.02.01-x86_64.iso of=/dev/sdb status=progress oflag=sync
```

After restarting and booting live environment, connect to Internet and update clock.
```
wifi-menu
ping -c 1 archlinux.org
timedatectl set-ntp true
```

Partition disk (helpful `sfdisk` [guide on StackExchange](https://superuser.com/a/1132834)). Don't create swap partition—create swap file later on instead.
```
# Check disks.
lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
loop0    7:0    0 531.2M  1 loop /run/archiso/sfs/airootfs
sda      8:0    0 238.5G  0 disk 
└─sda1   8:1    0 238.5G  0 part 
sdb      8:16   1  14.5G  0 disk 
├─sdb1   8:17   1   646M  0 part /run/archiso/bootmnt
└─sdb2   8:18   1    64M  0 part 

# Partition disk.
echo 'type=83' | sfdisk /dev/sda
Device     Boot Start       End   Sectors   Size Id Type
/dev/sda1        2048 500118191 500116144 238.5G 83 Linux
```

Format and mount file system.
```
mkfs.ext4 /dev/sda1
mount /dev/sda1 /mnt
```

Install essential packages. Takes about an hour.
```
pacstrap /mnt \
base base-devel \
linux linux-firmware \
grub \
netctl wpa_supplicant dhcpcd dialog \
gvim gnu-netcat
```

Generate fstab and change root into new system.
```
genfstab -L /mnt >> /mnt/etc/fstab
arch-chroot /mnt
```

[Create swap file](https://wiki.archlinux.org/index.php/Swap#Swap_file_creation). According to [this StackExchange post](https://unix.stackexchange.com/a/467498), add swap space in the order of `round(sqrt(RAM))`.

```
fallocate -l 3G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap defaults 0 0' >> /etc/fstab
```

Set time zone and hardware clock.
```
ln -sf /usr/share/zoneinfo/<REGION>/<CITY> /etc/localtime
hwclock --systohc
```

Uncomment locales, and generate them.
```
sed -i -E 's/#(en_US.UTF-8 UTF-8)/\1/' /etc/locale.gen
locale-gen
echo 'LANG=en_US.UTF-8' > /etc/locale.conf
```

Set hostname.
```
echo <HOSTNAME> > /etc/hostname
echo -e '127.0.0.1 localhost\n::1 localhost\n127.0.1.1 <HOSTNAME>.localdomain <HOSTNAME>' >> /etc/hosts
```

Set password.
```
passwd
```

Install bootloader.
```
grub-install --target=i386-pc /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

Exit chroot environment and reboot.
```
exit
reboot
```
