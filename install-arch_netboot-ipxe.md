From [Arch Linux Netboot](https://www.archlinux.org/releng/netboot/), download `ipxe.lkrn`.
```
wget https://www.archlinux.org/static/netboot/ipxe.419cd003a298.lkrn
NAME: ipxe.419cd003a298.lkrn
SIZE: 351588
TYPE: Linux kernel x86 boot executable bzImage, version 1.0.0+ (53af9), RO-rootFS,
SHA1: 9175e3911a638e6984cad9ace9859b5c09085d4e
```

Identify flash drive device path.
```
lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 238.5G  0 disk
└─sda1   8:1    0 238.5G  0 part /
sdb      8:16   1  14.5G  0 disk
└─sdb1   8:17   1  14.5G  0 part
```

Create MBR partition table, create primary partition with FAT32, flag as active, and create FAT32 filesystem. Should really use `sfdisk` here instead.
```
sudo fdisk /dev/sdb

# Create MBR partition table.
Command (m for help): o
Created a new DOS disklabel.

# Create primary partition (use defaults).
Command (m for help): n
Do you want to remove the signature? [Y]es/[N]o: n
Created a new partition 1 of type 'Linux' and of size 14.5 GiB.

# Set FAT32 partition type (b == W95 FAT32).
Command (m for help): t
Hex code (type L to list all codes): b
Changed type of partition 'Linux' to 'W95 FAT32'.

# Mark as active.
Command (m for help): a
The bootable flag on partition 1 is enabled now.

# Verify partition table.
Command (m for help): p
Device     Boot Start      End  Sectors  Size Id Type
/dev/sdb1  *     2048 30310399 30308352 14.5G  b W95 FAT32

# Write changes.
Command (m for help): w
The partition table has been altered.

# Create filesystem.
sudo mkfs.fat -F 32 /dev/sdb1
mkfs.fat 4.1 (2017-01-24)
```

Mount partition, create `boot/syslinux/` and copy `ipxe.lkrn` into `boot/`.
```
sudo mount /dev/sdb1 /mnt
sudo mkdir -p /mnt/boot/syslinux
sudo cp ipxe.419cd003a298.lkrn /mnt/boot/ipxe.lkrn
```

Create Syslinux config.
```
sudo vim /mnt/boot/syslinux/syslinux.cfg
DEFAULT arch_netboot
   SAY Booting Arch over the network.
LABEL arch_netboot
   KERNEL /boot/ipxe.lkrn
```

Unmount partition.
```
sudo umount /mnt
```

Install Syslinux MBR, and then Syslinux itself.
```
sudo dd bs=440 count=1 conv=notrunc if=/usr/lib/syslinux/bios/mbr.bin of=/dev/sdb
sudo syslinux --directory /boot/syslinux/ --install /dev/sdb1
```

Final flash drive contents.
```
tree -Fs /mnt
/mnt
└── [  8192]  boot/
    ├── [351588]  ipxe.lkrn*
    └── [  8192]  syslinux/
        ├── [119668]  ldlinux.c32*
        ├── [ 60416]  ldlinux.sys*
        └── [   104]  syslinux.cfg*

file /mnt/boot/syslinux/ldlinux.*
ldlinux.c32: ELF 32-bit LSB shared object, Intel 80386, version 1 (SYSV), dynamically linked, stripped
ldlinux.sys: SYSLINUX loader (version 6.04)
```

Build custom iPXE image.
```
git clone https://aur.archlinux.org/ipxe-netboot.git
echo -e '#!ipxe\nshell' > arch.ipxe
# Replace second sha256sum, corresponding to arch.ipxe, with SKIP.
makepkg -si
mkdir ipxe-netboot-r5931.18dc73d2-1-x86_64
tar -C ipxe-netboot-r5931.18dc73d2-1-x86_64 -xvJf ipxe-netboot-r5931.18dc73d2-1-x86_64.pkg.tar.xz
```

Test iPXE image.
```
sudo pacman -S ovmf qemu
qemu-system-x86_64 -enable-kvm -m 2G -kernel ipxe.lkrn

wget https://www.archlinux.org/static/netboot/ipxe.419cd003a298.lkrn
qemu-system-x86_64 -enable-kvm -m 2G -kernel ipxe.419cd003a298.lkrn
```
