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

TODO:
- DDNS
- VLAN
