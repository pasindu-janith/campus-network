# SW-A-DIS Distribution Switch Configuration

## Overview

The **SW-A-DIS** switch functions as the **distribution layer switch** for the server farm and management network. It aggregates access-layer devices and forwards traffic to the redundant core switches through dual trunk uplinks. The switch provides Layer 2 connectivity for server hosts, supports VLAN segmentation, and offers secure management through the Out-of-Band (OOB) management VLAN.

Unlike the core switches, **SW-A-DIS operates as a Layer 2 distribution switch** and relies on the core layer for inter-VLAN routing.

---

# Device Information

| Parameter | Value |
|------------|----------------|
| Hostname | SW-A-DIS |
| Cisco IOS Version | 15.2 |
| Device Type | Layer 2 Distribution Switch |
| Spanning Tree | PVST |
| Management VLAN | VLAN 99 |
| User VLAN | VLAN 40 |
| Native VLAN | VLAN 100 |
| Remote Management | SSH / HTTP / HTTPS |
| SNMP | Enabled (Read-Only) |

---

# Trunk Interfaces

| Interface | Description | Native VLAN | Allowed VLANs |
|-----------|-------------|-------------|---------------|
| GigabitEthernet0/0 | Uplink to Core Switches | 100 | 40, 99, 100 |
| GigabitEthernet0/1 | Uplink to Core Switches | 100 | 40, 99, 100 |



# Cisco IOS Configuration

```cisco
enable
configure terminal

! ==========================================================
! Basic Configuration
! ==========================================================

hostname SW-A-DIS

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
! Trunk Interfaces
! ==========================================================

interface GigabitEthernet0/0
 description Trunk Uplink to Core Switches
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 100
 switchport trunk allowed vlan 40,99,100
 no shutdown

!

interface GigabitEthernet0/1
 description Trunk Uplink to Core Switches
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 100
 switchport trunk allowed vlan 40,99,100
 no shutdown

! ==========================================================
! Access Ports - Server Farm
! ==========================================================

interface GigabitEthernet0/2
 description Server Farm Host
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast edge
 no shutdown

!

interface GigabitEthernet0/3
 description Server Farm Host
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast edge
 no shutdown

!

interface GigabitEthernet1/0
 description Server Farm Host
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast edge
 no shutdown

!

interface GigabitEthernet1/1
 description Management Host
 switchport mode access
 switchport access vlan 99
 spanning-tree portfast edge
 no shutdown

!

interface GigabitEthernet1/2
 description Management Host
 switchport mode access
 switchport access vlan 99
 spanning-tree portfast edge
 no shutdown

!

interface GigabitEthernet1/3
 description Administrator Laptop
 switchport mode access
 switchport access vlan 99
 spanning-tree portfast edge
 no shutdown

! ==========================================================
! Management VLAN
! ==========================================================

interface Vlan99
 description Out-of-Band Management
 ip address 10.99.99.34 255.255.255.0
 no shutdown

! ==========================================================
! Default Gateway
! ==========================================================

ip default-gateway 10.99.99.1

! ==========================================================
! HTTP / HTTPS
! ==========================================================

ip http server
ip http secure-server

! ==========================================================
! SSH
! ==========================================================

ip ssh version 2
ip ssh server algorithm encryption aes128-ctr aes192-ctr aes256-ctr
ip ssh client algorithm encryption aes128-ctr aes192-ctr aes256-ctr

line vty 0 4
 login local
 transport input ssh

! ==========================================================
! SNMP
! ==========================================================

snmp-server community FoE-Network RO

! ==========================================================
! Save Configuration
! ==========================================================

end
write memory
```