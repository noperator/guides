## General

Generate `sources.list` using [Debian Sources List Generator](https://debgen.simplylinux.ch/).

Release: Stable (Buster)
- [x] Include source
- [x] Contrib
- [x] Non-Free
- [x] Security
- [x] Updates
- [x] Backports

```
deb      http://deb.debian.org/debian/          stable            main  contrib  non-free
deb-src  http://deb.debian.org/debian/          stable            main  contrib  non-free
deb      http://deb.debian.org/debian/          stable-updates    main  contrib  non-free
deb      http://deb.debian.org/debian-security  stable/updates    main
deb      http://ftp.debian.org/debian           buster-backports  main
```

Install packages.

```
apt install -y sudo
usermod -aG sudo <USER>

sudo apt install -y \
    git vim-gtk3 curl unzip wget jq dnsutils exa lastpass-cli pinentry-tty ripgrep \
    xorg i3 i3blocks rxvt-unicode \
    fonts-roboto fonts-symbola \
    software-properties-common gpg snapd \
    bridge-utils pulseaudio sshfs

sudo snap install core
sudo snap install --classic code
```

Disable `pinentry` UI.

```
for TYPE in gnome3 gtk-2 curses; do
    sudo mv "/usr/bin/pinentry-$TYPE" "/usr/bin/pinentry-${TYPE}.bu"
    sudo ln -s /usr/bin/pinentry-tty "/usr/bin/pinentry-$TYPE"
done
```

Install some fonts manually:
- [Source Code Pro](https://askubuntu.com/a/193073)
- [Font Awesome](https://fontawesome.com/download)

```
wget https://github.com/adobe-fonts/source-code-pro/releases/download/2.038R-ro%2F1.058R-it%2F1.018R-VAR/TTF-source-code-pro-2.038R-ro-1.058R-it.zip
mkdir sourcecodepro
unzip -d sourcecodepro TTF-source-code-pro-2.038R-ro-1.058R-it.zip
mv sourcecodepro/*.ttf ~/.local/share/fonts
fc-cache -f -v
rm -r sourcecodepro TTF-source-code-pro-2.038R-ro-1.058R-it.zip
```

[Update kernel](https://unix.stackexchange.com/a/420682) to fix:
- max resolution issues
- `xrandr` error: "failed to get size of gamma for output default"
- `startx` error: "unable to find valid framebuffer device" when starting X server"

```
sudo apt -t buster-backports install linux-image-amd64
```

Install Chrome. Also installs Google repository for automatic updates—but it's unsigned?

```
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install -y ./google-chrome-stable_current_amd64.deb
```

Install `i3-gaps`.

```
git clone https://github.com/maestrogerardo/i3-gaps-deb && cd i3-gaps-deb
./i3-gaps-deb
```

Install Dropbox daemon and client.

```
cd &&
get -O - https://www.dropbox.com/download?plat=lnx.x86_64 |
    tar -xvzf -

sudo wget -O /usr/local/bin/dropbox https://www.dropbox.com/download?dl=packages/dropbox.py &&
sudo chmod +x /usr/local/bin/dropbox
```

[Install RTL-SDR](https://ranous.files.wordpress.com/2020/05/rtl-sdr4linux_quickstartguidev20.pdf).

```
sudo apt install -y libusb-1.0-0-dev

git clone git://git.osmocom.org/rtl-sdr.git
cd rtl-sdr/
mkdir build
cd build
cmake ../ -DINSTALL_UDEV_RULES=ON
make
sudo make install
sudo ldconfig
sudo cp ../rtl-sdr.rules /etc/udev/rules.d/

sudo tee /etc/modprobe.d/blacklist-rtl.conf > /dev/null << EOF
blacklist dvb_usb_rtl28xxu
EOF

rtl_test -s 2400000
```

[Install Docker](https://docs.docker.com/engine/install/debian/).

```
# Install dependencies.
sudo apt update
sudo apt install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

# Add Docker’s official GPG key.
curl -fsSL https://download.docker.com/linux/debian/gpg |
    sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Set up Docker repo.
echo \
        "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
        $(lsb_release -cs) stable" |
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine.
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io

# Verify installation.
sudo docker run hello-world

# If needed, re-enable IP bridge forwarding after Docker disables it.
sudo ufw route allow in on br0 out on br0
```

Install VirtualBox.

```
sudo tee /etc/apt/sources.list.d/virtualbox.list > /dev/null << EOF
deb http://download.virtualbox.org/virtualbox/debian buster contrib
EOF

wget https://www.virtualbox.org/download/oracle_vbox_2016.asc
sudo apt-key add oracle_vbox_2016.asc

sudo apt update
sudo apt install -y virtualbox-6.1
```

[Install Typora](https://typora.io/#linux).

```
# Add Typora key and repo.
wget -qO - 'https://typora.io/linux/public-key.asc' |
    sudo apt-key add - &&
sudo add-apt-repository 'deb https://typora.io/linux ./' &&
sudo apt update

# Install Typora.
sudo apt install typora
```

Fix [instantaneous wakeups from suspend](https://wiki.archlinux.org/index.php/Power_management/Suspend_and_hibernate#Instantaneous_wakeups_from_suspend) [triggered by USB devices](https://bbs.archlinux.org/viewtopic.php?pid=1575617#p1575617).

```
sudo tee /etc/tmpfiles.d/100-disable-usb-wake.conf > /dev/null << EOF
#    Path                  Mode UID  GID  Age Argument
w    /proc/acpi/wakeup     -    -    -    -   PTXHXHC0
EOF
```

[Save layout.](https://i3wm.org/docs/layout-saving.html)

```
i3-save-tree --workspace 10 > ~/.config/i3/workspace-av.json

i3-msg "workspace 10; append_layout ~/.config/i3/workspace-av.json"
```

[Install _latest_ Node.js](https://github.com/nodesource/distributions#installation-instructions).

```
# Check existing version.
for BIN in node npm; do echo -n "$BIN:"; "$BIN" -v; done
node:v10.24.0
npm:7.4.0

# Add Node.js PPA. Already had the dependencies, so didn't run this.
# sudo apt install curl software-properties-common
curl -fsSL 'https://deb.nodesource.com/setup_17.x' | sudo bash -

# Install Node.js.
sudo apt install -y nodejs

# Check new version.
for BIN in node npm; do echo -n "$BIN:"; "$BIN" -v; done
node:v16.4.1
npm:7.18.1
```

## Networking

[Create network bridge](https://wiki.archlinux.org/index.php/Network_bridge#With_iproute2).

```
sudo ip link add name br0 type bridge
sudo ip link set br0 up

for IFACE in eno1 enp3s0; do
    sudo ip link set "$IFACE" up
    sudo ip addr flush dev "$IFACE"
    sudo ip link set "$IFACE" master br0
done

sudo dhclient -v br0
```

Enable automatic networking. See [info on bridges](https://wiki.archlinux.org/index.php/systemd-networkd#Network_bridge_with_DHCP).

```
for SVC in networking NetworkManager
do
    sudo systemctl stop "$SVC"
    sudo systemctl disable "$SVC"
done
sudo systemctl enable --now systemd-networkd

sudo tee /etc/systemd/network/10-bridge.netdev > /dev/null << EOF
[NetDev]
Name=br0
Kind=bridge
EOF

sudo tee /etc/systemd/network/11-bind.network > /dev/null << EOF
[Match]
Name=en*

[Network]
Bridge=br0
EOF

sudo tee /etc/systemd/network/12-bridge.network > /dev/null << EOF
[Match]
Name=br0

[Network]
DHCP=yes
EOF

sudo systemctl restart systemd-networkd &&
sudo systemctl status systemd-networkd
```

Temporarily disable IPv6.

```
sudo sysctl -w net.ipv6.conf.all.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.default.disable_ipv6=1
sudo sysctl -w net.ipv6.conf.lo.disable_ipv6=1
```

Configure and start Barrier server.

```
mkdir -p ~/.local/share/barrier/SSL/SSL/Fingerprints
openssl req -x509 -nodes -days 365 -subj /CN=Barrier -newkey rsa:2048 -keyout ~/.local/share/barrier/SSL/Barrier.pem -out ~/.local/share/barrier/SSL/Barrier.pem
openssl x509 -fingerprint -sha1 -noout -in ~/.local/share/barrier/SSL/Barrier.pem > ~/.local/share/barrier/SSL/SSL/Fingerprints/Local.txt

barriers --enable-crypto
```

Mount remote folder via SSHFS.

```
sshfs -o idmap=user <USER>@<HOST>:<REMOTE-FOLDER> <LOCAL-MOUNT>
```

Open firewall port.

```
sudo ufw
sudo ufw allow 12345/tcp proto tcp from any to any port 80,443 comment 'my cool web app ports'

sudo ufw status numbered
sudo ufw delete <N>
```

VNC server.

```
sudo apt install -y tigervnc-scraping-server
vncpassword

# x0vncserver -passwordfile ~/.vnc/passwd -display :0
x0vncserver -display :0

# vncserver :1 -localhost no -geometry 1920x1080
vncserver :1 -localhost no
```

Change hostname.

```
sudo hostnamectl set-hostname <HOSTNAME>
sudoedit /etc/hosts  # Add new hostname, remove old one.
```

Add a printer.

```
sudo lpadmin -E \
    -p '<NAME>' \
    -v "ipp://<HOSTNAME>:631/ipp/print" \
    -m "driverless:ipp://<HOSTNAME>:631/ipp/print"
```

## Audio and video

[Stream audio](https://wiki.archlinux.org/index.php/PulseAudio/Examples#PulseAudio_over_network) from Linux to macOS.

- https://webcache.googleusercontent.com/search?q=cache:Gdrl14DTGzIJ:https://blogs.gnome.org/ignatenko/2015/07/31/how-to-set-up-network-audio-server-based-on-pulseaudio-and-auto-discovered-via-avahi/+&cd=2&hl=en&ct=clnk&gl=us
- https://manurevah.com/blah/en/p/PulseAudio-Sound-over-the-network
- https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Modules/#module-tunnel-sinksource
- https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Network/
- https://unix.stackexchange.com/questions/520724/how-to-robustly-switch-pulseaudio-output-device-from-command-line

```
# Set up macOS server.
brew install pulseaudio
pulseaudio --load=module-native-protocol-tcp --exit-idle-time=-1 --daemon

# Configure Linux client.
# Copy ~/.config/pulse/cookie from macOS server to Linux client.
sudo sed -i -E 's/.*(default-server =).*/\1 <SERVER-HOST>/' /etc/pulse/client.conf
```

Virtual webcam.

- https://askubuntu.com/a/881341
- https://obsproject.com/wiki/Launch-Parameters

```
sudo apt install -y linux-headers-$(uname -r)
sudo apt install -t buster-backports -y v4l2loopback*

sudo tee /etc/modprobe.d/v4l2loopback.conf > /dev/null << EOF
options v4l2loopback exclusive_caps=1 card_label=OBSVirtualCamera
EOF

sudo tee /etc/modules-load.d/v4l2loopback.conf > /dev/null << EOF
v4l2loopback
EOF

obs-studio --startvirtualcam &
```

[Disable microphone on USB webcam](https://www.mjt.me.uk/posts/blacklisting-certain-microphones-linux/).

```
lsusb | grep 'Logitech'
Bus 001 Device 010: ID 046d:08e5 Logitech, Inc.

sudo tee /etc/udev/rules.d/90-blacklist-webcam-sound.rules > /dev/null << EOF
# Disable webcam audio.
SUBSYSTEM=="usb", DRIVER=="snd-usb-audio", ATTRS{idVendor}=="046d", ATTRS{idProduct}=="08e5", ATTR{authorized}="0"
EOF

sudo udevadm control --reload-rules
```

[Configure GPU](https://wiki.debian.org/AtiHowTo).

- https://unix.stackexchange.com/a/420682
- remove `xserver-xorg-video-intel`

The drivers you need are:

```
firmware-linux
firmware-linux-nonfree
firmware-misc-nonfree
intel-microcode
amd64-microcode
```

```
lspci -k
inxi -G

sudo apt install -y firmware-amd-graphics libgl1-mesa-dri libglx-mesa0 mesa-vulkan-drivers xserver-xorg-video-all

sudo rm /etc/X11/xorg.conf.d/20-intel.conf
sudo tee /etc/X11/xorg.conf.d/20-amdgpu.conf > /dev/null << EOF
Section "Device"
   Identifier  "AMD Graphics"
   Driver      "amdgpu"
   Option      "TearFree"  "true"
EndSection
EOF
```

Configure GLX (https://unix.stackexchange.com/q/334623). Pick `mesa-diverted` for AMD.

```
sudo update-glx --config glx
```

Arrange displays.

```
xrandr --output HDMI-A-1 --mode 2560x1080 --pos 1920x0 --output HDMI-A-0 --mode 1920x1080 --pos 0x0

sudo tee /etc/X11/xorg.conf.d/10-monitor.conf > /dev/null << EOF
Section "Monitor"
  Identifier "HDMI-A-1"
  Option "PreferredMode" "2560x1080"
  Option "RightOf" "HDMI-A-0"
EndSection

Section "Monitor"
  Identifier "HDMI-A-0"
  Option "PreferredMode" "1920x1080"
  Option "LeftOf" "HDMI-A-1"
EndSection
EOF
```

[Connect AirPods to Bluetooth](https://askubuntu.com/a/1063582).

```
sudo apt install -u bluetooth bluez blueman

sudo sed -i -E 's/.*(ControllerMode =).*/\1 bredr/' /etc/bluetooth/main.conf
sudo systemctl restart bluetooth
blueman-manager
```

Rename audio devices.
- https://forums.linuxmint.com/viewtopic.php?t=324124

```
pacmd list-sinks | grep -E 'name:|device\.description ='
	name: <alsa_output.pci-0000_01_00.1.hdmi-stereo-extra3>
		device.description = "Ellesmere HDMI Audio [Radeon RX 470/480 / 570/580/590] Digital Stereo (HDMI 4)"
	name: <alsa_output.usb-Blue_Microphones_Yeti_Nano_2105SG00E768_888-000444040606-00.analog-stereo>
		device.description = "Yeti Nano Analog Stereo"
	name: <alsa_output.pci-0000_00_1f.3.analog-stereo>
		device.description = "Built-in Audio Analog Stereo"

pacmd 'update-sink-proplist alsa_output.pci-0000_01_00.1.hdmi-stereo-extra3 device.description="HDMI Monitor"'
pacmd 'update-sink-proplist alsa_output.pci-0000_00_1f.3.analog-stereo device.description="Analog Headphones"'
```
