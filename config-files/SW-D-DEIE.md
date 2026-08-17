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

hostname SW-D-DEIE

username admin privilege 15 secret 5 $1$zA2U$SWPbYp301uJzpTw1w1RXZ1

no aaa new-model

ip cef
no ipv6 cef

spanning-tree mode pvst
spanning-tree extend system-id

interface GigabitEthernet0/0
 description Trunk link down to SW-A-DEIE
 switchport trunk allowed vlan 10,99,100
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 100
 switchport mode trunk
 negotiation auto
exit

interface GigabitEthernet0/1
 no switchport
 ip address 10.255.255.1 255.255.255.252
 ip ospf 1 area 0
 negotiation auto
exit

interface GigabitEthernet0/2
 no switchport
 ip address 10.255.255.5 255.255.255.252
 ip ospf 1 area 0
 negotiation auto
exit

interface GigabitEthernet0/3
 negotiation auto
exit

interface GigabitEthernet1/0
 negotiation auto
exit

interface GigabitEthernet1/1
 negotiation auto
exit

interface GigabitEthernet1/2
 negotiation auto
exit

interface GigabitEthernet1/3
 negotiation auto
exit

interface GigabitEthernet2/0
 negotiation auto
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

interface Vlan10
 description Default Gateway for DEIE Department
 ip address 10.10.10.254 255.255.255.0
 ip access-group ACL_DEIE_IN in
 ip helper-address 10.10.40.10
 ip ospf 1 area 0
exit

interface Vlan99
 description DEIE Local Management
 ip address 10.99.10.21 255.255.255.0
 ip ospf 1 area 0
exit

router ospf 1
 router-id 21.21.21.21
exit

ip forward-protocol nd

ip http server
ip http secure-server

ip ssh server algorithm encryption aes128-ctr aes192-ctr aes256-ctr
ip ssh client algorithm encryption aes128-ctr aes192-ctr aes256-ctr

ip access-list extended ACL_DEIE_IN
 remark PERMIT DHCP Requests
 permit udp any any eq bootps
 permit udp any any eq bootpc
 remark PERMIT Engineering staff to Server Farm
 permit ip 10.10.10.0 0.0.0.255 10.10.40.0 0.0.0.255
 remark EXPLICIT DENY to other internal VLANs
 deny ip 10.10.10.0 0.0.0.255 10.10.20.0 0.0.0.255
 deny ip 10.10.10.0 0.0.0.255 10.10.30.0 0.0.0.255
 remark PERMIT Internet Access via R-EDGE
 permit ip 10.10.10.0 0.0.0.255 any
 remark Explicit DENY ALL
 deny ip any any log
exit

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
