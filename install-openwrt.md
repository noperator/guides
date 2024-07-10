**Note:** I've flashed/installed this software before but didn't document my steps at the time. Until then, I've just got instructions to perform a factory reset as that's what I've done most recently.

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
