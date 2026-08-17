# R-EDGE Router Configuration

## Overview

The **R-EDGE** router functions as the enterprise edge gateway connecting the internal OSPF network to the Internet Service Provider (ISP). It performs **Port Address Translation (PAT)** to provide Internet access for internal networks while advertising the default route to the OSPF domain. The router also provides secure remote management using SSH.

---

# Device Information

| Parameter | Value |
|------------|----------------|
| Hostname | R-EDGE |
| Cisco IOS Version | 15.1 |
| Routing Protocol | OSPF |
| OSPF Area | Area 0 |
| Router ID | 9.9.9.9 |
| NAT | PAT (Overload) |
| Internet Connection | DHCP |
| Remote Management | SSH Version 2 |

---

# Network Topology

```
                        Internet
                            |
                     DHCP Address
                            |
                    GigabitEthernet3/0
                            |
                    +----------------+
                    |    R-EDGE      |
                    +----------------+
                    |                |
             Gig1/0 |                | Gig2/0
                    |                |
               R-CORE-1          R-CORE-2
```

# Interface Configuration

| Interface | IP Address | Description | Role |
|------------|------------|-------------|------|
| FastEthernet0/0 | 10.99.99.1/24 | OOB Management Link | Management |
| GigabitEthernet1/0 | 10.255.255.46/30 | Link to R-CORE-1 | Internal |
| GigabitEthernet2/0 | 10.255.255.50/30 | Link to R-CORE-2 | Internal |
| GigabitEthernet3/0 | DHCP | ISP Connection | Internet |

---

# Cisco IOS Configuration

```cisco
enable
configure terminal

hostname R-EDGE

no aaa new-model

ip source-route
no ip icmp rate-limit unreachable
ip cef
no ip domain lookup
ip domain name eng.ruh.lk
no ipv6 cef

username admin privilege 15 secret 5 $1$15m6$jgiHywdvy3n/sOz900VDO/

ip tcp synwait-time 5
ip ssh version 2

interface FastEthernet0/0
 description OOB Management Link to SW-CORE-1
 ip address 10.99.99.1 255.255.255.0
 duplex full
exit

interface GigabitEthernet1/0
 description Link to R-CORE-1
 ip address 10.255.255.46 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 0
 negotiation auto
exit

interface GigabitEthernet2/0
 description Link to R-CORE-2
 ip address 10.255.255.50 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 0
 negotiation auto
exit

interface GigabitEthernet3/0
 description Link to ISP / Internet
 ip address dhcp
 ip nat outside
 ip virtual-reassembly in
 negotiation auto
exit

interface FastEthernet4/0
 no ip address
 shutdown
 duplex half
exit

router ospf 1
 router-id 9.9.9.9
 default-information originate
exit

ip forward-protocol nd
no ip http server
no ip http secure-server

ip nat inside source list 1 interface GigabitEthernet3/0 overload

ip route 0.0.0.0 0.0.0.0 192.168.42.1 254

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

access-list 1 permit 10.99.99.6
access-list 1 permit 10.10.40.10
access-list 1 permit 10.10.10.0 0.0.0.255
access-list 1 permit 10.10.20.0 0.0.0.255

snmp-server community FoE-Network RO

line con 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
 stopbits 1
exit

line aux 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous
 stopbits 1
exit

line vty 0 4
 access-class ACL_MGMT_ACCESS in
 login local
 transport input ssh
exit

end
write memory
```