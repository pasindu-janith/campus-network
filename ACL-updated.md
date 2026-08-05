# Access Control Lists
Apply this to SW-D-DEIE:
```bash
enable
configure terminal

! ---------------------------------------------------------
! 1. DEIE Policy
! ---------------------------------------------------------
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


interface Vlan10
 ip access-group ACL_DEIE_IN in

```

Apply this to SW-D-DCEE:

```bash
enable
configure terminal
! ---------------------------------------------------------
! 2. DCEE Policy
! ---------------------------------------------------------
ip access-list extended ACL_DCEE_IN
 remark PERMIT DHCP Requests
 permit udp any any eq bootps
 permit udp any any eq bootpc
 
 remark PERMIT DNS Queries to VM-DHCP
 permit udp 10.10.20.0 0.0.0.255 host 10.10.40.10 eq domain
 
 remark PERMIT Admin staff HTTP/HTTPS to DIS
 permit tcp 10.10.20.0 0.0.0.255 10.10.40.0 0.0.0.255 eq www
 permit tcp 10.10.20.0 0.0.0.255 10.10.40.0 0.0.0.255 eq 443
 
 remark EXPLICIT DENY to all other internal VLANs
 deny ip 10.10.20.0 0.0.0.255 10.10.10.0 0.0.0.255
 deny ip 10.10.20.0 0.0.0.255 10.10.30.0 0.0.0.255
 deny ip 10.10.20.0 0.0.0.255 10.10.40.0 0.0.0.255
 
 remark PERMIT Internet Access via R-EDGE
 permit ip 10.10.20.0 0.0.0.255 any
 
 remark EXPLICIT DENY ALL
 deny ip any any log

```

Apply this to SW-D-DMME:

```bash
enable
configure terminal

! ---------------------------------------------------------
! 3. DMME Policy
! ---------------------------------------------------------
ip access-list extended ACL_DMME_IN
 remark PERMIT DHCP Requests
 permit udp any any eq bootps
 permit udp any any eq bootpc
 
 
 remark Explicit DENY to ALL internal VLANs
 deny ip 10.10.30.0 0.0.0.255 10.10.10.0 0.0.0.255
 deny ip 10.10.30.0 0.0.0.255 10.10.20.0 0.0.0.255
 deny ip 10.10.30.0 0.0.0.255 10.10.40.0 0.0.0.255
 
 remark Explicit DENY ALL
 deny ip any any log

```

Apply this to all routers and switches:
```bash
enable
configure terminal

! ---------------------------------------------------------
! Infrastructure Management Policy
! ---------------------------------------------------------
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

line vty 0 4
 access-class ACL_MGMT_ACCESS in
exit

line vty 5 15
 access-class ACL_MGMT_ACCESS in
exit

! ---------------------------------------------------------
! Apply Policy to SNMP Server
! ---------------------------------------------------------
snmp-server community FoE-Network RO ACL_MGMT_ACCESS

write memory

```