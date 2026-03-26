# Lab 04 — OSPF (Open Shortest Path First)

## 🎯 Objective
Configure OSPFv2 on multiple Cisco routers to enable dynamic routing within a single area (Area 0). Unlike static routing where you manually define every path, OSPF routers automatically discover neighbors, share network information, and calculate the best path to every destination. Verify neighbor adjacencies form correctly and that all networks are reachable dynamically.

---

## 🗺️ Topology

> _Add your topology screenshot here once the lab is complete._
<img width="2560" height="1080" alt="Lab 04 Network Topology" src="https://github.com/user-attachments/assets/efd98bfb-a4f3-40ef-a518-67543ad73b04" />

```
                    [ PC1 ]
                       |
[ PC0 ]---[ R1 ]---[ R2 ]---[ R3 ]---[ PC2 ]
          10.0.0.0/30  |  10.0.1.0/30
                    [ R4 ]
                       |
                    [ PC3 ]

All routers in OSPF Area 0
```

**Devices Used:**
| Device | Model | Role |
|--------|-------|------|
| R1 | Cisco 2911 | OSPF Router |
| R2 | Cisco 2911 | OSPF Router (Hub) |
| R3 | Cisco 2911 | OSPF Router |
| R4 | Cisco 2911 | OSPF Router |
| PC0–PC3 | Generic PC | End Hosts |

---

## 📋 IP Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|--------|-----------|------------|-------------|-----------------|
| R1 | G0/0 | 192.168.10.1 | 255.255.255.0 | — |
| R1 | S0/0/0 | 10.0.0.1 | 255.255.255.252 | — |
| R2 | S0/0/0 | 10.0.0.2 | 255.255.255.252 | — |
| R2 | S0/0/1 | 10.0.1.1 | 255.255.255.252 | — |
| R2 | S0/0/2 | 10.0.2.1 | 255.255.255.252 | — |
| R2 | G0/0 | 192.168.20.1 | 255.255.255.0 | — |
| R3 | S0/0/0 | 10.0.1.2 | 255.255.255.252 | — |
| R3 | G0/0 | 192.168.30.1 | 255.255.255.0 | — |
| R4 | S0/0/0 | 10.0.2.2 | 255.255.255.252 | — |
| R4 | G0/0 | 192.168.40.1 | 255.255.255.0 | — |
| PC0 | NIC | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC1 | NIC | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |
| PC2 | NIC | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 |
| PC3 | NIC | 192.168.40.10 | 255.255.255.0 | 192.168.40.1 |

---

## 📖 Key Concepts Before You Start

- **OSPF:** A dynamic link-state routing protocol. Routers automatically share topology information and calculate best paths — no manual routes needed like in Lab 03.
- **Area 0 (Backbone Area):** All routers in this lab belong to Area 0, the backbone. Every OSPF network must have an Area 0.
- **Router ID:** A unique identifier for each OSPF router, typically set manually as a loopback address (e.g. 1.1.1.1). If not set, OSPF picks the highest active IP automatically.
- **Neighbor Adjacency:** Before exchanging routes, OSPF routers must become neighbors. You'll verify this with `show ip ospf neighbor`.
- **DR/BDR Election:** On multi-access networks, OSPF elects a Designated Router (DR) and Backup DR (BDR) to reduce routing overhead. On point-to-point serial links this doesn't apply.
- **Wildcard Mask:** The inverse of a subnet mask used in the OSPF `network` command. For a /24 (255.255.255.0) the wildcard is 0.0.0.255. For a /30 (255.255.255.252) the wildcard is 0.0.0.3.
- **Cost:** OSPF chooses the best path based on cost (lower = better). Cost = 100Mbps / interface bandwidth.

---

## ⚙️ Configuration

### R1 — Interfaces & OSPF
```
enable
configure terminal
hostname R1
!
interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 description LAN-PC0
 no shutdown
!
interface Serial0/0/0
 ip address 10.0.0.1 255.255.255.252
 description WAN-to-R2
 no shutdown
!
! Set Router ID using loopback
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
! Enable OSPF
router ospf 1
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
!
end
write memory
```

### R2 — Interfaces & OSPF
```
enable
configure terminal
hostname R2
!
interface GigabitEthernet0/0
 ip address 192.168.20.1 255.255.255.0
 description LAN-PC1
 no shutdown
!
interface Serial0/0/0
 ip address 10.0.0.2 255.255.255.252
 description WAN-to-R1
 no shutdown
!
interface Serial0/0/1
 ip address 10.0.1.1 255.255.255.252
 description WAN-to-R3
 no shutdown
!
interface Serial0/0/2
 ip address 10.0.2.1 255.255.255.252
 description WAN-to-R4
 no shutdown
!
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
!
router ospf 1
 router-id 2.2.2.2
 network 192.168.20.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
 network 10.0.1.0 0.0.0.3 area 0
 network 10.0.2.0 0.0.0.3 area 0
!
end
write memory
```

### R3 — Interfaces & OSPF
```
enable
configure terminal
hostname R3
!
interface Serial0/0/0
 ip address 10.0.1.2 255.255.255.252
 description WAN-to-R2
 no shutdown
!
interface GigabitEthernet0/0
 ip address 192.168.30.1 255.255.255.0
 description LAN-PC2
 no shutdown
!
interface Loopback0
 ip address 3.3.3.3 255.255.255.255
!
router ospf 1
 router-id 3.3.3.3
 network 10.0.1.0 0.0.0.3 area 0
 network 192.168.30.0 0.0.0.255 area 0
!
end
write memory
```

### R4 — Interfaces & OSPF
```
enable
configure terminal
hostname R4
!
interface Serial0/0/0
 ip address 10.0.2.2 255.255.255.252
 description WAN-to-R2
 no shutdown
!
interface GigabitEthernet0/0
 ip address 192.168.40.1 255.255.255.0
 description LAN-PC3
 no shutdown
!
interface Loopback0
 ip address 4.4.4.4 255.255.255.255
!
router ospf 1
 router-id 4.4.4.4
 network 10.0.2.0 0.0.0.3 area 0
 network 192.168.40.0 0.0.0.255 area 0
!
end
write memory
```

---

## ✅ Verification

### Step 1 — Check OSPF neighbors have formed
```
R1# show ip ospf neighbor
R2# show ip ospf neighbor
R3# show ip ospf neighbor
R4# show ip ospf neighbor
```
✔️ You should see each router's neighbors listed with a state of **FULL**. If the state is stuck at INIT or 2WAY, there is a configuration mismatch.

<img width="2560" height="1080" alt="R4 Show OSPF Neighbor" src="https://github.com/user-attachments/assets/3d1889c0-5eac-43b0-b69e-9c1891642cbc" />
<img width="2560" height="1080" alt="R3 Show OSPF Neighbor" src="https://github.com/user-attachments/assets/887ec81c-9a56-40cb-9f95-ddcf9253992b" />
<img width="2560" height="1080" alt="R2 Show OSPF Neighbor" src="https://github.com/user-attachments/assets/1379660c-9b59-41e9-aed9-1af650d3ef8a" />
<img width="2560" height="1080" alt="R1 Show OSPF Neighbor" src="https://github.com/user-attachments/assets/08d7b6af-40df-499c-8d4e-c59f136a7c38" />

### Step 2 — Check routing tables are populated dynamically
```
R1# show ip route
R2# show ip route
```
✔️ You should see **O** (OSPF) entries for all remote networks. This means OSPF learned the routes automatically — no static routes needed!

<img width="2560" height="1080" alt="R2 Show IP Route" src="https://github.com/user-attachments/assets/0b8563d1-2ef4-4503-a1de-2e626a511b0d" />
<img width="2560" height="1080" alt="R1 Show IP Route" src="https://github.com/user-attachments/assets/d0498173-5885-4c1b-83be-95d1c06708d3" />

### Step 3 — Check OSPF is running correctly
```
R1# show ip ospf
R2# show ip ospf
```
✔️ Confirms the Router ID, OSPF process, and Area 0 membership.

<img width="2560" height="1080" alt="R2 Show IP OSPF" src="https://github.com/user-attachments/assets/91857083-ae27-424b-978d-93fec41a498c" />
<img width="2560" height="1080" alt="R1 Show IP OSPF" src="https://github.com/user-attachments/assets/ac61500b-1c41-4510-b763-4478a6054799" />

### Step 4 — Check interface-level OSPF info
```
R2# show ip ospf interface
```
✔️ Shows OSPF cost, hello/dead timers, and DR/BDR status per interface.

<img width="2560" height="1080" alt="R2 Show IP OSPF Interface" src="https://github.com/user-attachments/assets/a907c09a-e26b-43bc-9503-7b72ae012ae6" />

### Step 5 — Test end-to-end connectivity
```
PC0> ping 192.168.30.10    (R1 LAN to R3 LAN)
PC0> ping 192.168.40.10    (R1 LAN to R4 LAN)
PC2> ping 192.168.40.10    (R3 LAN to R4 LAN)
PC3> ping 192.168.10.10    (R4 LAN back to R1 LAN)
```
<img width="2560" height="1080" alt="Ping R4 LAN to R1 LAN" src="https://github.com/user-attachments/assets/7cf35a4e-2d13-45b6-a8c3-6ad3b2d74cff" />
<img width="2560" height="1080" alt="Ping R3 LAN to R4 LAN" src="https://github.com/user-attachments/assets/3854c6c5-8937-44ac-b961-f61b6870f859" />
<img width="2560" height="1080" alt="Ping R1 LAN to R4 LAN" src="https://github.com/user-attachments/assets/62f19d4d-f820-47bd-96f7-578ee781d90b" />
<img width="2560" height="1080" alt="Ping R1 LAN to R3 LAN" src="https://github.com/user-attachments/assets/fe54c6ab-399c-4140-ac02-74093cf1326c" />

### Step 6 — Trace the path (bonus)
```
PC0> tracert 192.168.40.10
```
✔️ Should show hops through R1 → R2 → R4 confirming OSPF chose the optimal path.

<img width="2560" height="1080" alt="Tracert to R4 LAN" src="https://github.com/user-attachments/assets/c8e27a3c-61eb-4fde-a276-53792972ac88" />

---

## 🔍 Troubleshooting Tips

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| No OSPF neighbors showing | Interface down or wrong network statement | Check `show ip interface brief` and verify `network` commands |
| Neighbor stuck at INIT | Hello packets not being received | Check interfaces are up on both ends |
| Neighbor stuck at 2WAY | DR/BDR election issue on serial link | Add `ip ospf network point-to-point` on serial interfaces |
| Routes not appearing as `O` | Network not advertised in OSPF | Verify wildcard masks in `network` statements |
| Ping fails despite OSPF routes | Missing network in OSPF | Check `show ip route` for gaps in routing table |

---

## 💡 Key Takeaways

- ✅ Understood why dynamic routing protocols like OSPF are preferred over static routes at scale
- ✅ Configured OSPFv2 with a manually set Router ID using a loopback interface
- ✅ Advertised networks into OSPF using the `network` command with wildcard masks
- ✅ Verified OSPF neighbor adjacencies reached FULL state
- ✅ Read the routing table and identified OSPF-learned routes marked with `O`
- ✅ Understood OSPF Area 0 and why all routers must belong to the backbone area
- ✅ Tested full end-to-end connectivity across a 4-router topology

---

## 🔗 Resources
- [CLI Command Reference](../../notes/cli-commands.md)
- [Subnetting Cheat Sheet](../../notes/subnetting.md)

---

**Time to Complete:** 57 minutes · **Date:** 3/26/2026
