---
title: network forwarding
date: 2015-11-21
---

Forward external network interface (usb0) to internal (eth0):

    echo 1 > /proc/sys/net/ipv4/ip_forward
    iptables -t nat -A POSTROUTING -o usb0 -j MASQUERADE
    iptables -A FORWARD -i usb0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
    iptables -A FORWARD -i eth0 -o usb0 -j ACCEPT

http://askubuntu.com/questions/95199/two-network-cards-and-ip-forwarding
