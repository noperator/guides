Set up gocryptfs.

```
fido2-token -L
ioreg://<NUM>

mkdir ~/<ENC_LOC>/<ENC> ~/tmp/<ENC>
gpg --decrypt --armor <ENC> | gocryptfs --passfile --init ~/<ENC_LOC>/<ENC>
gocryptfs  -init ~/<ENC_LOC>/<ENC> -fido2 ioreg://NUM
gocryptfs -fido2 ioreg://NUM ~/<ENC_LOC>/<ENC> ~/tmp/<ENC>
gocryptfs -fido2 $(fido2-token -L | sed -E 's/: .*//') ~/<ENC_LOC>/<ENC> ~/tmp/<ENC>

mkdir -p ~/tmp/<ENC>.old
gocryptfs --fido2 ioreg://NUM ~/<ENC_LOC>/<ENC>.old ~/tmp/<ENC>.old
gpg --decrypt --armor ~/<ENC_LOC>/<ENC> |
    gocryptfs --passfile /dev/stdin ~/<ENC_LOC>/<ENC> ~/tmp/<ENC>

gocryptfs --fido2 ioreg://NUM ~/<ENC_LOC>/<ENC> ~/tmp/<ENC>

gocryptfs --passwd --fido2  ~/<ENC_LOC>/<ENC>

gpg --decrypt --armor ~/<ENC_LOC>/<ENC> |
    gocryptfs --passfile /dev/stdin ~/<ENC_LOC>/<ENC> ~/tmp/<ENC>

----------------------------------------------

# make

pwg | tee /dev/stderr | tr -d '\n' | gpg --encrypt -r <EMAIL> --armor
mkdir ~/<ENC_LOC>/<ENC>
op --account my.1password.com item get gocryptfs --fields <ENC> --format json |
    jq -r .value |
    gpg --decrypt --armor |
    gocryptfs --passfile /dev/stdin --init ~/<ENC_LOC>/<ENC>

# mount

op --account my.1password.com item get gocryptfs --fields <ENC> --format json |
    jq -r .value |
    gpg --decrypt --armor |
    gocryptfs --passfile /dev/stdin ~/<ENC_LOC>/<ENC> ~/tmp/<ENC>
```

SSH keys with yubikey

```
ssh-keygen -q -N '' -t ed25519-sk -O resident -f <KEY>
ssh-keygen -N '' -t ed25519-sk -f "$HOME/.ssh/<KEY>"
```

Upgrade OpenSSH to support FIDO

```
apt -t buster-backports install openssh-*
```

Fix docker error

```
failed to solve with frontend dockerfile.v0: failed to create LLB definition: rpc error: code = Unknown desc = open /Users/<USER>/.docker/.token_seed: permission denied
sudo chmod 660 ~/.docker/.token_seed*
```

Reboot WAP at 4a ET (8a UTC)

```
crontab -e
0 8 * * * reboot
```

```
# WAP
opkg update
opkg find openssh-server
openssh-server - 8.4p1-4 - OpenSSH server.
cat /etc/openwrt_release | grep DESC
DISTRIB_DESCRIPTION='OpenWrt 21.02.0 r16279-5cc0535800'
cat /proc/cpuinfo | grep -E 'model|system' | sort -u
cpu model		: MIPS 1004Kc V2.15
system type		: MediaTek MT7621 ver:1 eco:3
cat /etc/opkg/distfeeds.conf | grep _base
src/gz openwrt_base https://downloads.openwrt.org/releases/21.02.0/packages/mipsel_24kc/base

# router
opkg find openssh-server
openssh-server - 7.6p1-1 - OpenSSH server.
cat /etc/openwrt_release | grep DESC
DISTRIB_DESCRIPTION='OpenWrt Designated Driver 50107'
cat /proc/cpuinfo | grep -E 'model|system' | sort -u
system type		: Qualcomm Atheros QCA9558 ver 1 rev 0
cpu model		: MIPS 74Kc V5.0
cat /etc/opkg/distfeeds.conf | grep _base
src/gz designated_driver_base http://downloads.openwrt.org/snapshots/trunk/ar71xx/generic/packages/base


opkg update
opkg install openssh-server

not sure if enabled or overlapping with dropbear

# https://blog.actorsfit.com/a?ID=00200-52669bc5-76fa-43c8-88c8-1d1704c6a1e4

mkdir -p ~/.ssh
vim ~/.ssh/authorized_keys

uci set dropbear.@dropbear[0].Port=2222
uci commit dropbear
/etc/init.d/dropbear restart
sed -i.bu -E 's/^# *(PasswordAuthentication) +.*/\1 no/' /etc/ssh/sshd_config
/etc/init.d/sshd enable
/etc/init.d/sshd start
/etc/init.d/dropbear disable
/etc/init.d/dropbear stop

https://openwrt.org/docs/guide-user/installation/ar71xx.to.ath79
- upgrade from (old) ar71xx to ath79

https://forum.openwrt.org/t/solved-target-vs-architecture/10317/5
- how to get arch from target

https://github.com/openwrt/openwrt/blob/ffd29a55c31697a69f4ebc59305cd95bda82aae3/target/linux/ath79/Makefile#L3-L6
- ath79 means mips 24kc

https://downloads.openwrt.org/releases/22.03.1/packages/mips_24kc/packages/
- packages for that arch

http://downloads.openwrt.org/snapshots/trunk/ar71xx/generic/packages/packages/openssh-server_7.6p1-1_ar71xx.ipk.

vim /etc/opkg/distfeeds.conf
cat /etc/opkg/customfeeds.conf


wget -O /var/opkg-lists/my_packages.sig https://downloads.openwrt.org/releases/22.03.1/packages/mips_24kc/packages/Packages.sig

```

https://support.yubico.com/hc/en-us/articles/360013714379-Accidentally-Triggering-OTP-Codes-with-Your-Nano-YubiKey
https://support.yubico.com/hc/en-us/articles/360016649019-Swapping-Yubico-OTP-from-Slot-1-to-Slot-2

```
ykman otp swap
```

https://github.com/Genymobile/scrcpy#connection

```
scrcpy --tcpip=192.168.1.228
adb tcpip 21086
adb connect 192.168.1.228:21086
scrcpy --tcpip=192.168.1.228:21086

192.168.1.128 0xffffff00
100.98.105.119 0xffffffff
echo '192.168.1.128 0xffffff00' | python3 -c '
```

2023-12-19T21:55:35Z // boox adb scrcpy
```
# toggle usb debug mode off/on
# plug in 
$ adb devices

scrcpy -s $(adb devices | grep -v '^$' | tail -n +2 | cut -wf1)
```

GPG, yubikey

https://musigma.blog/2021/05/09/gpg-ssh-ed25519.html
https://www.reddit.com/r/GnuPG/comments/7wg7ot/confusion_re_master_keys_vs_subkeys/
https://security.stackexchange.com/questions/186649/gpg-masterkey-and-subkey-for-encryption-and-signature-and-default-keys#:~:text=The%20subkey%20with%20E%20flag,the%20validity%20of%20the%20signature.
https://support.yubico.com/hc/en-us/articles/360013790259-Using-Your-YubiKey-with-OpenPGP#Generating_Keys_externally_from_the_YubiKey_(Recommended)ui6vup
https://www.reddit.com/r/yubikey/comments/f77731/storing_gpg_master_key_on_a_separate_yubikey/
https://github.com/drduh/YubiKey-Guide#export-secret-keys

```
gpg --quick-generate-key 'NAME <EMAIL>' ed25519 default never
 export KEYFP=
gpg --quick-add-key $KEYFP cv25519 encr never
gpg --quick-add-key $KEYFP ed25519 auth never
cp -r $GNUPGHOME $GNUPGHOME.bu
gpg --edit-key $KEYFP
mv -vi $GNUPGHOME $GNUPGHOME.c
cp -r $GNUPGHOME.bu $GNUPGHOME
gpg --edit-key $KEYFP
# keytocard, etc.
mv -vi $GNUPGHOME $GNUPGHOME.b
cp -r $GNUPGHOME.bu $GNUPGHOME
...

gpg --armor --export $KEYFP > $GNUPGHOME.bu/mastersub.public.key
gpg --import ~/tmp/gnupg.bu/mastersub.public.key

echo "test message string" | gpg --encrypt --armor -o encrypted.txt
gpg --decrypt --armor encrypted.txt

pwg | tr -d '\n' | gpg --encrypt -r <EMAIL> --armor > <FILE>
gpg --decrypt --armor <FILE>

gpg-connect-agent "scd serialno" "learn --force" /bye

----------------------------------------------

echo "test message string" | gpg --encrypt -r <EMAIL> --armor > testenc
gpg --decrypt --armor testenc

----------------------------------------------

ykman info | grep Serial; gpg-connect-agent "scd serialno" "learn --force" /bye; gpg -k; gpg -K
ykman fido credentials list

ykman openpgp keys set-touch ENC Fixed
```

split flash drive bootable storage

```
wget https://releases.ubuntu.com/22.04.2/ubuntu-22.04.2-desktop-amd64.iso

𝄢 i ubuntu-22.04.2-desktop-amd64.iso
NAME: ubuntu-22.04.2-desktop-amd64.iso
SIZE: 4927586304 (4.6G)
SHA1: e4d2e429feabf67bffaa193915a4c6a89db6df81
TYPE: ISO 9660 CD-ROM filesystem data (DOS/MBR boot sector) 'Ubuntu 22.04.2 LTS amd64' (bootable)

𝄢 diskutil list
/dev/disk5 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *248.0 GB   disk5
   1:             Windows_FAT_32 ⁨USB321FD⁩                247.9 GB   disk5s1

diskutil partitionDisk /dev/disk5 2 GPT exFAT BOOTABLE 5G exFAT STORAGE R
{{{
Started partitioning on disk5
Unmounting disk
Creating the partition map
Waiting for partitions to activate
Formatting disk5s2 as ExFAT with name BOOTABLE
Volume name      : BOOTABLE
Partition offset : 411648 sectors (210763776 bytes)
Volume size      : 9764864 sectors (4999610368 bytes)
Bytes per sector : 512
Bytes per cluster: 32768
FAT offset       : 2048 sectors (1048576 bytes)
# FAT sectors    : 2048
Number of FATs   : 1
Cluster offset   : 4096 sectors (2097152 bytes)
# Clusters       : 152512
Volume Serial #  : 646bdb44
Bitmap start     : 2
Bitmap file size : 19064
Upcase start     : 3
Upcase file size : 5836
Root start       : 4
Mounting disk
Formatting disk5s3 as ExFAT with name STORAGE
Volume name      : STORAGE
Partition offset : 10176512 sectors (5210374144 bytes)
Volume size      : 474183680 sectors (242782044160 bytes)
Bytes per sector : 512
Bytes per cluster: 131072
FAT offset       : 2048 sectors (1048576 bytes)
# FAT sectors    : 16384
Number of FATs   : 1
Cluster offset   : 18432 sectors (9437184 bytes)
# Clusters       : 1852208
Volume Serial #  : 646bdb46
Bitmap start     : 2
Bitmap file size : 231526
Upcase start     : 4
Upcase file size : 5836
Root start       : 5
Mounting disk
Finished partitioning on disk5
}}}


diskutil list
/dev/disk5 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *248.0 GB   disk5
   1:                        EFI ⁨EFI⁩                     209.7 MB   disk5s1
   2:       Microsoft Basic Data ⁨BOOTABLE⁩                5.0 GB     disk5s2
   3:       Microsoft Basic Data ⁨STORAGE⁩                 242.8 GB   disk5s3

𝄢 diskutil unmount /Volumes/BOOTABLE
𝄢 diskutil unmount /Volumes/STORAGE

sudo dd if=ubuntu-22.04.2-desktop-amd64.iso of=/dev/disk5s2 bs=1m

diskutil eject /dev/diskX
```

cloud9 / c9

```
*/25 * * * * [[ $({ screen -ls | grep tached; /usr/sbin/ss -antp | grep ESTAB | grep -E ':22(\s|$)'; } | wc -l) -gt 0 ]] && sudo shutdown -c || { curl -sd "cloud9 shutting down" ntfy.sh/<ADDR>; sudo shutdown -h now; }
0 12 * * * curl -sd "cloud9 uptime: $(uptime -p); screens: $(screen -ls | grep tached | awk '{print $1}' | tr '\n' ' ')" ntfy.sh/<ADDR>
```
