# SW-CORE-2 Layer 3 Switch Configuration

## Overview

**SW-CORE-2** serves as the secondary Layer 3 core switch in the enterprise network. It provides inter-VLAN routing, dynamic routing through OSPF, gateway redundancy using HSRP, and DHCP relay services. Together with **SW-CORE-1**, it forms a redundant core layer that ensures high availability and resilient routing. The switch also maintains a dedicated Out-of-Band (OOB) management interface using VLAN 99.

Unlike SW-CORE-1, which is configured as the preferred HSRP active gateway through a higher priority, **SW-CORE-2 operates as the standby HSRP router** and automatically assumes the gateway role if SW-CORE-1 becomes unavailable.

---

# Device Information

| Parameter | Value |
|------------|----------------|
| Hostname | SW-CORE-2 |
| Cisco IOS Version | 15.2 |
| Device Type | Layer 3 Switch |
| Routing Protocol | OSPF |
| OSPF Area | Area 0 |
| Router ID | 12.12.12.12 |
| First Hop Redundancy | HSRP (Standby) |
| DHCP Relay | Enabled |
| Management | SSH / HTTP / HTTPS |

---

# Network Topology

```
                    Distribution Switches
                             │
                             │
                        SW-A-DIS
                             │
                     Trunk (VLAN 40,99,100)
                             │
                     +------------------+
                     |    SW-CORE-2     |
                     +------------------+
                    /    |      |      \
                   /     |      |       \
              R-CORE-2  R-EDGE  SW-CORE-1
```

---

# Routed Interfaces

| Interface | IP Address | Purpose |
|-----------|------------|---------|
| GigabitEthernet0/0 | 10.255.255.6/30 | Routed Link |
| GigabitEthernet0/1 | 10.255.255.14/30 | Routed Link |
| GigabitEthernet0/2 | 10.255.255.22/30 | Routed Link |
| GigabitEthernet1/0 | 10.255.255.37/30 | Routed Link |

All routed interfaces participate in **OSPF Area 0**.

---

Cisco IOS Configuration (SW-CORE-2)
```cisco
enable
configure terminal

! ==========================================================
! Basic Configuration
! ==========================================================

hostname SW-CORE-2

service timestamps debug datetime msec
service timestamps log datetime msec
service compress-config
no service password-encryption

ip cef

! ==========================================================
! User Configuration
! ==========================================================

username admin privilege 15 secret YOUR_PASSWORD_HERE

! ==========================================================
! Spanning Tree Configuration
! ==========================================================

spanning-tree mode pvst
spanning-tree extend system-id

! ==========================================================
! Routed Interfaces
! ==========================================================

interface GigabitEthernet0/0
 no switchport
 ip address 10.255.255.6 255.255.255.252
 ip ospf 1 area 0
 no shutdown

!

interface GigabitEthernet0/1
 no switchport
 ip address 10.255.255.14 255.255.255.252
 ip ospf 1 area 0
 no shutdown

!

interface GigabitEthernet0/2
 no switchport
 ip address 10.255.255.22 255.255.255.252
 ip ospf 1 area 0
 no shutdown

!

interface GigabitEthernet1/0
 no switchport
 ip address 10.255.255.37 255.255.255.252
 ip ospf 1 area 0
 no shutdown

! ==========================================================
! Trunk Interfaces
! ==========================================================

interface GigabitEthernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 100
 switchport trunk allowed vlan 99,100
 no shutdown

!

interface GigabitEthernet1/1
 description Trunk Downlink to SW-A-DIS
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 100
 switchport trunk allowed vlan 40,99,100
 no shutdown

!

interface GigabitEthernet1/2
 description Inter-Core L2 Link for HSRP Keepalives
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 100
 switchport trunk allowed vlan 40,99,100
 no shutdown

! ==========================================================
! VLAN Interfaces (SVIs)
! ==========================================================

interface Vlan40
 ip address 10.10.40.253 255.255.255.0
 ip helper-address 10.10.40.10
 standby 40 ip 10.10.40.254
 standby 40 preempt
 ip ospf 1 area 0
 no shutdown

!

interface Vlan99
 description Out-of-Band Management
 ip address 10.99.99.12 255.255.255.0
 ip ospf 1 area 0
 no shutdown

!

interface Vlan100
 ip address 10.255.255.33 255.255.255.252
 ip ospf 1 area 0
 no shutdown

! ==========================================================
! OSPF Configuration
! ==========================================================

router ospf 1
 router-id 12.12.12.12

! ==========================================================
! HTTP / HTTPS Management
! ==========================================================

ip http server
ip http secure-server

! ==========================================================
! SSH Configuration
! ==========================================================

ip ssh server algorithm encryption aes128-ctr aes192-ctr aes256-ctr
ip ssh client algorithm encryption aes128-ctr aes192-ctr aes256-ctr

line vty 0 4
 login local
 transport input ssh

! ==========================================================
! Save Configuration
! ==========================================================

end
write memory

```