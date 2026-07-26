
# R-CORE-2 Router Configuration

## Overview

The **R-CORE-2** router operates as a core routing device within the enterprise network. It participates in the OSPF backbone (Area 0), providing redundant routing paths and maintaining connectivity between internal routers and the edge router. Similar to **R-CORE-1**, this router maintains a dedicated Out-of-Band (OOB) management interface through VLAN 99 and does not perform Network Address Translation (NAT) or Internet gateway functions.

---
# Device Information

| Parameter | Value |
|------------|----------------|
| Hostname | R-CORE-2 |
| Cisco IOS Version | 15.1 |
| Routing Protocol | OSPF |
| OSPF Process ID | 1 |
| OSPF Area | Area 0 |
| Router ID | 2.2.2.2 |
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
                             |   R-CORE-2     |
                             +----------------+
                            /        |        \
                           /         |         \
                          /          |          \
                Gig2/0   /      Gig4/0       Gig1/0
                        /            |            \
                 Distribution     R-CORE-1     OOB VLAN 99
```

---

# Interface Configuration

| Interface | IP Address | Description | Role |
|------------|------------|-------------|------|
| FastEthernet0/0 | None | Management | Disabled |
| GigabitEthernet1/0 | 10.255.255.34/30 | Internal Link | OSPF |
| GigabitEthernet1/0.99 | 10.99.99.3/24 | OOB Management VLAN | Management |
| GigabitEthernet2/0 | 10.255.255.30/30 | Internal Link | OSPF |
| GigabitEthernet3/0 | 10.255.255.49/30 | Link to R-EDGE | OSPF |
| GigabitEthernet4/0 | 10.255.255.42/30 | Link to R-CORE-1 | OSPF |

---

# Cisco IOS Configuration
```cisco
enable
configure terminal

! ==========================================================
! Basic Configuration
! ==========================================================

hostname R-CORE-2
no ip domain lookup
ip domain name eng.ruh.lk

service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption

ip cef
ip source-route
no ip icmp rate-limit unreachable
ip tcp synwait-time 5

! ==========================================================
! User Configuration
! ==========================================================

username admin privilege 15 secret YOUR_PASSWORD_HERE

! ==========================================================
! SSH Configuration
! ==========================================================

crypto key generate rsa modulus 2048
ip ssh version 2

line vty 0 4
 login local
 transport input ssh

line console 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous

line aux 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous

! ==========================================================
! Interface Configuration
! ==========================================================

interface FastEthernet0/0
 no ip address
 shutdown
 duplex half

!

interface GigabitEthernet1/0
 ip address 10.255.255.34 255.255.255.252
 ip ospf 1 area 0
 no shutdown

!

interface GigabitEthernet1/0.99
 description Out-of-Band Management
 encapsulation dot1Q 99
 ip address 10.99.99.3 255.255.255.0
 no shutdown

!

interface GigabitEthernet2/0
 ip address 10.255.255.30 255.255.255.252
 ip ospf 1 area 0
 no shutdown

!

interface GigabitEthernet3/0
 ip address 10.255.255.49 255.255.255.252
 ip ospf 1 area 0
 no shutdown

!

interface GigabitEthernet4/0
 description Inter-Core Link to R-CORE-1
 ip address 10.255.255.42 255.255.255.252
 ip ospf 1 area 0
 no shutdown

! ==========================================================
! OSPF Configuration
! ==========================================================

router ospf 1
 router-id 2.2.2.2

! ==========================================================
! Disable Unused Services
! ==========================================================

no ip http server
no ip http secure-server

! ==========================================================
! Save Configuration
! ==========================================================

end
write memory

```