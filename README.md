# PiVPN HTTPS Rule for Fail2Ban, blocks nearly everything except OpenVPN TCP Pakets.

The following is based on https://stefan.angrick.me/block-unauthorized-openvpn-logins-using-fail2ban.

As well many thanks to @drmalex07

## 1. Create filter confifuration

Add a filter configuration under `/etc/fail2ban/filter.d/openvpn.conf`. The contents would be something like (regular expressions may need adjustments):

```ini
[INCLUDES]
before = common.conf

[Definition] 
failregex =%(__hostname)s ovpn-server.*:.<HOST>:[0-9]{4,5} TLS Auth Error:.*
           %(__hostname)s ovpn-server.*:.<HOST>:[0-9]{4,5} VERIFY ERROR:.*
           %(__hostname)s ovpn-server.*:.<HOST>:[0-9]{4,5} TLS Error: TLS handshake failed.*
           %(__hostname)s ovpn-server.*: TLS Error: cannot locate HMAC in incoming packet from \[AF_INET\]<HOST>:[0-9]{4,5}
           %(__hostname)s ovpn-server.*:.<HOST>:[0-9]{4,5} WARNING:.*this condition could also indicate a possible active attack on the TCP link
```

The Rule ```%(__hostname)s ovpn-server.*:.<HOST>:[0-9]{4,5} WARNING:.this condition could also indicate a possible active attack on the TCP link``` blocks everything that's going trough  except OpenVPN TCP Pakets

Test regular expressions against your logfiles using `fail2ban-regex`:

    fail2ban-regex -v /var/log/syslog /etc/fail2ban/filter.d/openvpn.conf

## 2. Create jail configuration

Add a jail configuration under `/etc/fail2ban/jail.d/openvpn.conf`:

```ini
[openvpn] 
enabled = true
port = 443
protocol = https
filter = openvpn
logpath = /var/log/openvpn.log
maxretry = 2
ignoreip = YOUR_GATEWAY/ROUTER YOUR_OPENVPN_SUBNET_WITH_CDR
bantime = -1 
```

Example for ignoreip:

ignoreip = 192.168.2.1 10.8.0.0/24

## 3. Restart fail2ban

Restart service:

    systemctl restart fail2ban.service
    
Watch your iptables for jailed hosts under `f2b-openvpn` chain (`-v` will also list number of packets involved in each rule):

    iptables -L -n -v
