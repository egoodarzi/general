local_lan(192.168.44.0/24) -> mikrotik (vpn_clinent: 10.8.0.2)-> ubuntu (vpn server: 10.8.0.1) ->internet

install libraries.

	sudo apt install strongswan xl2tpd ppp -y

create/modify files:

	sudo nano /etc/ipsec.conf

```
config setup
    uniqueids=no

conn L2TP-IPSEC
	keyexchange=ikev1
	authby=secret
	type=transport

	ike=aes128-sha1-modp1024
	esp=aes128-sha1

	left=%any
	leftprotoport=17/1701

	right=%any
	rightprotoport=17/%any

	auto=add
```

create file for secrets:

	sudo nano /etc/ipsec.secrets
```
%any %any : PSK "secret@ehsan00A"
```

modify config file:

	sudo nano /etc/xl2tpd/xl2tpd.conf

```
[global]
port = 1701

[lns default]
ip range = 192.168.23.2-192.168.23.30
local ip = 192.168.23.1  
require chap = yes
refuse pap = yes
require authentication = yes
name = l2tpd
pppoptfile = /etc/ppp/options.xl2tpd
length bit = yes
```

option file:

	sudo nano /etc/ppp/options.xl2tpd
  
  ```
require-mschap-v2
ms-dns 8.8.8.8
ms-dns 1.1.1.1
asyncmap 0
auth
crtscts
lock
hide-password
modem
proxyarp
mtu 1400
mru 1400
lcp-echo-interval 30
lcp-echo-failure 4
```

store username passwords here:

	sudo nano /etc/ppp/chap-secrets

```
vpntoger11 l2tpd gertovpn#EFTSGB$E$    *
```

enable ip forward:

	sudo nano /etc/sysctl.conf

```
net.ipv4.ip_forward=1
```	

make it permanent:

  	sudo sysctl -p

firewall settings:

	sudo iptables -A INPUT -p udp --dport 500 -j ACCEPT
	sudo iptables -A INPUT -p udp --dport 4500 -j ACCEPT
	sudo iptables -A INPUT -p udp --dport 1701 -j ACCEPT

enable NAT for lan ip range, i route clinet range instead of natting them, clinet range is behind mikrotik:
local network behind mikrotik is 192.168.44.0/24

	sudo iptables -t nat -A POSTROUTING -s 192.168.44.0/24 -o eth0 -j MASQUERADE

restart service:

	sudo systemctl restart strongswan-starter.service
	sudo systemctl restart xl2tpd

to enable at boot:

	sudo systemctl enable strongswan-starter.service
	sudo systemctl enable xl2tpd


# sudo systemctl enable xl2tpd
xl2tpd.service is not a native service, redirecting to systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable xl2tpd


troubleshoot:
	
	sudo ipsec status
	journalctl -u strongswan-starter.service -f
	journalctl -u xl2tpd -f
	


set authentication to no:

  	sudo nano /etc/xl2tpd/xl2tpd.conf
```
require authentication = no
```

comment auth in ppp, it might overwrite l2tp config.

  	sudo nano /etc/ppp/options
	
 ```
 # auth
```


running on mikrotik hap lite, if ipsec is enabled, throuput is low around 10-15 mbps, because of high CPU usage, if we disable ipsec, vpn throuput increases

# disable ipsec:


	sudo systemctl stop strongswan-starter.service 
	sudo systemctl disable strongswan-starter.service 

require quthentication

	sudo nano /etc/xl2tpd/xl2tpd.conf
	
```

require authentication = yes                    ; * Require peer to authenticate

```


  	sudo nano  /etc/ppp/options.xl2tpd 
	
```
require-mschap-v2
refuse-pap
refuse-chap
refuse-mschap
#noccp

ms-dns 8.8.8.8
ms-dns 1.1.1.1

# asyncmap 0
# auth
noauth

# crtscts
# lock
hide-password
# modem
proxyarp
hide-password

mtu 1400
mru 1400

debug

lcp-echo-interval 30
lcp-echo-failure 4
```


# make reverse route permanenet:

1. Create the PPP hook script

		sudo nano /etc/ppp/ip-up.d/99-l2tp-route


3. Paste this content

```
#!/bin/bash

# $1 = interface (ppp0, ppp1, etc.)
# $5 = remote IP (peer)

IFACE="$1"
PEER="$5"

# Only apply if the peer matches your L2TP endpoint
if [ "$PEER" = "192.168.23.2" ]; then
    ip route add 192.168.44.0/24 via 192.168.23.2 dev "$IFACE" 2>/dev/null
fi
```

3. Make it executable

		sudo chmod +x /etc/ppp/ip-up.d/99-l2tp-route

