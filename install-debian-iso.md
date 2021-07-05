Download latest ISO from [Debian downloads page](https://www.debian.org/distrib/), identify flash drive device path, and write ISO to flash drive.

```
NAME: debian-10.8.0-amd64-netinst.iso
SIZE: 352321536 (337M)
TYPE: ISO 9660 CD-ROM filesystem data (DOS/MBR boot sector) 'Debian 10.8.0 amd64 n' (bootable)
SHA1: a23693ca42ffa8ba6c3fa8d3b0a30f4509c9f9bd

lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 238.5G  0 disk
└─sda1   8:1    0 238.5G  0 part /
sdb      8:16   1  14.5G  0 disk
└─sdb1   8:17   1  14.5G  0 part

pv debian-10.8.0-amd64-netinst.iso | sudo dd bs=4M of=/dev/sdb oflag=sync
336MiB 0:01:26 [3.87MiB/s] [=========================================================>] 100%
0+5376 records in
0+5376 records out
352321536 bytes (352 MB, 336 MiB) copied, 87.0314 s, 4.0 MB/s
```
