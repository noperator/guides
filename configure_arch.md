Add a user.
```
useradd -m <USER>
passwd <USER>
echo '<USER> ALL=(ALL) ALL' >> /etc/sudoers
```

Install other utilities/drivers. Note: including `xf86-video-intel` due to an elusive X error, `AddScreen/ScreenInit failed for driver 0` and `drmSetMaster failed: Permission denied`.
```
sudo pacman -S \
i3 xorg-xinit xorg-server xf86-video-intel rxvt-unicode dmenu feh \
git \
qutebrowser \
lastpass-cli \
acpi acpilight \
alsa-utils \
ttf-roboto gnu-free-fonts ttf-font-awesome \
jq
```

Install AUR search agent.
```
git clone https://aur.archlinux.org/auracle-git.git && cd auracle-git
makepkg -si
```

Install AUR utilities.
```
# Add a key to keyring, if necessary.
gpg --recv-key --keyserver hkp://pgp.mit.edu <KEY>

dropbox
dropbox-cli
ttf-symbola
libfprint-vfs0090-git
```

Verify fonts.
```
for TYPEFACE in Sans Serif Monospace; do
    echo "$TYPEFACE: $(fc-match $TYPEFACE)"
done | column -t -s :

Sans        Roboto-Regular.ttf          "Roboto" "Regular"
Serif       FreeSerif.otf               "FreeSerif" "Regular"
Monospace   SourceCodePro-Regular.otf   "Source Code Pro" "Regular"
```

Disable `pinentry` UI.
```
for TYPE in gtk-2 curses; do
    sudo mv "/usr/bin/pinentry-$TYPE" "/usr/bin/pinentry-${TYPE}.bu"
    sudo ln -s /usr/bin/pinentry-tty "/usr/bin/pinentry-$TYPE"
done
```

Install dotfiles.
```
git clone https://github.com/noperator/dotfiles.git && cd dotfiles
./link.sh
```

Enable natural scrolling.
```
sudo tee /etc/X11/xorg.conf.d/30-touchpad.conf > /dev/null << EOF
Section "InputClass"
    Identifier "devname"
    Driver "libinput"
    MatchIsTouchpad "on"
    Option "NaturalScrolling" "true"
EndSection
EOF
```

Allow user to set display brightness (following [this post](https://forum.manjaro.org/t/xbacklight-does-not-have-permission/74061/5)).
```
# Create udev rule to allow users in the video group to set the display brightness.
sudo tee /etc/udev/rules.d/90-backlight.rules > /dev/null << EOF
SUBSYSTEM=="backlight", ACTION=="add",
RUN+="/bin/chgrp video /sys/class/backlight/%k/brightness",
RUN+="/bin/chmod g+w /sys/class/backlight/%k/brightness"
EOF

# Add <USER> to video group.
sudo usermod -a -G video <USER>
```

<!--
Enable fingerprint reader (requires `extra/gobject-introspection` and `libfprint-vfs0090-git`).
```
for /etc/pam.d/system-local-login and /etc/pam.d/sudo:  # Use fingerprint first.
    auth      sufficient pam_fprintd.so  # at top
```
-->
