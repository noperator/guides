Might need to manually set IP address.


```
sudo ifconfig en7 inet 192.168.1.2 netmask 255.255.255.0
sudo route add default 192.168.1.1
ssh root@192.168.1.1  # no password after `firstboot`
```

Set root password.

```
PASSWORD=$(cat /dev/urandom | sed -E 's/[^0-9a-zA-Z]//g' | head -c 32 | tr -d '\n'; echo)
echo "$PASSWORD"
for N in $(seq 1 2); do echo "$PASSWORD"; done | passwd
PASSWORD=''
```

Generate SSH key, transfer, confirm, and lock down SSH.

```
ssh-keygen -t rsa -b 4096 -o -a 100 -q -N '' -f ~/.ssh/id_rsa
ssh root@<OPENWRT> 'tee -a /etc/dropbear/authorized_keys' < ~/.ssh/id_rsa.pub
ssh -i ~/.ssh/id_rsa root@<OPENWRT> 'uname -a'
uci set dropbear.@dropbear[0].PasswordAuth='off'
uci set dropbear.@dropbear[0].RootPasswordAuth='off'
uci commit
/etc/init.d/dropbear reload
```

Enable and configure Wi-Fi.

```
uci set wireless.radio0.disabled='0'
uci set wireless.@wifi-iface[0].ssid='<SSID>'
uci set wireless.@wifi-iface[0].encryption='psk2'
uci set wireless.@wifi-iface[0].key='<KEY>'
uci commit
/etc/init.d/network reload
```

Set static IP address.

```
uci set network.lan.ipaddr='<IP>'
uci set network.lan.gateway='<GATEWAY>'
uci set network.lan.dns='<DNS>'
uci commit
/etc/init.d/network reload
```

…or DHCP (e.g., WAP), if you want DNS/hostnames to work.

```
uci set network.lan.proto='dhcp'
uci commit
/etc/init.d/network reload
```

If needed (e.g., WAP), disable DNS/DHCP and web server.

```
for SVC in dnsmasq uhttpd; do
    for CMD in disable stop; do
        /etc/init.d/$SVC $CMD
    done
done
```

Set hostname.

```
uci set system.@system[0].hostname='<HOSTNAME>'
uci commit
/etc/init.d/system reload
/etc/init.d/dnsmasq restart  # Optionally, flush DNS cache.
```

Set local domain. [RFC 8375](https://tools.ietf.org/html/rfc8375) specifies `.home.arpa` for home-scoped networks. Good discussion [here](https://unix.stackexchange.com/a/92517).

```
uci set dhcp.@dnsmasq[0].local='/<LOCAL_DOMAIN>/'
uci set dhcp.@dnsmasq[0].domain='<LOCAL_DOMAIN>'
uci commit
/etc/init.d/dnsmasq reload
```

_Attempt_ to make dnsmasq (handles DHCPv4 and DNS) IPv6-aware by interfacing it with odhcpd (handles DHCPv6). Only seems to work for Ethernet-connected devices, strangely. Potentially useful guides [here](https://superuser.com/a/1248857) and [here](https://openwrt.org/docs/guide-user/network/ipv6/ipv6.dns).

```
uci add_list dhcp.@dnsmasq[0].addnhost=$(uci get dhcp.odhcpd.leasefile)  # /tmp/hosts/odhcpd
uci commit
/etc/init.d/dnsmasq reload
```

Install Simple Adblock using [this guide](https://github.com/openwrt/packages/blob/master/net/simple-adblock/files/README.md).

```
# Install with dependencies. These took a _long_ time to download.
opkg update
opkg install \
    ca-certificates wget libopenssl \
    curl \
    ip6tables-mod-nat kmod-ipt-nat6 \
    simple-adblock
opkg --force-overwrite install coreutils-sort

# Enable and start service.
uci set simple-adblock.config.enabled=1
uci commit simple-adblock
/etc/init.d/simple-adblock start
```

Measure network bandwidth.

```
opkg update
opkg install iperf3
iperf3 -s

iperf3 -c <ROUTER>
Connecting to host <ROUTER> port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec  42.8 MBytes   359 Mbits/sec    0    345 KBytes
...
[  5]   9.00-10.00  sec  41.7 MBytes   350 Mbits/sec    0    345 KBytes
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec   408 MBytes   342 Mbits/sec    0             sender
[  5]   0.00-10.00  sec   406 MBytes   340 Mbits/sec                  receiver
```

Update Adblock.

```
opkg update
opkg list-upgradable
```

Failover with dual ISPs.

```
opkg update
opkg install mwan3
opkg install iptables-nft
opkg install ip6tables-nft

# helpful commands
swconfig dev switch0 show | grep -E 'Port|link' | grep -B 1 link:up | grep -v '\--'
/etc/init.d/network reload && ifdown wan && ifup wan && sleep 5 && ifconfig eth0.1 && logread | grep -i wan | grep -v mwan | tail
ifconfig -a | grep -E '^[^ ]|inet6? ' | grep -vE ': *(fe80|::|127\.|fd)'
for IFACE in eth0.2 eth0.3; do
    echo "iface: $IFACE"
    ping -c 1 -W 2 -I $IFACE 9.9.9.9
    ping6 -c 1 -W 2 -I $IFACE 2620:fe::9
done 2>&1 | grep -E 'iface: |bytes from|unreachable'
mwan3 use wan   ping  -c1 -W3 9.9.9.9    | grep 'bytes from'
mwan3 use wan6  ping6 -c1 -W3 2620:fe::9 | grep 'bytes from'
mwan3 use wanb  ping  -c1 -W3 9.9.9.9    | grep 'bytes from'
mwan3 use wanb6 ping6 -c1 -W3 2620:fe::9 | grep 'bytes from'

cd /etc/config
{
    echo "== /etc/config/network =="
    cat network
    echo "== /etc/config/mwan3 =="
    cat mwan3
    echo "== /etc/config/firewall =="
    cat firewall
    echo "== iptables-save =="
    iptables-save
    echo "== ip6tables-save =="
    ip6tables-save
    echo "== ifconfig -a =="
    ifconfig -a | grep -E '^[^ ]|inet6? '
    echo "== mwan3 status =="
    mwan3 status 2>&1 | grep -vE 'Warning: ip6?tables'
    echo "== ip route =="
    ip route
    echo "== ip -6 route =="
    ip -6 route
} | tee /tmp/cfg.txt

# list these policies
uci show mwan3 | grep 'policy$' | sed -E 's/^mwan3\.//; s/=policy//'
wan_only
wanb_only
balanced
wan_wanb
wanb_wan

# set these policies (and validate taht the incoming policy is in the list above)
uci show mwan3 | grep use_policy
mwan3.https.use_policy='wan_wanb'
mwan3.default_rule_v4.use_policy='wanb_wan'
```

[Tailscale](https://openwrt.org/docs/guide-user/services/vpn/tailscale/start).

```
opkg update
opkg install tailscale
tailscale up
tailscale status
```

adblock-fast
https://docs.openwrt.melmac.net/adblock-fast/#how-to-install
https://docs.openwrt.melmac.net/#on-your-router

```
opkg update
opkg install wget-ssl
echo -e -n 'untrusted comment: OpenWrt usign key of Stan Grishin\nRWR//HUXxMwMVnx7fESOKO7x8XoW4/dRidJPjt91hAAU2L59mYvHy0Fa\n' > /etc/opkg/keys/7ffc7517c4cc0c56
sed -i '/stangri_repo/d' /etc/opkg/customfeeds.conf
echo 'src/gz stangri_repo https://repo.openwrt.melmac.net' >> /etc/opkg/customfeeds.conf
opkg update

opkg find adblock-fast
adblock-fast - 1.1.2-2 - Fast AdBlocking script to block ad or abuse/malware domains with Dnsmasq, SmartDNS or Unbound.
 Script supports local/remote list of domains and hosts-files for both block-listing and allow-listing.
 Please see https://docs.openwrt.melmac.net/adblock-fast/ for more information.

opkg install adblock-fast

uci set adblock-fast.config.enabled='1'; uci commit adblock-fast;
/etc/init.d/adblock-fast enable
/etc/init.d/adblock-fast restart

/etc/init.d/adblock-fast allow awstrack.me
/etc/init.d/adblock-fast allow analytics.google.com
```

openssh for yubikey

```
openssh-server - 9.8p1-1 - OpenSSH server.
```


TODO:
- [ ] DDNS
- [ ] VLAN
- [ ] for vlan/mwan3 stuff:
  - try specifying eth0, eth1?
  - try using luci/web ui to generate text config?
