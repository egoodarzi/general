rules to dst-nat some ports:

    iptables -t nat -A PREROUTING -p tcp --dport 3389 -j DNAT --to-destination 10.8.0.2:3389
    iptables -t nat -A POSTROUTING -p tcp -d 10.8.0.2 --dport 3389 -j MASQUERADE

to make it persistent, without installing a package (for offline ubuntu), we make a service to run a script at startup:

## Save your current iptables rules

    mkdir /etc/iptables

    sudo iptables-save > /etc/iptables/iptables.rules
    sudo ip6tables-save > /etc/ip6tables.rules

## Create a shell script to restore iptables

    sudo nano /usr/local/sbin/restore-iptables.sh

Paste the following:

    #!/bin/sh
    /sbin/iptables-restore < /etc/iptables/iptables.rules

Make it executable:

    sudo chmod +x /usr/local/sbin/restore-iptables.sh


## creaate systemd service

    sudo nano /etc/systemd/system/iptables-restore.service

Paste:

    [Unit]
    Description=Restore iptables rules
    After=network.target
    
    [Service]
    Type=oneshot
    ExecStart=/usr/local/sbin/restore-iptables.sh
    RemainAfterExit=yes
    
    [Install]
    WantedBy=multi-user.target

## Reload systemd and start

    sudo systemctl daemon-reload
    sudo systemctl enable iptables-restore.service
    sudo systemctl start iptables-restore.service
    sudo systemctl status iptables-restore.service

## verify rules

    sudo iptables -t nat -L -n -v
