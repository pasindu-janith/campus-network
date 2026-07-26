# SW-CORE-1 Layer 3 Switch Configuration

## Overview

**SW-CORE-1** is the primary Layer 3 core switch within the enterprise network. It provides inter-VLAN routing, dynamic routing using OSPF, gateway redundancy using HSRP, DHCP relay services, and Out-of-Band (OOB) management connectivity. The switch also serves as the aggregation point between distribution switches, core routers, and the edge router.

---

# Device Information

| Parameter | Value |
|------------|----------------|
| Hostname | SW-CORE-1 |
| Cisco IOS Version | 15.2 |
| Device Type | Layer 3 Switch |
| Routing Protocol | OSPF |
| OSPF Area | Area 0 |
| Router ID | 11.11.11.11 |
| First Hop Redundancy | HSRP |
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
+----------------------+
| SW-CORE-1 |
+----------------------+
│ │ │
│ │ └────────── R-EDGE
│ │
│ └──────────── R-CORE-1
│
└────────────── SW-CORE-2

```

# Cisco IOS Configuration
```cisco
enable
configure terminal

! ==========================================================
! Basic Configuration
! ==========================================================

hostname SW-CORE-1

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
 ip address 10.255.255.2 255.255.255.252
 ip ospf 1 area 0
 no shutdown

!

interface GigabitEthernet0/1
 no switchport
 ip address 10.255.255.10 255.255.255.252
 ip ospf 1 area 0
 no shutdown

!

interface GigabitEthernet0/2
 no switchport
 ip address 10.255.255.18 255.255.255.252
 ip ospf 1 area 0
 no shutdown

!

interface GigabitEthernet1/0
 no switchport
 ip address 10.255.255.29 255.255.255.252
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

!

interface GigabitEthernet2/0
 description OOB Link up to R-EDGE
 switchport mode access
 switchport access vlan 99
 no shutdown

! ==========================================================
! VLAN Interfaces
! ==========================================================

interface Vlan40
 ip address 10.10.40.252 255.255.255.0
 ip helper-address 10.10.40.10
 standby 40 ip 10.10.40.254
 standby 40 priority 110
 standby 40 preempt
 ip ospf 1 area 0
 no shutdown

!

interface Vlan99
 description Out-of-Band Management
 ip address 10.99.99.11 255.255.255.0
 ip ospf 1 area 0
 no shutdown

!

interface Vlan100
 ip address 10.255.255.25 255.255.255.252
 ip ospf 1 area 0
 no shutdown

! ==========================================================
! OSPF Configuration
! ==========================================================

router ospf 1
 router-id 11.11.11.11

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