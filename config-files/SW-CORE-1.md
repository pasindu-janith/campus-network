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



# Cisco IOS Configuration
```cisco
enable
configure terminal

hostname SW-CORE-1

username admin privilege 15 secret 5 $1$15m6$jgiHywdvy3n/sOz900VDO/

no aaa new-model

ip cef
no ipv6 cef

spanning-tree mode pvst
spanning-tree extend system-id

interface GigabitEthernet0/0
 no switchport
 ip address 10.255.255.2 255.255.255.252
 ip ospf 1 area 0
 negotiation auto
exit

interface GigabitEthernet0/1
 no switchport
 ip address 10.255.255.10 255.255.255.252
 ip ospf 1 area 0
 negotiation auto
exit

interface GigabitEthernet0/2
 no switchport
 ip address 10.255.255.18 255.255.255.252
 ip ospf 1 area 0
 negotiation auto
exit

interface GigabitEthernet0/3
 switchport trunk allowed vlan 99,100
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 100
 switchport mode trunk
 negotiation auto
exit

interface GigabitEthernet1/0
 no switchport
 ip address 10.255.255.29 255.255.255.252
 ip ospf 1 area 0
 negotiation auto
exit

interface GigabitEthernet1/1
 description Trunk Downlink to SW-A-DIS
 switchport trunk allowed vlan 40,99,100
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 100
 switchport mode trunk
 negotiation auto
exit

interface GigabitEthernet1/2
 description Inter-Core L2 Link for HSRP Keepalives
 switchport trunk allowed vlan 40,99,100
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 100
 switchport mode trunk
 negotiation auto
exit

interface GigabitEthernet1/3
 negotiation auto
exit

interface GigabitEthernet2/0
 description OOB Link up to R-EDGE
 switchport access vlan 99
 switchport mode access
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

interface Vlan40
 ip address 10.10.40.252 255.255.255.0
 ip helper-address 10.10.40.10
 standby 40 ip 10.10.40.254
 standby 40 priority 110
 standby 40 preempt
 ip ospf 1 area 0
exit

interface Vlan99
 description Out-of-Band Management
 ip address 10.99.99.11 255.255.255.0
 standby 99 ip 10.99.99.254
 standby 99 priority 110
 standby 99 preempt
 ip ospf 1 area 0
exit

interface Vlan100
 ip address 10.255.255.25 255.255.255.252
 ip ospf 1 area 0
exit

router ospf 1
 router-id 11.11.11.11
exit

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

line vty 5 15
 access-class ACL_MGMT_ACCESS in
 login
exit

end
write memory
```