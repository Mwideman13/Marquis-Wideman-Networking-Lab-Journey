# Lab 05 — DHCP & DNS

## 🎯 Objective
Configure a Cisco router as a DHCP server to automatically assign IP addresses to end devices, and set up a DNS server so devices can resolve hostnames to IP addresses. Verify that PCs receive IP configuration automatically and can ping each other by name instead of IP address.

---

## 🗺️ Topology

> _Add your topology screenshot here once the lab is complete._

```
[ PC0 ]---+                          +---[ PC2 ]
          |                          |
       [ SW1 ]---[ R1 ]---[ R2 ]---[ SW2 ]
          |                          |
[ PC1 ]---+                          +---[ PC3 ]

R1 = DHCP Server for LAN A (192.168.10.0/24)
R2 = DHCP Server for LAN B (192.168.20.0/24)
DNS Server = 192.168.10.2 (static IP on LAN A)
```

**Devices Used:**
| Device | Model | Role |
|--------|-------|------|
| R1 | Cisco 2911 | Router + DHCP Server for LAN A |
| R2 | Cisco 2911 | Router + DHCP Server for LAN B |
| SW1 | Cisco 2960 | Access Switch — LAN A |
| SW2 | Cisco 2960 | Access Switch — LAN B |
| PC0, PC1 | Generic PC | DHCP Clients — LAN A |
| PC2, PC3 | Generic PC | DHCP Clients — LAN B |
| DNS Server | Generic Server | Hostname resolution — LAN A |

---

## 📋 IP Addressing Table

| Device | Interface | IP Address | Assigned By |
|--------|-----------|------------|-------------|
| R1 | G0/0 | 192.168.10.1 | Static |
| R1 | S0/0/0 | 10.0.0.1 | Static |
| R2 | S0/0/0 | 10.0.0.2 | Static |
| R2 | G0/0 | 192.168.20.1 | Static |
| DNS Server | NIC | 192.168.10.2 | Static |
| PC0 | NIC | 192.168.10.x | DHCP |
| PC1 | NIC | 192.168.10.x | DHCP |
| PC2 | NIC | 192.168.20.x | DHCP |
| PC3 | NIC | 192.168.20.x | DHCP |

> 💡 PCs will receive IPs automatically from the DHCP pool — you won't set these manually.

---

## 📖 Key Concepts Before You Start

- **DHCP (Dynamic Host Configuration Protocol):** Automatically assigns IP address, subnet mask, default gateway, and DNS server to clients. Without DHCP, every device needs manual IP configuration.
- **DHCP Pool:** A range of IP addresses the DHCP server can hand out to clients.
- **DHCP Exclusion:** Reserves IP addresses so the server never assigns them to clients. Always exclude your router interfaces and static devices (like the DNS server).
- **DHCP Lease:** The amount of time a client holds an assigned IP before it must renew. Default in Packet Tracer is 1 day.
- **DNS (Domain Name System):** Translates human-readable hostnames (like `pc0.lab.local`) into IP addresses. Without DNS, you can only communicate using IP addresses.
- **DNS Record Types:** An **A record** maps a hostname to an IPv4 address. This is what you'll create in Packet Tracer's DNS server.
- **DHCP Relay (ip helper-address):** When a DHCP server is on a different network than the clients, routers don't forward broadcast traffic by default. The `ip helper-address` command tells the router to forward DHCP requests to a specific server across the network.

---

## ⚙️ Configuration

### Step 1 — Configure Router Interfaces

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
end
write memory
```

#### R2
```
enable
configure terminal
hostname R2
!
interface GigabitEthernet0/0
 ip address 192.168.20.1 255.255.255.0
 description LAN-B
 no shutdown
!
interface Serial0/0/0
 ip address 10.0.0.2 255.255.255.252
 description WAN-to-R1
 no shutdown
!
end
write memory
```

---

### Step 2 — Configure Static Routes (so both LANs can reach each other)
```
! On R1
ip route 192.168.20.0 255.255.255.0 10.0.0.2

! On R2
ip route 192.168.10.0 255.255.255.0 10.0.0.1
```

---

### Step 3 — Configure DHCP on R1 (for LAN A)
```
! Exclude static addresses from the pool
ip dhcp excluded-address 192.168.10.1 192.168.10.10
!
! Create the DHCP pool
ip dhcp pool LAN-A
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.10.2
 domain-name lab.local
 lease 1
!
end
write memory
```

> 💡 The exclusion range (192.168.10.1–10) reserves those IPs for routers, servers, and other static devices. DHCP will only hand out .11 and above.

---

### Step 4 — Configure DHCP on R2 (for LAN B)
```
! Exclude static addresses from the pool
ip dhcp excluded-address 192.168.20.1 192.168.20.10
!
! Create the DHCP pool
ip dhcp pool LAN-B
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 192.168.10.2
 domain-name lab.local
 lease 1
!
end
write memory
```

---

### Step 5 — Configure DNS Server (in Packet Tracer GUI)

1. Click on the **DNS Server** device
2. Go to the **Services** tab → click **DNS**
3. Turn DNS service **On**
4. Add the following **A Records**:

| Hostname | IP Address |
|----------|------------|
| pc0.lab.local | 192.168.10.11 |
| pc1.lab.local | 192.168.10.12 |
| pc2.lab.local | 192.168.20.11 |
| pc3.lab.local | 192.168.20.12 |
| r1.lab.local | 192.168.10.1 |
| r2.lab.local | 192.168.20.1 |

> 💡 Note: PC IPs above assume DHCP assigns .11 and .12. Verify actual assigned IPs first with `ipconfig` on each PC, then update your DNS records to match.

---

### Step 6 — Set PC Network Adapters to DHCP

On each PC (PC0, PC1, PC2, PC3):
1. Click the PC → **Desktop** tab → **IP Configuration**
2. Select **DHCP**
3. Wait a moment — the PC should receive an IP address automatically

---

## ✅ Verification

### Check DHCP assignments on routers
```
R1# show ip dhcp binding
R2# show ip dhcp binding
```
✔️ You should see each PC listed with its assigned IP, MAC address, and lease expiry time.

### Check DHCP pool status
```
R1# show ip dhcp pool
R2# show ip dhcp pool
```
✔️ Shows how many addresses have been handed out and how many remain available.

### Check for any DHCP conflicts
```
R1# show ip dhcp conflict
```
✔️ Should return empty. If any IPs appear here, DHCP detected a conflict and pulled that address from the pool.

### Verify PCs received IP configuration
On each PC: **Desktop** → **Command Prompt**
```
PC> ipconfig
```
✔️ Each PC should show an IP in the correct range, the correct default gateway, and the DNS server IP (192.168.10.2).

### Test basic connectivity
```
PC0> ping 192.168.10.1      (ping R1 — same LAN)
PC0> ping 192.168.20.1      (ping R2 — across WAN)
PC0> ping 192.168.20.11     (ping PC2 — cross-LAN)
```

### Test DNS resolution (the exciting part!)
```
PC0> ping pc2.lab.local
PC0> ping r2.lab.local
PC2> ping pc0.lab.local
```
✔️ If DNS is working, these hostnames will resolve to IP addresses and the pings will succeed — no need to remember IP addresses!

### Check DNS server is responding
On any PC:
```
PC> nslookup pc2.lab.local
```
✔️ Should return the IP address mapped to that hostname in your DNS records.

---

## 🔍 Troubleshooting Tips

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| PC not getting an IP | DHCP pool not configured or interface down | Check `show ip dhcp pool` and `show ip interface brief` |
| PC gets 169.254.x.x IP | Failed to reach DHCP server (APIPA address) | Check interface is up and DHCP pool exists |
| Excluded addresses being assigned | Exclusion range entered incorrectly | Re-enter `ip dhcp excluded-address` before the pool |
| DNS ping fails but IP ping works | DNS records not added or DNS service is off | Check DNS server GUI — verify service is On and A records exist |
| nslookup fails | PC not pointing to correct DNS server | Run `ipconfig` — verify DNS server shows 192.168.10.2 |
| Cross-LAN ping fails | Missing static routes | Verify `ip route` on both R1 and R2 |

---

## 💡 Key Takeaways

- [ ] Understood why DHCP is essential in real networks — manual IP config doesn't scale
- [ ] Configured a DHCP pool with network, gateway, DNS, and lease settings
- [ ] Used `ip dhcp excluded-address` to protect static devices from being overwritten
- [ ] Verified DHCP bindings with `show ip dhcp binding`
- [ ] Configured a DNS server with A records mapping hostnames to IPs
- [ ] Successfully pinged devices by hostname instead of IP address
- [ ] Used `nslookup` to verify DNS resolution is working
- [ ] Understood the difference between static and dynamically assigned IPs

---

## 🔗 Resources
- [CLI Command Reference](../../notes/cli-commands.md)
- [Subnetting Cheat Sheet](../../notes/subnetting.md)

---

**Time to Complete:** ___ minutes · **Date:** ___________
