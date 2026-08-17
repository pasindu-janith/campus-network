# R-CORE-1 Router Configuration

## Overview

The **R-CORE-1** router functions as one of the core routers within the enterprise network. It is responsible for interconnecting the distribution and edge routers using the Open Shortest Path First (OSPF) routing protocol. The router maintains routing information for the internal network and provides a dedicated Out-of-Band (OOB) management interface through a VLAN subinterface.

---

# Device Information

| Parameter | Value |
|------------|----------------|
| Hostname | R-CORE-1 |
| Cisco IOS Version | 15.1 |
| Routing Protocol | OSPF |
| OSPF Process ID | 1 |
| OSPF Area | Area 0 |
| Router ID | 1.1.1.1 |
| NAT | Disabled |
| Internet Gateway | No |
| Remote Management | SSH Version 2 |

---


# Network Topology

```
                             +----------------+
                             |    R-EDGE      |
                             +----------------+
                                     |
                            GigabitEthernet3/0
                                     |
                             +----------------+
                             |   R-CORE-1     |
                             +----------------+
                            /        |        \
                           /         |         \
                          /          |          \
                Gig2/0   /      Gig4/0       Gig1/0
                        /            |            \
                 Distribution     R-CORE-2     OOB VLAN 99
```

---

# Interface Configuration

| Interface | IP Address | Description | Role |
|------------|------------|-------------|------|
| GigabitEthernet1/0 | 10.255.255.26/30 | Internal Link | OSPF |
| GigabitEthernet1/0.99 | 10.99.99.2/24 | OOB Management VLAN | Management |
| GigabitEthernet2/0 | 10.255.255.38/30 | Internal Link | OSPF |
| GigabitEthernet3/0 | 10.255.255.45/30 | Link to R-EDGE | OSPF |
| GigabitEthernet4/0 | 10.255.255.41/30 | Link to R-CORE-2 | OSPF |

---

# Cisco IOS Configuration

```cisco
enable
configure terminal

hostname R-CORE-1

no aaa new-model

ip source-route
no ip icmp rate-limit unreachable
ip cef
no ip domain lookup
ip domain name eng.ruh.lk
no ipv6 cef

username admin privilege 15 secret 5 $1$15m6$jgiHywdvy3n/sOz900VDO/

ip tcp synwait-time 5
ip ssh version 2

interface FastEthernet0/0
 description OOB management
 no ip address
 shutdown
 duplex half
exit

interface GigabitEthernet1/0
 ip address 10.255.255.26 255.255.255.252
 ip ospf 1 area 0
 negotiation auto
exit

interface GigabitEthernet1/0.99
 description Out-of-Band Management
 encapsulation dot1Q 99
 ip address 10.99.99.2 255.255.255.0
exit

interface GigabitEthernet2/0
 ip address 10.255.255.38 255.255.255.252
 ip ospf 1 area 0
 negotiation auto
exit

interface GigabitEthernet3/0
 ip address 10.255.255.45 255.255.255.252
 ip ospf 1 area 0
 negotiation auto
exit

interface GigabitEthernet4/0
 description Inter-Core Link to R-CORE-2
 ip address 10.255.255.41 255.255.255.252
 ip ospf 1 area 0
 negotiation auto
exit

router ospf 1
 router-id 1.1.1.1
exit

ip forward-protocol nd
no ip http server
no ip http secure-server

ip access-list extended ACL_MGMT_ACCESS
 remark PERMIT SSH from MGMT VLAN 99 only
 permit tcp 10.99.10.0 0.0.0.255 any eq 22
 permit tcp 10.99.20.0 0.0.0.255 any eq 22
 permit tcp 10.99.30.0 0.0.0.255 any eq 22
 permit tcp 10.99.99.0 0.0.0.255 any eq 22
 remark Permit SNMP from VM-ZABBIX only
 permit udp host 10.99.99.7 any eq snmp
 remark Explicit DENY ALL Other MGMT Traffic
 deny ip any any log
exit

snmp-server community FoE-Network RO

line con 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
 stopbits 1
exit

line aux 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
 stopbits 1
exit

line vty 0 4
 access-class ACL_MGMT_ACCESS in
 login local
 transport input ssh
exit

end
write memory
```