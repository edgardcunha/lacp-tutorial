# Tutorial: Testing Link Aggregation with Mininet
Testing Link Aggregation Control Protocol with Mininet

## SETUP
VM: Ubuntu 21.04, 1024 MB de RAM, 1 CPU, Python 3.9.5

## Implementation

### Setting Link Aggregation in Host h1

```zsh
py h1.cmd("ifconfig")
```

```zsh
h1-eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.0.1  netmask 255.0.0.0  broadcast 10.255.255.255
        inet6 fe80::34c5:aff:fe44:992f  prefixlen 64  scopeid 0x20<link>
        ether 36:c5:0a:44:99:2f  txqueuelen 1000  (Ethernet)
        RX packets 17  bytes 1286 (1.2 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 17  bytes 1286 (1.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

h1-eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::4031:2bff:fe83:4c7c  prefixlen 64  scopeid 0x20<link>
        ether 42:31:2b:83:4c:7c  txqueuelen 1000  (Ethernet)
        RX packets 17  bytes 1286 (1.2 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 17  bytes 1286 (1.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

```zsh
py h1.cmd("modprobe bonding")
```

```zsh
py h1.cmd("ip link add bond0 type bond")
py h1.cmd("ip link set bond0 address 02:01:02:03:04:08")
```

```zsh
py h1.cmd("ip link set h1-eth0 down")
py h1.cmd("ip link set h1-eth0 address 00:00:00:00:00:11")
py h1.cmd("ip link set h1-eth0 master bond0")
py h1.cmd("ip link set h1-eth1 down")
py h1.cmd("ip link set h1-eth1 address 00:00:00:00:00:12")
py h1.cmd("ip link set h1-eth1 master bond0")
```

```zsh
py h1.cmd("ip addr add 10.0.0.1/8 dev bond0")
py h1.cmd("ip addr del 10.0.0.1/8 dev h1-eth0")
```

```zsh
py h1.cmd("ip link set bond0 up")
```

```zsh
py h1.cmd("ifconfig")
```

```zsh
py h1.cmd("cat /proc/net/bonding/bond0")
```

```zsh
Ethernet Channel Bonding Driver: v5.11.0-31-generic

Bonding Mode: IEEE 802.3ad Dynamic link aggregation
Transmit Hash Policy: layer2 (0)
MII Status: up
MII Polling Interval (ms): 100
Up Delay (ms): 0
Down Delay (ms): 0
Peer Notification Delay (ms): 0

802.3ad info
LACP rate: slow
Min links: 0
Aggregator selection policy (ad_select): stable
System priority: 65535
System MAC address: 02:01:02:03:04:08
Active Aggregator Info:
	Aggregator ID: 1
	Number of ports: 1
	Actor Key: 15
	Partner Key: 1
	Partner Mac Address: 00:00:00:00:00:00

Slave Interface: h1-eth0
MII Status: up
Speed: 10000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 00:00:00:00:00:11
Slave queue ID: 0
Aggregator ID: 1
Actor Churn State: monitoring
Partner Churn State: monitoring
Actor Churned Count: 0
Partner Churned Count: 0
details actor lacp pdu:
    system priority: 65535
    system mac address: 02:01:02:03:04:08
    port key: 15
    port priority: 255
    port number: 1
    port state: 77
details partner lacp pdu:
    system priority: 65535
    system mac address: 00:00:00:00:00:00
    oper key: 1
    port priority: 255
    port number: 1
    port state: 1

Slave Interface: h1-eth1
MII Status: up
Speed: 10000 Mbps
Duplex: full
Link Failure Count: 0
Permanent HW addr: 00:00:00:00:00:12
Slave queue ID: 0
Aggregator ID: 2
Actor Churn State: monitoring
Partner Churn State: monitoring
Actor Churned Count: 0
Partner Churned Count: 0
details actor lacp pdu:
    system priority: 65535
    system mac address: 02:01:02:03:04:08
    port key: 15
    port priority: 255
    port number: 2
    port state: 69
details partner lacp pdu:
    system priority: 65535
    system mac address: 00:00:00:00:00:00
    oper key: 1
    port priority: 255
    port number: 1
    port state: 1
```


## References
[Ryu-Book - Link Aggregation](https://osrg.github.io/ryu-book/en/html/link_aggregation.html)
