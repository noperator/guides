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
    software-properties-common gpg \
    bridge-utils
```

Disable `pinentry` UI.

```
for TYPE in gtk-2 curses; do
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
get -O - https://www.dropbox.com/download?plat=lnx.x86_64 | tar -xvzf -

sudo wget -O /usr/local/bin/dropbox https://www.dropbox.com/download?dl=packages/dropbox.py &&
sudo chmod +x /usr/local/bin/dropbox
```

Fix video tearing.

```
# sudo mkdir /etc/X11/xorg.conf.d
sudo tee /etc/X11/xorg.conf.d/20-intel.conf > /dev/null << EOF
Section "Device"
    Identifier "Intel Graphics"
    Driver "intel"
    Option "TearFree" "true"
EndSection
EOF
```

Configure and start Barrier server.

```
mkdir -p ~/.local/share/barrier/SSL/SSL/Fingerprints
openssl req -x509 -nodes -days 365 -subj /CN=Barrier -newkey rsa:2048 -keyout ~/.local/share/barrier/SSL/Barrier.pem -out ~/.local/share/barrier/SSL/Barrier.pem
openssl x509 -fingerprint -sha1 -noout -in ~/.local/share/barrier/SSL/Barrier.pem > ~/.local/share/barrier/SSL/SSL/Fingerprints/Local.txt

barriers --enable-crypto
```

[Create network bridge](https://wiki.archlinux.org/index.php/Network_bridge#With_iproute2).

```
sudo ip link add name br0 type bridge
sudo ip link set br0 up

for IFACE in eno1 enp2s0; do
    sudo ip link set "$IFACE" up
    sudo ip addr flush dev "$IFACE"
    sudo ip link set "$IFACE" master br0
done

sudo dhclient -v br0
```

TODO:
- Configure firewall
