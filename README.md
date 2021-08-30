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

## Setting OpenFlow Version

Set the OpenFlow version of switch s1 to 1.3. Input this command on xterm of switch s1.

```zsh
py s1.cmd("ovs-vsctl set Bridge s1 protocols=OpenFlow13")
```

Executing the Switching Hub
```zsh
py c0.cmd("ryu-manager lacp/ryu.app.simple_switch_lacp_13")
```

```zsh
...
[LACP][INFO] SW=0000000000000001 PORT=1 LACP received.
[LACP][INFO] SW=0000000000000001 PORT=1 the slave i/f has just been up.
[LACP][INFO] SW=0000000000000001 PORT=1 the timeout time has changed.
[LACP][INFO] SW=0000000000000001 PORT=1 LACP sent.
slave state changed port: 1 enabled: True
[LACP][INFO] SW=0000000000000001 PORT=2 LACP received.
[LACP][INFO] SW=0000000000000001 PORT=2 the slave i/f has just been up.
[LACP][INFO] SW=0000000000000001 PORT=2 the timeout time has changed.
[LACP][INFO] SW=0000000000000001 PORT=2 LACP sent.
slave state changed port: 2 enabled: True
...
```

Let’s check flow entry.
Node: s1:

```zsh
py s1.cmd("ovs-ofctl -O openflow13 dump-flows s1")
```

```zsh{.line-numbers}
cookie=0x0, duration=291.803s, table=0, n_packets=28, n_bytes=3472, idle_timeout=90, send_flow_rem priority=65535,in_port="s1-eth1",dl_src=00:00:00:00:00:11,dl_type=0x8809 actions=CONTROLLER:65509```
cookie=0x0, duration=291.791s, table=0, n_packets=28, n_bytes=3472, idle_timeout=90, send_flow_rem priority=65535,in_port="s1-eth2",dl_src=00:00:00:00:00:12,dl_type=0x8809 actions=CONTROLLER:65509
cookie=0x0, duration=303.608s, table=0, n_packets=6, n_bytes=528, priority=0 actions=CONTROLLER:65535
```

In the switch,

* The Packet-In message is sent when the LACP data unit (ethertype is 0x8809) is sent from h1's h1-eth1 (the input port is s1-eth2 and the MAC address is 00:00:00:00:00:12).
* The Packet-In message is sent when the LACP data unit (ethertype is 0x8809) is sent from h1's h1-eth0 (the input port is s1-eth1 and the MAC address is 00:00:00:00:00:11)
* The same Table-miss flow entry as that of "Switching Hub".

The above three flow entries have been registered.

### Checking the Link Aggregation Function

#### Improving Communication Speed
First of all, check improvement in the communication speed as a result of link aggregation. Let's take a look at the ways of using different links depending on communication.

First, execute ping from host h2 to host h1.

Node: h2:
```zsh
py h2.cmd("ping 10.0.0.1 -c4")
```

```zsh
...
64 bytes from 10.0.0.1: icmp_seq=1 ttl=64 time=0.525 ms
64 bytes from 10.0.0.1: icmp_seq=2 ttl=64 time=0.167 ms
64 bytes from 10.0.0.1: icmp_seq=3 ttl=64 time=0.127 ms
64 bytes from 10.0.0.1: icmp_seq=4 ttl=64 time=0.169 ms
...
```

While continuing to send pings, check the flow entry of switch s1.

Node: s1:
```zsh
py s1.cmd("ovs-ofctl -O openflow13 dump-flows s1")
```

```zsh
cookie=0x0, duration=71029.586s, table=0, n_packets=2295, n_bytes=284580, idle_timeout=90, send_flow_rem priority=65535,in_port="s1-eth1",dl_src=00:00:00:00:00:11,dl_type=0x8809 actions=CONTROLLER:65509
cookie=0x0, duration=71029.574s, table=0, n_packets=2295, n_bytes=284580, idle_timeout=90, send_flow_rem priority=65535,in_port="s1-eth2",dl_src=00:00:00:00:00:12,dl_type=0x8809 actions=CONTROLLER:65509
cookie=0x0, duration=69963.229s, table=0, n_packets=19, n_bytes=1694, priority=1,in_port="s1-eth1",dl_dst=00:00:00:00:00:22 actions=output:"s1-eth3"
cookie=0x0, duration=69963.226s, table=0, n_packets=18, n_bytes=1596, priority=1,in_port="s1-eth3",dl_dst=02:01:02:03:04:08 actions=output:"s1-eth1"
cookie=0x0, duration=71041.391s, table=0, n_packets=87, n_bytes=6142, priority=0 actions=CONTROLLER:65535
```

After the previous check point, two flow entries have been added. They are the 4th and 5th entries with a small duration value.

The respective flow entry is as follows:

When a packet address to bond0 of h1 is received from the 3rd port (s1-eth3, that is, the counterpart interface of h2), it is output from the first port (s1-eth1).
When a packet addressed to h2 is received from the 1st port (s1-eth1), it is output from the 3rd port (s1-eth3).
You can tell that s1-eth1 is used for communication between h2 and h1.

Next, execute ping from host h3 to host h1.

Node: h3:

```zsh
py h3.cmd("ping 10.0.0.1 -c4")
```

```zsh
...
64 bytes from 10.0.0.1: icmp_seq=1 ttl=64 time=27.7 ms
64 bytes from 10.0.0.1: icmp_seq=2 ttl=64 time=0.421 ms
64 bytes from 10.0.0.1: icmp_seq=3 ttl=64 time=0.132 ms
64 bytes from 10.0.0.1: icmp_seq=4 ttl=64 time=0.045 ms
...
```

While continuing to send pings, check the flow entry of switch s1.

Node: s1:
```zsh
py s1.cmd("ovs-ofctl -O openflow13 dump-flows s1")
```

```zsh
cookie=0x0, duration=71384.263s, table=0, n_packets=2306, n_bytes=285944, idle_timeout=90, send_flow_rem priority=65535,in_port="s1-eth1",dl_src=00:00:00:00:00:11,dl_type=0x8809 actions=CONTROLLER:65509
cookie=0x0, duration=71384.251s, table=0, n_packets=2306, n_bytes=285944, idle_timeout=90, send_flow_rem priority=65535,in_port="s1-eth2",dl_src=00:00:00:00:00:12,dl_type=0x8809 actions=CONTROLLER:65509
cookie=0x0, duration=70317.906s, table=0, n_packets=19, n_bytes=1694, priority=1,in_port="s1-eth1",dl_dst=00:00:00:00:00:22 actions=output:"s1-eth3"
cookie=0x0, duration=70317.903s, table=0, n_packets=18, n_bytes=1596, priority=1,in_port="s1-eth3",dl_dst=02:01:02:03:04:08 actions=output:"s1-eth1"
cookie=0x0, duration=43.380s, table=0, n_packets=5, n_bytes=434, priority=1,in_port="s1-eth2",dl_dst=00:00:00:00:00:23 actions=output:"s1-eth4"
cookie=0x0, duration=43.376s, table=0, n_packets=4, n_bytes=336, priority=1,in_port="s1-eth4",dl_dst=02:01:02:03:04:08 actions=output:"s1-eth2"
cookie=0x0, duration=71396.068s, table=0, n_packets=91, n_bytes=6366, priority=0 actions=CONTROLLER:65535
```

After the previous check point, two flow entries have been added. They are the 5th and 6th entries with a small duration value.

The respective flow entry is as follows:

When a packet addressed to h3 is received from the 2nd port (s1-eth2), it is output from the 4th port (s1-eth4).
When a packet address to bond0 of h1 is received from the 4th port (s1-eth4, that is, the counterpart interface of h3), it is output from the 2nd port (s1-eth2).
You can tell that s1-eth2 is used for communication between h3 and h1.

As a matter of course, ping can be executed from host H4 to host h1 as well. As before, new flow entries are registered and s1-eth1 is used for communication between h4 and h1.

Destination host | Port used
h2 | 1
h3 | 2
h4 | 1

As shown above, we were able to confirm use of different links depending on communication.

### Improving Fault Tolerance

Check improvement in fault tolerance as a result of link aggregation. The current state is that when h2 and h4 communicate with h1, `s1-eth2` is used and when h3 communicates with h1, s1-eth1 is used.

Here, we separate h1-eth0, which is the counterpart interface of `s1-eth1`, from the link aggregation group.

Node: h1:
```zsh
py s1.cmd("ip link set h1-eth0 nomaster")
```

## References
[Ryu-Book - Link Aggregation](https://osrg.github.io/ryu-book/en/html/link_aggregation.html)
