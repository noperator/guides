from macos
- https://thornelabs.net/posts/create-a-bootable-ubuntu-usb-drive-in-os-x/

```
diskutil list

/dev/disk5 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     Apple_partition_scheme                        *15.5 GB    disk5
   1:        Apple_partition_map                         4.1 KB     disk5s1
   2:                  Apple_HFS                         2.7 MB     disk5s2

diskutil unmountDisk /dev/disk5

pv ~/Downloads/ubuntu-22.04.2-live-server-amd64.iso | sudo dd of=/dev/rdisk5 bs=1048576
```

https://github.com/huginn/huginn/blob/master/doc/docker/install.md
```
docker run -it -p 3000:3000 ghcr.io/huginn/huginn
ssh -fNL 3000:127.0.0.1:3000 <NUC>
```

https://docs.n8n.io/hosting/installation/docker/#prerequisites
https://github.com/n8n-io/n8n/issues/6793#issuecomment-1658682534
```
docker run -it --rm --name n8n -p 5678:5678 -v ~/.n8n:/home/node/.n8n docker.n8n.io/n8nio/n8n
sudo chown -R <USER>:root /home/<USER>/.n8n
ssh -fNL 5678:127.0.0.1:5678 <NUC>

docker run -d --restart=unless-stopped -it --name n8n -p 5678:5678 -v ~/.n8n:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

https://www.home-assistant.io/installation/linux#install-home-assistant-container
```
docker run -d \
  --name homeassistant \
  --privileged \
  --restart=unless-stopped \
  -e TZ=America/New_York \
  -v ~/.homeassistant:/config \
  --network=host \
  ghcr.io/home-assistant/home-assistant:stable

ssh -fNL 8123:127.0.0.1:8123 <NUC>

# configuration.yaml
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 127.0.0.1
```

https://www.reddit.com/r/yubikey/comments/u7ecpl/yubikey_remote_sudo_authentication/
https://serverfault.com/questions/847783/using-yubikey-for-sudo-over-ssh-session
```
/var/yubikey/yubico_otp:
enterusernamehere:first12charofyubikey:first12charofnextyubikey

/etc/pam.d/system-auth:
auth required pam_yubico.so id=16 authfile=/var/yubikey/yubico_otp

sudoedit /etc/pam.d/sudo

sudo apt install libpam-yubico
```

https://github.com/searxng/searxng-docker
```
cd
git clone https://github.com/searxng/searxng-docker.git
cd searxng-docker

docker-compose up

ssh -fNL 8080:127.0.0.1:8080 <NUC>

letsencrypt not working
```

caddy

```
go install github.com/caddyserver/xcaddy/cmd/xcaddy@latest
xcaddy build --with github.com/caddy-dns/cloudflare
sudo mv caddy /usr/bin

sudo groupadd --system caddy
sudo useradd --system \
    --gid caddy \
    --create-home \
    --home-dir /var/lib/caddy \
    --shell /usr/sbin/nologin \
    --comment "Caddy web server" \
    caddy

sudo mkdir /etc/caddy
sudo chown -R root:caddy /etc/caddy
sudo mkdir /var/www
sudo chown caddy:caddy /var/www

# sudoedit /etc/systemd/system/caddy.service
{{{
[Unit]
Description=Caddy
Documentation=https://caddyserver.com/docs/
After=network.target network-online.target
Requires=network-online.target

[Service]
Type=notify
User=caddy
Group=caddy
ExecStart=/usr/bin/caddy run --environ --config /etc/caddy/Caddyfile
ExecReload=/usr/bin/caddy reload --config /etc/caddy/Caddyfile
TimeoutStopSec=5s
LimitNOFILE=1048576
LimitNPROC=512
PrivateTmp=true
ProtectSystem=full
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
}}}

sudo systemctl edit caddy
{{{
[Service]
Environment="CF_API_TOKEN=your-token-here"
}}}}
sudo chmod 600 /etc/systemd/system/caddy.service.d/override.conf
sudo chown root:root /etc/systemd/system/caddy.service.d/override.conf

sudo chmod 644 /etc/systemd/system/caddy.service
sudo systemctl daemon-reload
sudo systemctl enable caddy
sudo systemctl start caddy

```

steps for caddy:
- add cname to openwrt /etc/config/dhcp + restart
- add entry in /etc/caddy/Caddyfile
- to troubleshoot, stop caddy, clear TXT from cloudflare, start caddy, check logs
- helpful commands:
  ```
  sudoedit /etc/caddy/Caddyfile
  sudo caddy fmt --overwrite -c /etc/caddy/Caddyfile
  caddy reload -c /etc/caddy/Caddyfile
  sudo journalctl -u caddy -f
  ```
