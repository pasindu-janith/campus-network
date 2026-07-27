# SW-D-DEIE Layer 3 Distribution Switch Configuration

## Overview

**SW-D-DEIE** is the distribution-layer switch serving the **DEIE (Department of Electrical & Information Engineering)**. It performs inter-VLAN routing for the department's local network, participates in the OSPF domain to reach the core layer, relays DHCP requests to the central DHCP server, and provides a local Out-of-Band (OOB) management VLAN. It uplinks to both core switches for redundancy and downlinks to the access switch **SW-A-DEIE**.

---

# Device Information

| Parameter | Value |
|------------|----------------|
| Hostname | SW-D-DEIE |
| Cisco IOS Version | 15.2 |
| Device Type | Layer 3 Switch |
| Department | DEIE |
| Routing Protocol | OSPF |
| OSPF Process ID | 1 |
| OSPF Area | Area 0 |
| Router ID | 21.21.21.21 |
| DHCP Relay | Enabled (helper 10.10.40.10) |
| Management | SSH / HTTP / HTTPS |

---

# Network Topology

```
                        SW-CORE-1
                            |
                    Gi0/1 (OSPF)
                            |
                    +----------------+
                    |   SW-D-DEIE    |
                    +----------------+
                       /            \
              Gi0/2 (OSPF)       Gi0/0
                     /                \
              SW-CORE-2         Trunk (VLAN 10,99,100)
                                         \
                                     SW-A-DEIE
```

---

# Interface Configuration

| Interface | IP Address / VLAN | Description | Role |
|------------|------------|-------------|------|
| GigabitEthernet0/0 | Trunk (VLAN 10,99,100) | Trunk link down to SW-A-DEIE | Access Trunk |
| GigabitEthernet0/1 | 10.255.255.1/30 | Uplink to SW-CORE-1 | OSPF |
| GigabitEthernet0/2 | 10.255.255.5/30 | Uplink to SW-CORE-2 | OSPF |
| Vlan10 | 10.10.10.254/24 | Default Gateway for DEIE Department | SVI |
| Vlan99 | 10.99.10.21/24 | DEIE Local Management | SVI |

---

# Cisco IOS Configuration

```cisco
enable
configure terminal

! ==========================================================
! Basic Configuration
! ==========================================================

hostname SW-D-DEIE

service timestamps debug datetime msec
service timestamps log datetime msec
service compress-config
no service password-encryption

ip cef
no ipv6 cef

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

interface GigabitEthernet0/1
 no switchport
 ip address 10.255.255.1 255.255.255.252
 ip ospf 1 area 0
 negotiation auto

!

interface GigabitEthernet0/2
 no switchport
 ip address 10.255.255.5 255.255.255.252
 ip ospf 1 area 0
 negotiation auto

! ==========================================================
! Trunk Interfaces
! ==========================================================

interface GigabitEthernet0/0
 description Trunk link down to SW-A-DEIE
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 100
 switchport trunk allowed vlan 10,99,100
 negotiation auto

! ==========================================================
! Unused Interfaces
! ==========================================================

interface range GigabitEthernet0/3, GigabitEthernet1/0 - 3, GigabitEthernet2/0 - 3, GigabitEthernet3/0 - 3
 negotiation auto

! ==========================================================
! VLAN Interfaces
! ==========================================================

interface Vlan10
 description Default Gateway for DEIE Department
 ip address 10.10.10.254 255.255.255.0
 ip helper-address 10.10.40.10
 ip ospf 1 area 0

!

interface Vlan99
 description DEIE Local Management
 ip address 10.99.10.21 255.255.255.0
 ip ospf 1 area 0

! ==========================================================
! OSPF Configuration
! ==========================================================

router ospf 1
 router-id 21.21.21.21

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
