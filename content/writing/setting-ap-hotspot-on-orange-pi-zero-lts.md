---
title: "Setting Ap Hotspot on Orange Pi Zero Lts"
date: 2024-06-08T03:48:29+07:00
draft: true
---

apt-get -y install iw hostapd isc-dhcp-server

### create /etc/hostapd/hostapd.conf

```conf
interface=wlan0
driver=nl80211
ssid=Opi0 AP
hw_mode=g
channel=1
macaddr_acl=0
auth_algs=1
wpa=2
wpa_passphrase=orangepi
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

### modify /etc/default/hostapd

```sh
sed -i /etc/default/hostapd -e '/DAEMON_CONF=/c DAEMON_CONF="/etc/hostapd/hostapd.conf"'
```

## Set up DHCP server for IP address management
### set the INTERFACES on /etc/default/isc-dhcp-server

```bash
# Path to dhcpd's config file (default: /etc/dhcp/dhcpd.conf).
DHCPDv4_CONF=/etc/dhcp/dhcpd.conf

# On what interfaces should the DHCP server (dhcpd) serve DHCP requests?
INTERFACESv4="wlan0"
```

### modify /etc/dhcp/dhcpd.conf
```conf
subnet 10.10.0.0 netmask 255.255.255.0 {
        range 10.10.0.2 10.10.0.16;
        option domain-name-servers 8.8.4.4, 192.168.1.1;
        option routers 10.10.0.1;
        interface wlan0;
}
```
 ### enable IP Routing/forwarding
``` bash
iptables -t nat -F
iptables -F
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
echo '1' > /proc/sys/net/ipv4/ip_forward
```
# save your current iptables config
```
apt install iptables-persistent
```

### start services dhcpd and hostapd
```bash
service hostapd start
service isc-dhcp-server start
```

