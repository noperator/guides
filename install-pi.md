From macOS, write Raspbian to SD card.
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

Confirm log in. Networked via Ethernet/DHCP.
```
ssh pi@raspberrypi
# raspberry
```

Generate SSH key, transfer, and confirm.
```
ssh-keygen -t rsa -b 4096 -o -a 100 -q -N '' -f ~/.ssh/pi
ssh-copy-id -i ~/.ssh/pi pi@raspberrypi
ssh -i ~/.ssh/pi pi@raspberrypi 'uname -a'ssh -i ~/.ssh/pi pi@raspberrypi 'uname -a'
```

Set password.
```
echo "pi:$(base64 /dev/urandom | tr -d '/+=' | head -c 32)" | tee /dev/tty | sudo chpasswd
```

Update.
```
sudo apt update
sudo apt upgrade -y
```

Install display server, browser, and other essentials.
```
sudo apt install -y \
xserver-xorg-core x11-xserver-utils xinit xdotool \
chromium-browser \
git vim jq
```

Get dotfiles.
```
git clone https://github.com/noperator/dotfiles
```

Fix `startx` error, "Only console users are allowed to run the X server."
```
sudo sed -i -E 's/(allowed_users=)console/\1anybody/' /etc/X11/Xwrapper.config
```
