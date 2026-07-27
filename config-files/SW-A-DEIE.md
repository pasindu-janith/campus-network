# SW-A-DEIE Layer 2 Access Switch Configuration

## Overview

**SW-A-DEIE** is the access-layer switch serving end-user devices in the **DEIE (Department of Electrical & Information Engineering)** department. It operates purely at Layer 2, trunking VLAN 10 (data) and VLAN 99 (management) up to the distribution switch **SW-D-DEIE**, while providing wired access ports for department hosts. Local management is reached via the OOB VLAN 99 SVI.

---

# Device Information

| Parameter | Value |
|------------|----------------|
| Hostname | SW-A-DEIE |
| Cisco IOS Version | 15.2 |
| Device Type | Layer 2 Switch |
| Department | DEIE |
| IP Routing | Disabled |
| Access VLAN | VLAN 10 |
| Management VLAN | VLAN 99 |
| Default Gateway | 10.99.10.21 (SW-D-DEIE) |
| Management | SSH / HTTP / HTTPS |

---

# Network Topology

```
                        SW-D-DEIE
                            |
                Trunk (VLAN 10,99,100)
                            |
                    +----------------+
                    |   SW-A-DEIE    |
                    +----------------+
                     /   /   |   \   \
                   Gi0/1 ... Gi2/3 (Access VLAN 10)
                            |
                       End Hosts
```

---

# Interface Configuration

| Interface | IP Address / VLAN | Description | Role |
|------------|------------|-------------|------|
| GigabitEthernet0/0 | Trunk (VLAN 10,99,100) | Trunk Uplink to SW-D-DEIE | Access Trunk |
| GigabitEthernet0/1 - 0/3, 1/0 - 1/3, 2/0 - 2/3 | VLAN 10 | End-Host Access Ports | Access |
| Vlan99 | 10.99.10.31/24 | Out-of-Band Management | Management |

---

# Cisco IOS Configuration

```cisco
enable
configure terminal

! ==========================================================
! Basic Configuration
! ==========================================================

hostname SW-A-DEIE

service timestamps debug datetime msec
service timestamps log datetime msec
service compress-config
no service password-encryption

no ip routing
no ip cef
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
! Trunk Interfaces
! ==========================================================

interface GigabitEthernet0/0
 description Trunk Uplink to SW-D-DEIE
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 100
 switchport trunk allowed vlan 10,99,100
 negotiation auto

! ==========================================================
! Access Interfaces
! ==========================================================

interface range GigabitEthernet0/1 - 3, GigabitEthernet1/0 - 3, GigabitEthernet2/0 - 3
 switchport access vlan 10
 switchport mode access
 negotiation auto

! ==========================================================
! Unused Interfaces
! ==========================================================

interface range GigabitEthernet3/0 - 3
 negotiation auto

! ==========================================================
! Management VLAN Interface
! ==========================================================

interface Vlan99
 description Out-of-Band Management
 ip address 10.99.10.31 255.255.255.0
 no ip route-cache

! ==========================================================
! Default Gateway
! ==========================================================

ip default-gateway 10.99.10.21

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
