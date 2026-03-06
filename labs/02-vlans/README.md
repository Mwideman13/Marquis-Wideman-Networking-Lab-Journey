# Lab 02 — VLANs & Trunking

## 🎯 Objective
Create and configure VLANs on Cisco switches to segment network traffic, assign access ports to the correct VLANs, and configure trunk links between switches to carry multiple VLANs across the network. Verify that devices in the same VLAN can communicate, and devices in different VLANs cannot (without a router).

---

## 🗺️ Topology
<img width="1440" height="900" alt="Lab 02-VLANs   Trunking (topology screenshot)" src="https://github.com/user-attachments/assets/d39bfe59-0271-42cc-b583-d353562895c1" />

```
[ PC0 - VLAN 10 ]---+                        +---[ PC2 - VLAN 10 ]
                    |                        |
[ PC1 - VLAN 20 ]---[ SW1 ]====TRUNK====[ SW2 ]---[ PC3 - VLAN 20 ]
                    |                        |
[ PC4 - VLAN 30 ]---+                        +---[ PC5 - VLAN 30 ]
```

**Devices Used:**
| Device | Model | Role |
|--------|-------|------|
| SW1 | Cisco 2960 | Distribution Switch |
| SW2 | Cisco 2960 | Access Switch |
| PC0, PC2 | Generic PC | VLAN 10 — Sales |
| PC1, PC3 | Generic PC | VLAN 20 — IT |
| PC4, PC5 | Generic PC | VLAN 30 — Management |

---

## 📋 IP Addressing Table

| Device | VLAN | IP Address | Subnet Mask | Default Gateway |
|--------|------|------------|-------------|-----------------|
| PC0 | 10 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC1 | 20 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |
| PC2 | 10 | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 |
| PC3 | 20 | 192.168.20.11 | 255.255.255.0 | 192.168.20.1 |
| PC4 | 30 | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 |
| PC5 | 30 | 192.168.30.11 | 255.255.255.0 | 192.168.30.1 |

---

## 📖 Key Concepts Before You Start

- **VLAN (Virtual Local Area Network):** Logically segments a switch into separate networks. Devices in different VLANs cannot communicate without a Layer 3 device (router or Layer 3 switch).
- **Access Port:** A switch port assigned to a single VLAN. End devices (PCs) connect here.
- **Trunk Port:** A switch port that carries traffic for multiple VLANs between switches using **802.1Q tagging**.
- **802.1Q Tag:** A 4-byte tag added to Ethernet frames to identify which VLAN the traffic belongs to as it crosses a trunk link.
- **Native VLAN:** Untagged traffic on a trunk link. Default is VLAN 1 — best practice is to change it.

---

## ⚙️ Configuration

### SW1 — Create VLANs
```
enable
configure terminal
hostname SW1
!
! Create VLANs
vlan 10
 name Sales
vlan 20
 name IT
vlan 30
 name Management
```

### SW1 — Assign Access Ports
```
! Assign PC0 to VLAN 10
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 description PC0-Sales
!
! Assign PC1 to VLAN 20
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20
 description PC1-IT
!
! Assign PC4 to VLAN 30
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 30
 description PC4-Management
```

### SW1 — Configure Trunk Port to SW2
```
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 switchport trunk native vlan 99
 description Trunk-to-SW2
!
end
write memory
```

### SW2 — Create VLANs & Assign Access Ports
```
enable
configure terminal
hostname SW2
!
vlan 10
 name Sales
vlan 20
 name IT
vlan 30
 name Management
!
! Assign PC2 to VLAN 10
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 description PC2-Sales
!
! Assign PC3 to VLAN 20
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20
 description PC3-IT
!
! Assign PC5 to VLAN 30
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 30
 description PC5-Management
```

### SW2 — Configure Trunk Port to SW1
```
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 switchport trunk native vlan 99
 description Trunk-to-SW1
!
end
write memory
```

---

## ✅ Verification

Run these commands and screenshot the output for each:

### Check VLANs exist on both switches
```
SW1# show vlan brief
SW2# show vlan brief
```
✔️ You should see VLANs 10, 20, and 30 listed as active with the correct ports assigned.
<img width="1440" height="900" alt="SW1 Show VLAN Brief" src="https://github.com/user-attachments/assets/ca0b74f5-436e-4f73-9fb7-b4fe2817a2cb" />
<img width="1440" height="900" alt="SW2 Show VLAN Brief" src="https://github.com/user-attachments/assets/4bcdcf72-1b0b-4216-a6bd-ddecdc5a6cc5" />

### Check trunk is up
```
SW1# show interfaces trunk
SW2# show interfaces trunk
```
✔️ You should see GigabitEthernet0/1 listed as a trunk carrying VLANs 10, 20, and 30.
<img width="1440" height="900" alt="SW1 Show Interface Trunk" src="https://github.com/user-attachments/assets/89b65420-d2af-4fa3-88aa-d791c028b01a" />
<img width="1440" height="900" alt="SW2 Show Interface Trunk" src="https://github.com/user-attachments/assets/ca485c6e-7f94-4046-a904-7e3fdb018bb3" />

### Test same-VLAN connectivity (should SUCCEED ✅)
```
PC0> ping 192.168.10.11    (PC0 to PC2 — both VLAN 10)
PC1> ping 192.168.20.11    (PC1 to PC3 — both VLAN 20)
PC4> ping 192.168.30.11    (PC4 to PC5 — both VLAN 30)
```
<img width="1440" height="900" alt="Ping PC0 to PC2 (Succeed)" src="https://github.com/user-attachments/assets/a0487193-3232-4053-8fe8-d2205eea4128" />
<img width="1440" height="900" alt="Ping PC1 to PC3 (Succeed)" src="https://github.com/user-attachments/assets/62f1bc07-4703-4fc2-bf5f-5461caaa3037" />
<img width="1440" height="900" alt="Ping PC4 to PC5 (Succeed)" src="https://github.com/user-attachments/assets/800d4cf3-707a-4948-a3fe-62f4a6931576" />

### Test cross-VLAN connectivity (should FAIL ❌)
```
PC0> ping 192.168.20.10    (PC0 VLAN 10 → PC1 VLAN 20)
PC1> ping 192.168.30.10    (PC1 VLAN 20 → PC4 VLAN 30)
```
✔️ These pings **failing** confirms your VLANs are properly isolated — this is correct behavior! Inter-VLAN routing requires a router, which is covered in a future lab.
<img width="1440" height="900" alt="Ping PC0 to PC1 (Failure)" src="https://github.com/user-attachments/assets/20366f26-5808-4436-b6b8-43c2b8c6cc9a" />
<img width="1440" height="900" alt="Ping PC1 to PC4 (Failure)" src="https://github.com/user-attachments/assets/c789b21a-d8c8-4af6-9c2a-bb806d989f51" />

---

## 🔍 Troubleshooting Tips

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| `show vlan brief` doesn't show your VLANs | VLAN not created | Re-enter `vlan 10` etc. in config mode |
| Trunk not showing in `show interfaces trunk` | Port not set to trunk mode | Check `switchport mode trunk` on both ends |
| Same-VLAN ping fails | Port assigned to wrong VLAN | Verify with `show vlan brief` |
| VLANs not passing over trunk | VLAN not allowed on trunk | Run `switchport trunk allowed vlan 10,20,30` |

---

## 💡 Key Takeaways

- ✅ Understood why VLANs are used to segment network traffic
- ✅ Created VLANs and assigned meaningful names
- ✅ Configured access ports to assign end devices to VLANs
- ✅ Configured a trunk link between two switches using 802.1Q
- ✅ Verified same-VLAN communication works
- ✅ Verified cross-VLAN communication is blocked (as expected)
- ✅ Understood the role of the native VLAN on a trunk

---

## 🔗 Resources
- [Cisco VLAN Configuration Guide](https://www.cisco.com/c/en/us/support/docs/lan-switching/vlan/10023-3.html)
- [CLI Command Reference](../../notes/cli-commands.md)
- [Subnetting Cheat Sheet](../../notes/subnetting.md)

---

**Time to Complete:** 60 minutes · **Date:** 3/5/2026
