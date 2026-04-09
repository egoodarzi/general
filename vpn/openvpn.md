when it is setup, ubuntu is openvpn server,  i want to add a route when a specific client connects:

modify server.conf file and add this:

    client-connect /etc/openvpn/scripts/client-connect.sh

create a new file:

    vim /etc/openvpn/scripts/client-connect.sh


```
#!/bin/bash

if [ "$common_name" = "username_of_vpn_account" ]; then
    ip route add 192.168.23.0/24 via $ifconfig_pool_remote_ip dev tun0
    # ip route add 192.168.23.0/24 via 10.8.0.2 dev tun0
fi
```

make it executable

    chmod +x /etc/openvpn/scripts/client-connect.sh

restart openvpn server:

    systemctl restart openvpn@server.service 

same thing you can do for disconnect:

    client-disconnect /etc/openvpn/scripts/client-disconnect.sh
