# Lab 03 — Static Routing

## 🎯 Objective
Configure static routes on multiple Cisco routers to enable communication between different networks. Understand how routers make forwarding decisions and manually define the paths traffic should take across a multi-router topology. Verify full end-to-end connectivity across all networks.

---

## 🗺️ Topology

<img width="2560" height="1080" alt="Lab 3 Network Topology" src="https://github.com/user-attachments/assets/d0870f9e-883c-4c5e-b701-b53f0a41788d" />

```
[ PC0 ]---[ R1 ]---[ R2 ]---[ R3 ]---[ PC3 ]
           |                  |
         [ PC1 ]           [ PC2 ]

Network Layout:
  PC0:  192.168.10.0/24  (R1 LAN)
  PC1:  192.168.20.0/24  (R1 LAN 2)
  Link: 10.0.0.0/30      (R1 <-> R2)
  Link: 10.0.1.0/30      (R2 <-> R3)
  PC2:  192.168.30.0/24  (R3 LAN)
  PC3:  192.168.40.0/24  (R3 LAN 2)
```

**Devices Used:**
| Device | Model | Role |
|--------|-------|------|
| R1 | Cisco 2911 | Edge Router - Site A |
| R2 | Cisco 2911 | Core Router |
| R3 | Cisco 2911 | Edge Router - Site B |
| SW1 | Cisco 2960 | Access Switch - Site A |
| SW2 | Cisco 2960 | Access Switch - Site B |
| PC0, PC1 | Generic PC | End Hosts - Site A |
| PC2, PC3 | Generic PC | End Hosts - Site B |

---

## 📋 IP Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|--------|-----------|------------|-------------|-----------------|
| R1 | G0/0 | 192.168.10.1 | 255.255.255.0 | — |
| R1 | G0/1 | 192.168.20.1 | 255.255.255.0 | — |
| R1 | S0/0/0 | 10.0.0.1 | 255.255.255.252 | — |
| R2 | S0/0/0 | 10.0.0.2 | 255.255.255.252 | — |
| R2 | S0/0/1 | 10.0.1.1 | 255.255.255.252 | — |
| R3 | S0/0/0 | 10.0.1.2 | 255.255.255.252 | — |
| R3 | G0/0 | 192.168.30.1 | 255.255.255.0 | — |
| R3 | G0/1 | 192.168.40.1 | 255.255.255.0 | — |
| PC0 | NIC | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC1 | NIC | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |
| PC2 | NIC | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 |
| PC3 | NIC | 192.168.40.10 | 255.255.255.0 | 192.168.40.1 |

> 💡 **Note on /30 subnets:** The WAN links between routers use /30 (255.255.255.252) subnets because they only need 2 usable host addresses — one for each end of the link. This conserves IP space.

---

## 📖 Key Concepts Before You Start

- **Static Route:** A manually configured route telling a router how to reach a specific network via a next-hop IP address or exit interface.
- **Next-Hop:** The IP address of the next router in the path toward the destination.
- **Routing Table:** A router's internal map of known networks. Use `show ip route` to view it.
- **Directly Connected Route:** Networks automatically added to the routing table when an interface is configured and active (shown as `C` in `show ip route`).
- **Static Route Code:** Static routes appear as `S` in the routing table.
- **Bidirectional Routing:** For two devices to communicate, routes must exist in **both directions**. A common mistake is only configuring routes one way.
- **/30 Subnet:** Used for point-to-point WAN links — provides exactly 2 usable host IPs.

---

## ⚙️ Configuration

### R1 — Interface Configuration
```
enable
configure terminal
hostname R1
!
interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 description LAN-A-Network1
 no shutdown
!
interface GigabitEthernet0/1
 ip address 192.168.20.1 255.255.255.0
 description LAN-A-Network2
 no shutdown
!
interface Serial0/0/0
 ip address 10.0.0.1 255.255.255.252
 description WAN-Link-to-R2
 no shutdown
```

### R1 — Static Routes
```
! Route to R2-R3 WAN link
ip route 10.0.1.0 255.255.255.252 10.0.0.2
!
! Routes to Site B networks (via R2)
ip route 192.168.30.0 255.255.255.0 10.0.0.2
ip route 192.168.40.0 255.255.255.0 10.0.0.2
!
end
write memory
```

### R2 — Interface Configuration
```
enable
configure terminal
hostname R2
!
interface Serial0/0/0
 ip address 10.0.0.2 255.255.255.252
 description WAN-Link-to-R1
 no shutdown
!
interface Serial0/0/1
 ip address 10.0.1.1 255.255.255.252
 description WAN-Link-to-R3
 no shutdown
```

### R2 — Static Routes
```
! Routes to Site A networks (via R1)
ip route 192.168.10.0 255.255.255.0 10.0.0.1
ip route 192.168.20.0 255.255.255.0 10.0.0.1
!
! Routes to Site B networks (via R3)
ip route 192.168.30.0 255.255.255.0 10.0.1.2
ip route 192.168.40.0 255.255.255.0 10.0.1.2
!
end
write memory
```

### R3 — Interface Configuration
```
enable
configure terminal
hostname R3
!
interface Serial0/0/0
 ip address 10.0.1.2 255.255.255.252
 description WAN-Link-to-R2
 no shutdown
!
interface GigabitEthernet0/0
 ip address 192.168.30.1 255.255.255.0
 description LAN-B-Network1
 no shutdown
!
interface GigabitEthernet0/1
 ip address 192.168.40.1 255.255.255.0
 description LAN-B-Network2
 no shutdown
```

### R3 — Static Routes
```
! Route to R1-R2 WAN link
ip route 10.0.0.0 255.255.255.252 10.0.1.1
!
! Routes to Site A networks (via R2)
ip route 192.168.10.0 255.255.255.0 10.0.1.1
ip route 192.168.20.0 255.255.255.0 10.0.1.1
!
end
write memory
```

---

## ✅ Verification

### Check routing tables on all routers
```
R1# show ip route
R2# show ip route
R3# show ip route
```
✔️ You should see `S` (static) entries for all remote networks and `C` (connected) entries for directly connected networks.
<img width="1440" height="900" alt="R0 Show IP Route" src="https://github.com/user-attachments/assets/dd4e2871-5104-416e-a088-67e77445c007" />
<img width="1440" height="900" alt="R1 Show IP Route" src="https://github.com/user-attachments/assets/03ee9c13-e60d-4b8e-9106-243e3ba2271f" />
<img width="1440" height="900" alt="R2 Show IP Route" src="https://github.com/user-attachments/assets/e57234e5-55aa-4bd9-9fdb-7d2afbbfd11d" />

### Check interfaces are up
```
R1# show ip interface brief
R2# show ip interface brief
R3# show ip interface brief
```
✔️ All configured interfaces should show `up/up` — not `administratively down`.
<img width="1440" height="900" alt="R0 Show IP Interface Brief" src="https://github.com/user-attachments/assets/95a42706-dec3-47f8-9f76-7a1b285023c7" />
<img width="1440" height="900" alt="R1 Show IP Interface Brief" src="https://github.com/user-attachments/assets/502c7f39-704e-414c-a15a-7a8542a535e1" />
<img width="1440" height="900" alt="R2 Show IP Interface Brief" src="https://github.com/user-attachments/assets/ffd8a7e0-3450-4538-93cc-d49dce59a090" />

### Test connectivity — Same site (should SUCCEED ✅)
```
PC0> ping 192.168.20.10    (PC0 to PC1 — same router, different LAN)
PC2> ping 192.168.40.10    (PC2 to PC3 — same router, different LAN)
```
<img width="1440" height="900" alt="Ping Test (PC0 to PC1 — same router, different LAN)" src="https://github.com/user-attachments/assets/71ca6792-fe04-4d33-8a59-77f293e77a99" />
<img width="1440" height="900" alt="Ping Test (PC2 to PC3 — same router, different LAN)" src="https://github.com/user-attachments/assets/87de65eb-3e0a-4610-9105-b5b2f16d09bf" />

### Test connectivity — Across routers (should SUCCEED ✅)
```
PC0> ping 192.168.30.10    (Site A to Site B — across R1, R2, R3)
PC0> ping 192.168.40.10    (Site A to Site B LAN 2)
PC1> ping 192.168.30.10    (Site A LAN 2 to Site B)
PC3> ping 192.168.10.10    (Site B to Site A — reverse path)
```
<img width="1440" height="900" alt="Ping Test (Site A to Site B — across R1, R2, R3)" src="https://github.com/user-attachments/assets/1a29ad95-a4c6-4727-b061-6f9fa4fac41c" />
<img width="1440" height="900" alt="Ping Test (Site A to Site B LAN 2)" src="https://github.com/user-attachments/assets/3e3f26ea-7507-496d-b752-6ed000adec7f" />
<img width="1440" height="900" alt="Ping Test (Site A LAN 2 to Site B)" src="https://github.com/user-attachments/assets/823c4344-00b8-4bbd-8051-347d52cabfaf" />
<img width="1440" height="900" alt="Ping Test (Site B to Site A — reverse path)" src="https://github.com/user-attachments/assets/d49a6c44-8366-47df-ab03-b8bf831dc5dc" />

### Trace the path (bonus)
```
PC0> tracert 192.168.40.10
```
✔️ You should see the packet hop through R1 → R2 → R3 before reaching PC3. This visually confirms your static routes are working hop by hop.
<img width="1440" height="900" alt="Tracert Test (PC0 to PC3)" src="https://github.com/user-attachments/assets/b8c00b05-eda1-4a8b-824c-4f3be8bbacae" />

---

## 🔍 Troubleshooting Tips

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| Interface shows `administratively down` | Missing `no shutdown` | Enter interface and run `no shutdown` |
| `show ip route` missing static entries | Route not configured or typo in IP | Re-enter the `ip route` command carefully |
| Ping succeeds one way but not the other | Missing return route | Check routes on the destination router — routing must be bidirectional |
| Ping fails across all routers | WAN link down | Check Serial interfaces with `show ip interface brief` |
| Tracert stops at a specific router | That router has no route to destination | Add the missing static route on that router |

---

## 💡 Key Takeaways

- ✅ Understood how routers use routing tables to forward traffic
- ✅ Configured static routes using next-hop IP addresses
- ✅ Used /30 subnets for point-to-point WAN links
- ✅ Understood that routing must be configured in both directions
- ✅ Used `show ip route` to verify static and connected routes
- ✅ Used `tracert` to visualize the hop-by-hop path through the network
- ✅ Diagnosed and fixed a routing issue using troubleshooting steps

---

## 🔗 Resources
- [CLI Command Reference](../../notes/cli-commands.md)
- [Subnetting Cheat Sheet](../../notes/subnetting.md)

---

**Time to Complete:** 65 minutes · **Date:** 03/23/2026
