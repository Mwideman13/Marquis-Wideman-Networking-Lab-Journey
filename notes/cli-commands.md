# Cisco IOS CLI Command Reference

## 🔑 Exec Modes
| Command | Description |
|---------|-------------|
| `enable` | Enter privileged EXEC mode |
| `configure terminal` | Enter global configuration mode |
| `end` or `Ctrl+Z` | Return to privileged EXEC mode |
| `exit` | Go back one mode level |
| `write memory` / `copy run start` | Save configuration |

---

## 🔍 Show Commands
| Command | Description |
|---------|-------------|
| `show running-config` | View current active configuration |
| `show startup-config` | View saved configuration |
| `show ip interface brief` | Summary of all interfaces and IP addresses |
| `show interfaces` | Detailed interface statistics |
| `show ip route` | Routing table |
| `show vlan brief` | VLAN table summary |
| `show interfaces trunk` | Trunk port information |
| `show ip ospf neighbor` | OSPF neighbor adjacencies |
| `show ip protocols` | Routing protocol summary |
| `show mac address-table` | MAC address table on a switch |
| `show cdp neighbors` | Directly connected Cisco devices |
| `show spanning-tree` | STP topology |
| `show etherchannel summary` | EtherChannel status |
| `show ip nat translations` | Active NAT translations |
| `show access-lists` | ACL entries and hit counts |
| `show ip dhcp binding` | DHCP address assignments |

---

## 🖥️ Interface Configuration
```
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 description Link to LAN A
 no shutdown
```

---

## 🏷️ VLANs
```
! Create VLAN
vlan 10
 name SALES

! Assign access port
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10

! Configure trunk
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

---

## 🛣️ Routing
```
! Static route
ip route 192.168.2.0 255.255.255.0 10.0.0.2

! Default route
ip route 0.0.0.0 0.0.0.0 10.0.0.1

! OSPF
router ospf 1
 network 192.168.1.0 0.0.0.255 area 0
```

---

## 🔒 ACLs
```
! Standard ACL
access-list 10 permit 192.168.1.0 0.0.0.255

! Extended ACL
access-list 100 permit tcp 192.168.1.0 0.0.0.255 any eq 80

! Apply to interface
interface G0/0
 ip access-group 100 in
```

---

## 🌍 NAT
```
! Define inside/outside
interface G0/0
 ip nat inside
interface G0/1
 ip nat outside

! PAT (overload)
ip nat inside source list 1 interface G0/1 overload
access-list 1 permit 192.168.1.0 0.0.0.255
```

---

## 🏠 DHCP
```
ip dhcp excluded-address 192.168.1.1 192.168.1.10
!
ip dhcp pool LAN_POOL
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8
 lease 7
```

---

## 🔐 Basic Security
```
! Console password
line console 0
 password cisco
 login

! Enable secret
enable secret MySecret123

! SSH setup
ip domain-name lab.local
crypto key generate rsa modulus 1024
ip ssh version 2
line vty 0 4
 transport input ssh
 login local
username admin privilege 15 secret cisco
```
