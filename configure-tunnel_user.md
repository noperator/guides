This will create a locked-down `ssh_tunnel` user who'll only be allowed to open SSH tunnels on a remote server. Note that this user will be allowed to authenticate with a password, even if the broader SSH configuration doesn't allow that.

Create user.
```
sudo useradd ssh_tunnel -m -d /home/ssh_tunnel -s /bin/bash
```

Set password.
```
echo "ssh_tunnel:$(base64 /dev/urandom | tr -d '/+=' | head -c 32)" | tee /dev/tty | sudo chpasswd
```

Disable login shell.
```
sudo usermod -s /usr/sbin/nologin ssh_tunnel
```

Remove MOTD.
```
sudo touch /home/ssh_tunnel/.hushlogin
```

Lock down SSH settings to only allow tunneling.
```
sudo tee -a /etc/ssh/sshd_config > /dev/null << EOF
Match User ssh_tunnel
  AllowTcpForwarding yes
  X11Forwarding no
  PermitTunnel no
  GatewayPorts clientspecified
  AllowAgentForwarding no
  PasswordAuthentication yes
EOF
```

Restart SSHD.
```
sudo service sshd restart
```
