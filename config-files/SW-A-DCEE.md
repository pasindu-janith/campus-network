# SW-A-DCEE Layer 2 Access Switch Configuration

## Overview

**SW-A-DCEE** is the access-layer switch serving end-user devices in the **DCEE (Department of Civil & Environmental Engineering)** department. It operates purely at Layer 2, trunking VLAN 20 (data) and VLAN 99 (management) up to the distribution switch **SW-D-DCEE**, while providing wired access ports for department hosts. Access ports have **PortFast** enabled for faster host connectivity. Local management is reached via the OOB VLAN 99 SVI.

---

# Device Information

| Parameter | Value |
|------------|----------------|
| Hostname | SW-A-DCEE |
| Cisco IOS Version | 15.2 |
| Device Type | Layer 2 Switch |
| Department | DCEE |
| IP Routing | Disabled |
| Access VLAN | VLAN 20 |
| Management VLAN | VLAN 99 |
| PortFast | Enabled on access ports |
| Default Gateway | 10.99.20.21 (SW-D-DCEE) |
| Management | SSH / HTTP / HTTPS |

---

# Network Topology

```
                        SW-D-DCEE
                            |
                Trunk (VLAN 20,99,100)
                            |
                    +----------------+
                    |   SW-A-DCEE    |
                    +----------------+
                     /   /   |   \   \
                   Gi0/1 ... Gi2/3 (Access VLAN 20, PortFast)
                            |
                       End Hosts
```

---

# Interface Configuration

| Interface | IP Address / VLAN | Description | Role |
|------------|------------|-------------|------|
| GigabitEthernet0/0 | Trunk (VLAN 20,99,100) | Trunk Uplink to SW-D-DCEE | Access Trunk |
| GigabitEthernet0/1 - 0/3, 1/0 - 1/3, 2/0 - 2/3 | VLAN 20 | End-Host Access Ports (PortFast) | Access |
| Vlan99 | 10.99.20.31/24 | Out-of-Band Management | Management |

---

# Cisco IOS Configuration

```cisco
enable
configure terminal

! ==========================================================
! Basic Configuration
! ==========================================================

hostname SW-A-DCEE

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
 description Trunk Uplink to SW-D-DCEE
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 100
 switchport trunk allowed vlan 20,99,100
 negotiation auto

! ==========================================================
! Access Interfaces (PortFast Enabled)
! ==========================================================

interface range GigabitEthernet0/1 - 3, GigabitEthernet1/0 - 3, GigabitEthernet2/0 - 3
 switchport access vlan 20
 switchport mode access
 negotiation auto
 spanning-tree portfast edge

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
 ip address 10.99.20.31 255.255.255.0
 no ip route-cache

! ==========================================================
! Default Gateway
! ==========================================================

ip default-gateway 10.99.20.21

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
