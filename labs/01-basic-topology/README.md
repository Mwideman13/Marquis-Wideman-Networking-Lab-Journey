# Lab 01 — Basic Topology & Cabling

## 🎯 Objective
Build and connect a basic network topology in Cisco Packet Tracer using routers, switches, and end devices. Verify connectivity between all hosts using ping.

---

## 🗺️ Topology

> _Add your topology screenshot here once the lab is complete._

```
[ PC0 ] ---+                        +--- [ PC2 ]
           |                        |
        [ SW1 ] --- [ R1 ] --- [ SW2 ]
           |                        |
[ PC1 ] ---+                        +--- [ PC3 ]
```

**Devices Used:**
| Device | Model | Role |
|--------|-------|------|
| R1 | Cisco 2911 | Router |
| SW1 | Cisco 2960 | Access Switch (LAN A) |
| SW2 | Cisco 2960 | Access Switch (LAN B) |
| PC0–PC3 | Generic PC | End Hosts |

---

## 📋 IP Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|--------|-----------|------------|-------------|-----------------|
| R1 | G0/0 | 192.168.1.1 | 255.255.255.0 | — |
| R1 | G0/1 | 192.168.2.1 | 255.255.255.0 | — |
| PC0 | NIC | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC1 | NIC | 192.168.1.11 | 255.255.255.0 | 192.168.1.1 |
| PC2 | NIC | 192.168.2.10 | 255.255.255.0 | 192.168.2.1 |
| PC3 | NIC | 192.168.2.11 | 255.255.255.0 | 192.168.2.1 |

---

## ⚙️ Configuration

### Router R1
```
enable
configure terminal
hostname R1
!
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/1
 ip address 192.168.2.1 255.255.255.0
 no shutdown
!
end
write memory
```

---

## ✅ Verification

Commands used to verify the lab:
```
R1# show ip interface brief
R1# show interfaces
PC> ping 192.168.2.10
```

> _Add verification screenshots to the `/screenshots` folder and embed them here._

---

## 💡 Key Takeaways

- [ ] Understood the difference between straight-through and crossover cables
- [ ] Practiced assigning IP addresses to router interfaces
- [ ] Verified end-to-end connectivity with ping
- [ ] Learned how to use `show ip interface brief` to check interface status

---

## 🔗 Resources
- [Cisco IOS Interface Configuration](https://www.cisco.com/c/en/us/support/docs/)
- [Subnetting Practice](../notes/subnetting.md)

---

**Time to Complete:** ___ minutes · **Date:** ___________
