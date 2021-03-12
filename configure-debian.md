Generate `sources.list` using [Debian Sources List Generator](https://debgen.simplylinux.ch/).

Release: Stable (Buster)
- [ ] Include source
- [x] Contrib
- [x] Non-Free
- [x] Security
- [x] Updates
- [x] Backports

```
deb http://deb.debian.org/debian/         stable           main contrib non-free
deb http://deb.debian.org/debian/         stable-updates   main contrib non-free
deb http://deb.debian.org/debian-security stable/updates   main
deb http://ftp.debian.org/debian          buster-backports main
```

Install packages.

```
apt install -y sudo
usermod -aG sudo <USER>

sudo apt install -y \
    git vim curl unzip wget jq dnsutils \
    xorg i3 i3blocks rxvt-unicode \
    fonts-roboto fonts-symbola
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

TODO:
- Install `i3-gaps`: https://github.com/maestrogerardo/i3-gaps-deb
