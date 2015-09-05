## Netfilter

        netfilter://mychain/myset

When using IPv6 containers, the NAT is gone and your container and ports are by default reachable. You can use this module to firewall those.

If no chain/set is specified, it will default to `netfilter://FORWARD_direct/containerports`

This module does on initialization:
- creates an ipset (http://ipset.netfilter.org) called <myset> (hash:ip,port)
- appends a rule to chain <mychain> that allows <ip,port> addresses in a set <myset> to be forwarded to the container.
- appends a rule to chain <mychain> that will drop packets going to the docker0 device.

Or in actual commands
```
/usr/sbin/ipset create <myset> hash:ip,port family inet6
/usr/sbin/ip6tables -A <mychain> -o docker0 -m set --match-set <myset> dst,dst -j ACCEPT
/usr/sbin/ip6tables -A <mychain> -o docker0 -j DROP
```

When an IPv6 service gets registered:
- the container <ip,port> will be added to <myset> and access to this port will be allowed.
- icmpv6 echo request will also be allowed so that you can ping the container

Or in actual commands
```
/usr/sbin/ipset add <myset> <ip,proto:port>
/usr/sbin/ipset add <myset> <ip,icmpv6:128/0>
```

When the service gets deregistered, the access will be removed.

### Firewalld support
The module will communicate with firewalld when detected.   
The default FORWARD_direct chain would be a good chain to use with firewalld

### Prerequisites
- You need the iptables (v1.4.21+) and ipset (v6.19+) packages
- ipset and ip6tables are expected to be found in /usr/sbin

