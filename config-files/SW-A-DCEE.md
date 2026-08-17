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

hostname SW-A-DCEE

username admin privilege 15 secret 5 $1$15m6$jgiHywdvy3n/sOz900VDO/

no aaa new-model
no ip routing
no ip cef
no ipv6 cef

spanning-tree mode pvst
spanning-tree extend system-id

interface GigabitEthernet0/0
 description Trunk Uplink to SW-D-DCEE
 switchport trunk allowed vlan 20,99,100
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 100
 switchport mode trunk
 negotiation auto
exit

interface GigabitEthernet0/1
 switchport access vlan 20
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
exit

interface GigabitEthernet0/2
 switchport access vlan 20
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
exit

interface GigabitEthernet0/3
 switchport access vlan 20
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
exit

interface GigabitEthernet1/0
 switchport access vlan 20
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
exit

interface GigabitEthernet1/1
 switchport access vlan 20
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
exit

interface GigabitEthernet1/2
 switchport access vlan 20
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
exit

interface GigabitEthernet1/3
 switchport access vlan 20
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
exit

interface GigabitEthernet2/0
 switchport access vlan 20
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
exit

interface GigabitEthernet2/1
 switchport access vlan 20
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
exit

interface GigabitEthernet2/2
 switchport access vlan 20
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
exit

interface GigabitEthernet2/3
 switchport access vlan 20
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
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
 ip address 10.99.20.31 255.255.255.0
 no ip route-cache
exit

ip default-gateway 10.99.20.21
ip forward-protocol nd

ip http server
ip http secure-server

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
