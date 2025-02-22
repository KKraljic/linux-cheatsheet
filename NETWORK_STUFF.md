# Networking commands

## `ip`

Check IP addresses:

```sh
ip a
# or
ip addr
# or
ip a show dev DEVICE_NAME
```

Change IP addresses ephemerally:

```sh
ip a add IP_ADDR_CIDR dev DEVICE_NAME
ip l set DEVICE_NAME down && ip l set DEVICE_NAME up
# or
ip a del IP_ADDR_CIDR dev DEVICE_NAME
ip l set DEVICE_NAME down && ip l set DEVICE_NAME up
```

Show network interfaces:

```sh
ip l
# or
ip link
# or
ip l show dev DEVICE_NAME
```

Much more info on interfaces and types with `ip link`: https://developers.redhat.com/blog/2018/10/22/introduction-to-linux-interfaces-for-virtual-networking

Check your routing table:

```sh
ip route
# or
ip r
# or IPv6
ip -6 route
# or IPv6
ip -6 r
```

Add custom routes to your routing table:

```sh
# Default route
ip route add default via GATEWAY_IP dev DEVICE_NAME
# Random other CIDR as target
ip route add 10.1.1.0/30 via 10.1.1.1 dev eth0
```

TODO: Add info on `ip netns`, `ip link add` and more advanced stuff

## `ss`

You can investigate any sockets on your machine with `ss`. List all open UDP and TCP sockets with this:

```sh
ss -tulpn
```

## `netcat`

Use OpenBSD netcat for this guide:

```sh
dnf install netcat
apt install netcat-openbsd
```

Test for open TCP ports:

```sh
netcat -vz HOSTNAME PORT
```

Send a message:

```sh
netcat -v HOSTNAME PORT <message>
# or for UDP
netcat -uv HOSTNAME PORT <message>
```

Multicast can be debugged with this using the right multicast IP group:

```
224.0.0.251 5353
ff02::fb 5353
```

## BGP

TODO: Add some info on bird2
