Set root password.


```
for i in $(seq 1 2); do echo '<PASSWORD>'; done | passwd
```

Lock down SSH.

```
ssh root@192.168.1.1 'tee -a /etc/dropbear/authorized_keys' < ~/.ssh/id_rsa.pub
ssh -i ~/.ssh/id_rsa root@192.168.1.1 'uname -a'
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

TODO:
- DDNS
- VLAN
