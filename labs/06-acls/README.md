# Lab 06 — Access Control Lists (ACLs)

## 🎯 Objective
Configure standard and extended Access Control Lists (ACLs) on Cisco routers to control which traffic is permitted or denied through the network. Understand the difference between standard and extended ACLs, where to place them, and how to verify they are working correctly.

---

## 🗺️ Topology

> _Add your topology screenshot here once the lab is complete._

```
[ PC0 - 192.168.10.10 ]---+
                           |
[ PC1 - 192.168.10.11 ]---[ SW1 ]---[ R1 ]---[ R2 ]---[ SW2 ]---[ Server - 192.168.20.10 ]
                           |                              |
[ PC2 - 192.168.10.12 ]---+              [ PC3 - 192.168.20.11 ]
```

**Devices Used:**
| Device | Model | Role |
|--------|-------|------|
| R1 | Cisco 2911 | Router — ACL enforcement point |
| R2 | Cisco 2911 | Router — destination side |
| SW1 | Cisco 2960 | Access Switch — LAN A |
| SW2 | Cisco 2960 | Access Switch — LAN B |
| PC0 | Generic PC | Permitted host |
| PC1 | Generic PC | Permitted host |
| PC2 | Generic PC | Denied host (used to test ACL blocking) |
| PC3 | Generic PC | LAN B end host |
| Server | Generic Server | Web/HTTP server on LAN B |

---

## 📋 IP Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|--------|-----------|------------|-------------|-----------------|
| R1 | G0/0 | 192.168.10.1 | 255.255.255.0 | — |
| R1 | S0/0/0 | 10.0.0.1 | 255.255.255.252 | — |
| R2 | S0/0/0 | 10.0.0.2 | 255.255.255.252 | — |
| R2 | G0/0 | 192.168.20.1 | 255.255.255.0 | — |
| PC0 | NIC | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC1 | NIC | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 |
| PC2 | NIC | 192.168.10.12 | 255.255.255.0 | 192.168.10.1 |
| PC3 | NIC | 192.168.20.11 | 255.255.255.0 | 192.168.20.1 |
| Server | NIC | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |

---

## 📖 Key Concepts Before You Start

- **ACL:** A list of permit/deny rules applied to a router interface. The router checks each packet against the list top-to-bottom and acts on the first match.
- **Implicit Deny:** Every ACL ends with an invisible `deny any` rule. If a packet doesn't match any rule in the list, it is dropped automatically. This is critical to understand — if you only write a deny rule, everything else gets blocked too.
- **Standard ACL:** Filters traffic based on **source IP address only**. Numbered 1–99 or 1300–1999. Place these **close to the destination** because they can only identify the source.
- **Extended ACL:** Filters traffic based on **source IP, destination IP, protocol, and port number**. Numbered 100–199 or 2000–2699. Place these **close to the source** to stop unwanted traffic as early as possible.
- **Inbound vs Outbound:** `ip access-group ACL_NAME in` filters traffic entering the interface. `ip access-group ACL_NAME out` filters traffic leaving the interface.
- **Wildcard Mask:** The inverse of a subnet mask used in ACL rules. `0.0.0.0` matches exactly one host. `0.0.0.255` matches an entire /24 network. Use the `host` keyword as a shortcut for `0.0.0.0`.

---

## ⚙️ Configuration

### Step 1 — Configure Router Interfaces & Static Routes

#### R1
```
enable
configure terminal
hostname R1
!
interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 description LAN-A
 no shutdown
!
interface Serial0/0/0
 ip address 10.0.0.1 255.255.255.252
 description WAN-to-R2
 no shutdown
!
ip route 192.168.20.0 255.255.255.0 10.0.0.2
!
end
write memory
```

#### R2
```
enable
configure terminal
hostname R2
!
interface Serial0/0/0
 ip address 10.0.0.2 255.255.255.252
 description WAN-to-R1
 no shutdown
!
interface GigabitEthernet0/0
 ip address 192.168.20.1 255.255.255.0
 description LAN-B
 no shutdown
!
ip route 192.168.10.0 255.255.255.0 10.0.0.1
!
end
write memory
```

---

### Step 2 — Verify Baseline Connectivity (Before ACLs)

Before applying any ACLs, confirm all PCs can reach the server:
```
PC0> ping 192.168.20.10    (should succeed ✅)
PC1> ping 192.168.20.10    (should succeed ✅)
PC2> ping 192.168.20.10    (should succeed ✅)
```
> 💡 This is important — always verify your network works BEFORE applying ACLs. If something is broken after you apply an ACL, you'll know the ACL caused it.

---

### Part A — Standard ACL

**Goal:** Block PC2 (192.168.10.12) from reaching LAN B, while allowing PC0 and PC1 through.

Standard ACLs filter on source IP only, so place this **close to the destination** — on R2's G0/0 interface outbound.

#### Create the Standard ACL on R2
```
enable
configure terminal
!
! Permit PC0
access-list 10 permit host 192.168.10.10
!
! Permit PC1
access-list 10 permit host 192.168.10.11
!
! PC2 is implicitly denied by the ACL's implicit deny any
! (no need to write a deny rule for PC2 explicitly)
!
! Apply ACL 10 outbound on R2's LAN B interface
interface GigabitEthernet0/0
 ip access-group 10 out
!
end
write memory
```

#### Verify Standard ACL
```
PC0> ping 192.168.20.10    (should SUCCEED ✅ — permitted)
PC1> ping 192.168.20.10    (should SUCCEED ✅ — permitted)
PC2> ping 192.168.20.10    (should FAIL ❌ — implicitly denied)
```

Check ACL hit counters:
```
R2# show access-lists
```
✔️ You should see match counts incrementing next to each rule — this confirms the ACL is actively filtering traffic.

---

### Part B — Extended ACL

**Goal:** Allow PC0 to access the web server via HTTP (port 80) but block PC0 from pinging (ICMP) the server. PC1 should have full access. PC2 should be blocked entirely.

Extended ACLs filter on source, destination, protocol, and port — place this **close to the source** on R1's G0/0 inbound.

#### Remove the Standard ACL first
```
! On R2 — remove the standard ACL before applying the extended one
interface GigabitEthernet0/0
 no ip access-group 10 out
!
no access-list 10
```

#### Create the Extended ACL on R1
```
enable
configure terminal
!
! Allow PC0 HTTP access to the server
access-list 100 permit tcp host 192.168.10.10 host 192.168.20.10 eq 80
!
! Block PC0 ICMP (ping) to the server
access-list 100 deny icmp host 192.168.10.10 host 192.168.20.10
!
! Allow PC0 all other traffic
access-list 100 permit ip host 192.168.10.10 any
!
! Allow PC1 full access
access-list 100 permit ip host 192.168.10.11 any
!
! Deny PC2 all traffic
access-list 100 deny ip host 192.168.10.12 any
!
! Permit all other traffic (important — prevents blocking unintended traffic)
access-list 100 permit ip any any
!
! Apply ACL 100 inbound on R1's LAN A interface
interface GigabitEthernet0/0
 ip access-group 100 in
!
end
write memory
```

#### Verify Extended ACL
```
PC0> ping 192.168.20.10         (should FAIL ❌ — ICMP blocked)
PC1> ping 192.168.20.10         (should SUCCEED ✅ — full access)
PC2> ping 192.168.20.10         (should FAIL ❌ — all traffic blocked)
```

To test HTTP from PC0:
1. Click PC0 → **Desktop** → **Web Browser**
2. Type `http://192.168.20.10` in the address bar
3. The page should load ✅ — HTTP is permitted even though ping is blocked

Check ACL hit counters:
```
R1# show access-lists
```
✔️ Each rule should show match counts. This is very useful for seeing exactly which rule is firing.

---

## 🔍 Troubleshooting Tips

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| All traffic blocked after ACL | Implicit deny at end of ACL blocking everything | Add `permit ip any any` as the last rule |
| ACL not filtering anything | ACL not applied to interface | Check `show ip interface G0/0` for ACL assignment |
| Wrong traffic being blocked | ACL rules in wrong order | Remember — ACLs match top to bottom, first match wins. Reorder rules |
| Standard ACL blocking too much | Placed too close to source | Move standard ACLs closer to the destination |
| Extended ACL not matching | Wrong protocol or port number | Double check `tcp` vs `udp` and verify port number (HTTP=80, HTTPS=443, DNS=53) |
| Can't remove ACL | ACL still applied to interface | Remove from interface first with `no ip access-group`, then `no access-list` |

---

## 💡 Key Takeaways

- [ ] Understood how ACLs process rules top-to-bottom with first-match logic
- [ ] Understood the implicit deny at the end of every ACL
- [ ] Configured a standard ACL filtering on source IP only
- [ ] Placed the standard ACL close to the destination (best practice)
- [ ] Configured an extended ACL filtering on source, destination, protocol, and port
- [ ] Placed the extended ACL close to the source (best practice)
- [ ] Verified ACL hit counters using `show access-lists`
- [ ] Understood the difference between inbound and outbound ACL placement
- [ ] Removed and replaced an ACL without breaking the network

---

## 🔗 Resources
- [CLI Command Reference](../../notes/cli-commands.md)
- [Subnetting Cheat Sheet](../../notes/subnetting.md)

---

**Time to Complete:** ___ minutes · **Date:** ___________
