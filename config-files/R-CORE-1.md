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

! ==========================================================
! Basic Configuration
! ==========================================================

hostname R-CORE-1
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
! OSPF Configuration
! ==========================================================

router ospf 1
 router-id 1.1.1.1


! ==========================================================
! Interface Configuration
! ==========================================================

interface FastEthernet0/0
 description OOB Management
 no ip address
 shutdown
 duplex half

!

interface GigabitEthernet1/0
 ip address 10.255.255.26 255.255.255.252
 ip ospf 1 area 0
 no shutdown

!

interface GigabitEthernet1/0.99
 description Out-of-Band Management
 encapsulation dot1Q 99
 ip address 10.99.99.2 255.255.255.0
 no shutdown

!

interface GigabitEthernet2/0
 ip address 10.255.255.38 255.255.255.252
 ip ospf 1 area 0
 no shutdown

!

interface GigabitEthernet3/0
 ip address 10.255.255.45 255.255.255.252
 ip ospf 1 area 0
 no shutdown

!

interface GigabitEthernet4/0
 description Inter-Core Link to R-CORE-2
 ip address 10.255.255.41 255.255.255.252
 ip ospf 1 area 0
 no shutdown


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