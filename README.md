# Tutorial: Testing Link Aggregation with Mininet
Testing Link Aggregation Control Protocol with Mininet

## SETUP
VM: Ubuntu 21.04, 1024 MB de RAM, 1 CPU, Python 3.9.5

## Implementation

### Setting Link Aggregation in Host h1
```
py h1.cmd("modprobe bonding")
```
```
py h1.cmd("ip link add bond0 type bond")
py h1.cmd("ip link set bond0 address 02:01:02:03:04:08")
```

```
py h1.cmd("ip link set h1-eth0 down")
py h1.cmd("ip link set h1-eth0 address 00:00:00:00:00:11")
py h1.cmd("ip link set h1-eth0 master bond0")
py h1.cmd("ip link set h1-eth1 down")
py h1.cmd("ip link set h1-eth1 address 00:00:00:00:00:12")
py h1.cmd("ip link set h1-eth1 master bond0")
```

```
py h1.cmd("ip addr add 10.0.0.1/8 dev bond0")
py h1.cmd("ip addr del 10.0.0.1/8 dev h1-eth0")
```

```
py h1.cmd("ip link set bond0 up")
```

```
py h1.cmd("ifconfig")
```

```
py h1.cmd("cat /proc/net/bonding/bond0")
```


## References
[Ryu-Book - Link Aggregation](https://osrg.github.io/ryu-book/en/html/link_aggregation.html)
