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

hostname SW-A-DIS

username admin privilege 15 secret 5 $1$15m6$jgiHywdvy3n/sOz900VDO/

no aaa new-model

ip cef
no ipv6 cef

spanning-tree mode pvst
spanning-tree extend system-id

interface GigabitEthernet0/0
 description Trunk Uplinks to SW-CORE-1 and SW-CORE-2
 switchport trunk allowed vlan 40,99,100
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 100
 switchport mode trunk
 negotiation auto
exit

interface GigabitEthernet0/1
 description Trunk Uplinks to SW-CORE-1 and SW-CORE-2
 switchport trunk allowed vlan 40,99,100
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 100
 switchport mode trunk
 negotiation auto
exit

interface GigabitEthernet0/2
 description Connected to DIS Server Farm Hosts
 switchport access vlan 40
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
exit

interface GigabitEthernet0/3
 description Connected to DIS Server Farm Hosts
 switchport access vlan 40
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
exit

interface GigabitEthernet1/0
 description Connected to DIS Server Farm Hosts
 switchport access vlan 40
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
exit

interface GigabitEthernet1/1
 description Connected to DIS Server Farm Hosts
 switchport access vlan 99
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
exit

interface GigabitEthernet1/2
 description Connected to DIS Server Farm Hosts
 switchport access vlan 99
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
exit

interface GigabitEthernet1/3
 description Link to Admin Physical Laptop
 switchport access vlan 99
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
exit

interface GigabitEthernet2/0
 switchport access vlan 40
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
exit

interface GigabitEthernet2/1
 negotiation auto
exit

interface GigabitEthernet2/2
 negotiation auto
exit

interface GigabitEthernet2/3
 negotiation auto
exit

interface GigabitEthernet3/0
 negotiation auto
exit

interface GigabitEthernet3/1
 negotiation auto
exit

interface GigabitEthernet3/2
 negotiation auto
exit

interface GigabitEthernet3/3
 negotiation auto
exit

interface Vlan99
 description Out-of-Band Management
 ip address 10.99.99.34 255.255.255.0
exit

ip default-gateway 10.99.99.1
ip forward-protocol nd

ip http server
ip http secure-server

ip ssh version 2
ip ssh server algorithm encryption aes128-ctr aes192-ctr aes256-ctr
ip ssh client algorithm encryption aes128-ctr aes192-ctr aes256-ctr

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

snmp-server community FoE-Network RO ACL_MGMT_ACCESS

line con 0
exit

line aux 0
exit

line vty 0 4
 access-class ACL_MGMT_ACCESS in
 login local
 transport input ssh
exit

end
write memory
```