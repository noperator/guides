**Note:** I've flashed/installed this software before but didn't document my steps at the time. Until then, I've just got instructions to perform a factory reset as that's what I've done most recently.

# Factory Reset

Perform a [Factory Reset](https://openwrt.org/docs/guide-user/troubleshooting/failsafe_and_factory_reset#factory_reset); i.e., erase all your packages and settings, returning the router to its initial state after installing OpenWrt.

```
firstboot
reboot now
```

## Upgrade

[Upgrade via CLI](https://openwrt.org/docs/guide-user/installation/sysupgrade.cli#command-line_instructions). Pretty painless. Just did the hash check manually.

```
FILE=openwrt-22.03.0-ath79-generic-<PLATFORM>-squashfs-sysupgrade.bin
cd /tmp
wget "https://downloads.openwrt.org/releases/22.03.0/targets/ath79/generic/$FILE"
# check hashes
sysupgrade -v "$FILE"
```

# TP-Link TL-WA801ND v5

https://openwrt.org/toh/tp-link/tl-wa801nd#:~:text=23.05.5-,Factory%20image,-Sysupgrade%20image

```
ssh <ROUTER> 'cat /proc/net/arp' |
    tail -n +2 |
    awk '{print $1, $4}' |
    grep -v '00:00:00:00:00:00' |
    while read IP MAC; do
        echo -n "$IP "
        cat <OUI_FILE> |
            rg $(echo "$MAC" | cut -d : -f 1,2,3 | tr -d :)
        echo
    done |
    grep -v '^$' |
    grep -iE 'tp.link' |
    awk '{print $1}' |
    sort -u |
    while read IP; do
        ncat -nvzw1 "$IP" 80
    done |&
    grep 'Connected to'

ls -l /dev/mtd* /usr/bin/tddp /usr/bin/reg
-rwxrwxr-x    1     10287 /usr/bin/reg
-rwxrwxr-x    1     50820 /usr/bin/tddp
brw-rw-r--    1   31,   6 /dev/mtdblock6
brw-rw-r--    1   31,   5 /dev/mtdblock5
brw-rw-r--    1   31,   4 /dev/mtdblock4
brw-rw-r--    1   31,   3 /dev/mtdblock3
brw-rw-r--    1   31,   2 /dev/mtdblock2
brw-rw-r--    1   31,   1 /dev/mtdblock1
brw-rw-r--    1   31,   0 /dev/mtdblock0
crw-rw-r--    1   90,  12 /dev/mtd6
crw-rw-r--    1   90,  10 /dev/mtd5
crw-rw-r--    1   90,   8 /dev/mtd4
crw-rw-r--    1   90,   6 /dev/mtd3
crw-rw-r--    1   90,   4 /dev/mtd2
crw-rw-r--    1   90,   2 /dev/mtd1
crw-rw-r--    1   90,   0 /dev/mtd0
brw-rw-r--    1   31,   0 /dev/mtd

cat /proc/mtd
dev:    size   erasesize  name
mtd0: 00020000 00010000 "boot"
mtd1: 00140000 00010000 "kernel"
mtd2: 00660000 00010000 "rootfs"
mtd3: 00010000 00010000 "config"
mtd4: 00010000 00010000 "romfile"
mtd5: 00010000 00010000 "rom"
mtd6: 00010000 00010000 "radio"

tddp --help
[cmd_dutInit():697] init shm
[tddp_taskEntry():151] tddp task start

wget https://downloads.openwrt.org/releases/23.05.5/targets/ramips/mt76x8/openwrt-23.05.5-ramips-mt76x8-tplink_tl-wa801nd-v5-squashfs-tftp-recovery.bin
```
