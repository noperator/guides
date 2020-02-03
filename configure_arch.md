Add a user.
```
useradd -m <USER>
passwd <USER>
echo '<USER> ALL=(ALL) ALL' >> /etc/sudoers
```

Install other utilities.
```
sudo pacman -S \
i3 xorg-xinit xorg-server rxvt-unicode dmenu \
git \
qutebrowser \
lastpass-cli \
acpi acpilight
```

Install AUR search agent.
```
git clone https://aur.archlinux.org/auracle-git.git
makepkg -si
```

Install AUR utilities.
```
# Add a key to keyring, if necessary.
gpg --recv-key --keyserver hkp://pgp.mit.edu <KEY>

dropbox
dropbox-cli
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
git clone https://github.com/noperator/dotfiles.git
cd dotfiles
./link.sh
```
