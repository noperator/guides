Note: When using macOS to interact with devices, use `/dev/rdiskX` (note the `r`) when possible (i.e., if it doesn't cause an `Input/output error`) as it [tends to be much faster](https://superuser.com/a/631601); `rdisk` is a "raw" character device (closer to the physical disk) while `disk` is a buffered block device.

If needed, mount the filesystem on macOS (read-only) and back up files before wiping the SD card.

```
brew cask install osxfuse
brew install ext4fuse
sudo mkdir /Volumes/rpi
sudo ext4fuse /dev/diskXsX /Volumes/rpi -o allow_other
```

From macOS, write Raspbian Lite to SD card. Took 5:45 for a 1.72 GB image (5.10 MB/s) when using `rdisk`.

```
pv 2019-09-26-raspbian-buster-lite.img | sudo dd bs=1m of=/dev/diskX
```

Support iMac G4 display via `boot/config.txt`.

```
hdmi_group=2  # DMT
hdmi_mode=47  # 1440x900 @ 60Hz
```

Enable SSH.

```
touch boot/ssh
```

If needed, enable Wi-Fi.

```
tee boot/wpa_supplicant.conf > /dev/null << EOF
country=US
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="<SSID>"
    psk="<PSK>"
}
EOF
```

Confirm log in. Networked via Ethernet/DHCP.

```
ssh pi@raspberrypi
# raspberry
```

Generate SSH key, transfer, and confirm.

```
ssh-keygen -t rsa -b 4096 -o -a 100 -q -N '' -f ~/.ssh/pi
ssh-copy-id -i ~/.ssh/pi pi@raspberrypi
ssh -i ~/.ssh/pi pi@raspberrypi 'uname -a'
```

Set password.

```
echo "pi:$(base64 /dev/urandom | tr -d '/+=' | head -c 32)" | tee /dev/tty | sudo chpasswd
```

Disable SSH password authentication.

```
sudo sed -i -E 's/^[# ]*(PasswordAuthentication) (yes|no)$/\1 no/' /etc/ssh/sshd_config
sudo service sshd restart
```

Update.

```
sudo apt update
sudo apt upgrade -y
```

Install display server, browser, and other essentials.

```
sudo apt install -y \
xserver-xorg-core x11-xserver-utils xinit \
chromium-browser \
git vim jq
```

Install dotfiles.

```
git clone https://github.com/noperator/dotfiles.git && cd dotfiles
./link.sh
```

Set country code [non-interactively](https://raspberrypi.stackexchange.com/a/66939) to remove login banner, "Wi-Fi is currently blocked by `rfkill`. Use `raspi-config` to set the country before use." While we're here, set the time zone, too.

```
sudo raspi-config nonint do_wifi_country US
sudo raspi-config nonint do_change_timezone US/Eastern
```

Fix `startx` error, "Only console users are allowed to run the X server."

```
sudo sed -i -E 's/(allowed_users=)console/\1anybody/' /etc/X11/Xwrapper.config
```

[Rotate display](https://www.raspberrypi-spy.co.uk/2017/11/how-to-rotate-the-raspberry-pi-display-output/).

```
echo 'display_rotate=1' | sudo tee -a /boot/config.txt
```
