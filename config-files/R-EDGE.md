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

! ==========================================================
! Basic Configuration
! ==========================================================

hostname R-EDGE
no ip domain lookup
ip domain name eng.ruh.lk

service timestamps debug datetime msec
service timestamps log datetime msec
no service password-encryption

ip cef
ip source-route
no ip icmp rate-limit unreachable
ip tcp synwait-time 5

! ==========================================================
! User Configuration
! ==========================================================

username admin privilege 15 secret YOUR_PASSWORD_HERE

! ==========================================================
! SSH Configuration
! ==========================================================

crypto key generate rsa modulus 2048
ip ssh version 2

line vty 0 4
 login local
 transport input ssh

line console 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous

line aux 0
 exec-timeout 0 0
 privilege level 15
 logging synchronous

! ==========================================================
! Interface Configuration
! ==========================================================

interface FastEthernet0/0
 description OOB Management Link to SW-CORE-1
 ip address 10.99.99.1 255.255.255.0
 duplex full
 no shutdown

!

interface GigabitEthernet1/0
 description Link to R-CORE-1
 ip address 10.255.255.46 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 0
 no shutdown

!

interface GigabitEthernet2/0
 description Link to R-CORE-2
 ip address 10.255.255.50 255.255.255.252
 ip nat inside
 ip virtual-reassembly in
 ip ospf 1 area 0
 no shutdown

!

interface GigabitEthernet3/0
 description Link to ISP / Internet
 ip address dhcp
 ip nat outside
 ip virtual-reassembly in
 no shutdown

! ==========================================================
! OSPF Configuration
! ==========================================================

router ospf 1
 router-id 9.9.9.9
 default-information originate

! ==========================================================
! NAT Configuration
! ==========================================================

access-list 1 permit 10.10.10.0 0.0.0.255
access-list 1 permit 10.10.20.0 0.0.0.255

ip nat inside source list 1 interface GigabitEthernet3/0 overload

! ==========================================================
! Default Route
! ==========================================================

ip route 0.0.0.0 0.0.0.0 192.168.42.1

! ==========================================================
! Disable Unused Services
! ==========================================================

no ip http server
no ip http secure-server

! ==========================================================
! Save Configuration
! ==========================================================

end
write memory
```