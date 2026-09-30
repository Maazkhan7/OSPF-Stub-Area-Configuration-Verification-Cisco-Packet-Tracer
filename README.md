# 🌐 OSPF Stub Area Configuration & Verification — Cisco Packet Tracer

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![OSPF](https://img.shields.io/badge/Protocol-OSPFv2-blue?style=for-the-badge)
![Area](https://img.shields.io/badge/Feature-Stub%20Area-orange?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-Intermediate%20%2F%20CCNA%20%2F%20CCNP-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed%20%26%20Verified-success?style=for-the-badge)

> **Reduce routing-table size, cut LSA flooding, and simplify edge routers by turning an OSPF area into a Stub Area — proven with real before/after evidence.**

---

## 📑 Table of Contents

1. [Project Overview](#-project-overview)
2. [Objectives](#-objectives)
3. [Network Topology](#-network-topology)
4. [IP Addressing Table](#-ip-addressing-table)
5. [OSPF Area Design](#-ospf-area-design)
6. [Theory: What Is an OSPF Stub Area?](#-theory-what-is-an-ospf-stub-area)
7. [Configuration](#-configuration)
8. [Verification & Results](#-verification--results)
9. [Before vs After Comparison](#-before-vs-after-comparison)
10. [Troubleshooting Notes](#-troubleshooting-notes)
11. [Key Learnings](#-key-learnings)
12. [Real-World Use Cases](#-real-world-use-cases)
13. [Useful Commands Cheat Sheet](#-useful-commands-cheat-sheet)
14. [Author](#-author)

---

## 📌 Project Overview

This lab demonstrates how to configure and verify an **OSPF Stub Area** in Cisco Packet Tracer using a three-router, two-area design.

- **R1** sits inside **Area 1** (the future stub area).
- **R2** is the **Area Border Router (ABR)** connecting Area 1 and Area 0.
- **R3** sits in **Area 0 (Backbone)** and acts as the **ASBR**, injecting four **external routes** (`192.1.65.0/24` – `192.1.68.0/24`) using static routes pointing to `Null0`.

Before the stub configuration, R1 learns every external route (**Type 5 LSAs → `O E2` routes**). After making Area 1 a stub area, those external routes disappear from R1 and are replaced by a **single default route (`O*IA 0.0.0.0/0`)** generated automatically by the ABR.

---

## 🎯 Objectives

- ✅ Build a multi-area OSPF network (Area 0 + Area 1)
- ✅ Inject external routes from an ASBR (R3) using static routes to `Null0`
- ✅ Capture the routing tables **before** the stub configuration
- ✅ Configure **Area 1 as a stub area** on both R1 and R2
- ✅ Observe the **adjacency reset** and re-formation after the change
- ✅ Prove that the **Backbone (Area 0) cannot be a stub area**
- ✅ Verify Type 5 (external) routes are **filtered** and a **default route** is injected
- ✅ Compare routing tables before vs after

---

## 🗺️ Network Topology

```
                     AREA 1 (STUB)                     |            AREA 0 (BACKBONE)
                                                       |
   Lo1 10.1.1.1/32                Lo1 20.1.1.1/32      |          Lo1 30.1.1.1/32
        ┌────┐   S0/0/0  S0/0/0   ┌────┐  G0/1    G0/1  ┌────┐
        │ R1 │═══════════════════│ R2 │- - - - - - - - -│ R3 │
        └─┬──┘  1.1.1.1/24 1.1.1.2/24 └─┬──┘ 2.1.1.1/24  2.1.1.2/24 └─┬──┘
          │ G0/0                    │ G0/0                          │ G0/0
     192.168.1.0/24            192.168.2.0/24                  192.168.3.0/24
          │                         │                               │
      [Switch0]                 [Switch1]                       [Switch2]
      ┌──┼──┐                   ┌──┼──┐                        ┌──┼──┐
    PC0 PC1 PC2               PC3 PC4 PC5                    PC6 PC7 PC8

   R1 = Internal router (Area 1)
   R2 = ABR  (Area 1 ↔ Area 0)
   R3 = ASBR (Area 0) → static routes to Null0:
        192.1.65.0/24 | 192.1.66.0/24 | 192.1.67.0/24 | 192.1.68.0/24
```

> 📷 *Add your Packet Tracer topology screenshot here:* `![Topology](images/Ospf_stub_area.png)`

---

## 🧾 IP Addressing Table

| Device | Interface | IP Address | Subnet Mask | Connected To | OSPF Area |
|--------|-----------|------------|-------------|--------------|-----------|
| **R1** | Loopback1 | 10.1.1.1 | 255.255.255.255 | — (Router-ID) | Area 1 |
| **R1** | S0/0/0 | 1.1.1.1 | 255.255.255.0 | R2 S0/0/0 | Area 1 |
| **R1** | G0/0 | 192.168.1.1 | 255.255.255.0 | Switch0 (PC0–PC2) | Area 1 |
| **R2** | Loopback1 | 20.1.1.1 | 255.255.255.255 | — (Router-ID) | Area 0 |
| **R2** | S0/0/0 | 1.1.1.2 | 255.255.255.0 | R1 S0/0/0 | Area 1 |
| **R2** | G0/0 | 192.168.2.1 | 255.255.255.0 | Switch1 (PC3–PC5) | Area 1 |
| **R2** | G0/1 | 2.1.1.1 | 255.255.255.0 | R3 G0/1 | Area 0 |
| **R3** | Loopback1 | 30.1.1.1 | 255.255.255.255 | — (Router-ID) | Area 0 |
| **R3** | G0/1 | 2.1.1.2 | 255.255.255.0 | R2 G0/1 | Area 0 |
| **R3** | G0/0 | 192.168.3.1 | 255.255.255.0 | Switch2 (PC6–PC8) | Area 0 |

| LAN | Network | Gateway | Hosts |
|-----|---------|---------|-------|
| Green | 192.168.1.0/24 | R1 G0/0 | PC0, PC1, PC2 |
| Cyan | 192.168.2.0/24 | R2 G0/0 | PC3, PC4, PC5 |
| Yellow | 192.168.3.0/24 | R3 G0/0 | PC6, PC7, PC8 |

---

## 🧩 OSPF Area Design

| Router | Role | Router-ID | Areas |
|--------|------|-----------|-------|
| R1 | Internal Router | 10.1.1.1 | Area 1 |
| R2 | **ABR** | 20.1.1.1 | Area 1 + Area 0 |
| R3 | **ASBR** | 30.1.1.1 | Area 0 |

**External routes injected by R3 (static → Null0):**

```
ip route 192.1.65.0 255.255.255.0 Null0
ip route 192.1.66.0 255.255.255.0 Null0
ip route 192.1.67.0 255.255.255.0 Null0
ip route 192.1.68.0 255.255.255.0 Null0
```

---

## 📚 Theory: What Is an OSPF Stub Area?

A **stub area** is an OSPF area that does **not accept external routes (Type 5 LSAs)**. Instead, the ABR injects a **default route** into the area so internal routers can still reach outside destinations.

| LSA Type | Name | Allowed in Stub Area? |
|----------|------|-----------------------|
| Type 1 | Router LSA | ✅ Yes |
| Type 2 | Network LSA | ✅ Yes |
| Type 3 | Summary LSA (inter-area) | ✅ Yes |
| Type 4 | ASBR Summary LSA | ❌ Blocked |
| Type 5 | External LSA | ❌ Blocked |
| Default | Type 3 default route (0.0.0.0/0) from ABR | ✅ Injected automatically |

### ✔️ Benefits
- Smaller **routing tables** on internal routers
- Smaller **LSDB** and less memory usage
- Less **LSA flooding** and faster convergence
- Lower CPU load on low-end edge routers

### ⚠️ Rules & Restrictions
- **All routers** in the area must be configured as stub (the *E-bit* in Hello packets must match, or the adjacency will not form)
- The **Backbone (Area 0) can never be a stub**
- A stub area **cannot contain an ASBR** (no redistribution inside it)
- **Virtual links cannot** transit a stub area

---

## ⚙️ Configuration

### Step 1 — Baseline (before stub)

OSPF is running with normal areas. Capture the routing tables on **R1** and **R2** and record the external (`O E2`) routes.

### Step 2 — Try to make the Backbone a stub (expected failure)

```text
R2(config)# router ospf 1
R2(config-router)# area 0 stub
OSPF: Backbone can not be configured as stub area
```

> ❌ IOS rejects the command — **Area 0 can never be a stub area.**

### Step 3 — Configure Area 1 as stub on the ABR (R2)

```text
R2(config)# router ospf 1
R2(config-router)# area 1 stub

%OSPF-5-ADJCHG: Process 1, Nbr 10.1.1.1 on Serial0/0/0 from FULL to DOWN, Neighbor Down: Adjacency forced to reset
%OSPF-5-ADJCHG: Process 1, Nbr 10.1.1.1 on Serial0/0/0 from FULL to DOWN, Neighbor Down: Interface down or detached
```

> ⚠️ The neighbor goes **DOWN** because R2 now advertises the stub flag, but R1 does not yet.

### Step 4 — Configure Area 1 as stub on the internal router (R1)

```text
R1(config)# router ospf 1
R1(config-router)# area 1 stub

%OSPF-5-ADJCHG: Process 1, Nbr 20.1.1.1 on Serial0/0/0 from LOADING to FULL, Loading Done
```

> ✅ Flags now match — adjacency returns to **FULL**.

### Step 5 — Full reference configuration

> 🔎 *The `network` statements and R3 redistribution below are the standard configuration for this design — adjust to match your exact lab file.*

**R1**
```text
hostname R1
!
interface Loopback1
 ip address 10.1.1.1 255.255.255.255
!
interface Serial0/0/0
 ip address 1.1.1.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 no shutdown
!
router ospf 1
 router-id 10.1.1.1
 area 1 stub
 network 10.1.1.1 0.0.0.0 area 1
 network 1.1.1.0 0.0.0.255 area 1
 network 192.168.1.0 0.0.0.255 area 1
```

**R2 (ABR)**
```text
hostname R2
!
interface Loopback1
 ip address 20.1.1.1 255.255.255.255
!
interface Serial0/0/0
 ip address 1.1.1.2 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/0
 ip address 192.168.2.1 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/1
 ip address 2.1.1.1 255.255.255.0
 no shutdown
!
router ospf 1
 router-id 20.1.1.1
 area 1 stub
 network 1.1.1.0 0.0.0.255 area 1
 network 192.168.2.0 0.0.0.255 area 1
 network 20.1.1.1 0.0.0.0 area 0
 network 2.1.1.0 0.0.0.255 area 0
```

**R3 (ASBR)**
```text
hostname R3
!
interface Loopback1
 ip address 30.1.1.1 255.255.255.255
!
interface GigabitEthernet0/1
 ip address 2.1.1.2 255.255.255.0
 no shutdown
!
interface GigabitEthernet0/0
 ip address 192.168.3.1 255.255.255.0
 no shutdown
!
ip route 192.1.65.0 255.255.255.0 Null0
ip route 192.1.66.0 255.255.255.0 Null0
ip route 192.1.67.0 255.255.255.0 Null0
ip route 192.1.68.0 255.255.255.0 Null0
!
router ospf 1
 router-id 30.1.1.1
 redistribute static subnets
 network 30.1.1.1 0.0.0.0 area 0
 network 2.1.1.0 0.0.0.255 area 0
 network 192.168.3.0 0.0.0.255 area 0
```

---

## ✅ Verification & Results

### 🔹 1. Routing table on R1 — BEFORE stub

```text
R1# show ip route
Gateway of last resort is not set

C    1.1.1.0/24 is directly connected, Serial0/0/0
O IA 2.1.1.0/24     [110/65] via 1.1.1.2, Serial0/0/0
O IA 20.1.1.1/32    [110/65] via 1.1.1.2, Serial0/0/0
O IA 30.1.1.1/32    [110/66] via 1.1.1.2, Serial0/0/0
O E2 192.1.65.0/24  [110/20] via 1.1.1.2, Serial0/0/0
O E2 192.1.66.0/24  [110/20] via 1.1.1.2, Serial0/0/0
O E2 192.1.67.0/24  [110/20] via 1.1.1.2, Serial0/0/0
O E2 192.1.68.0/24  [110/20] via 1.1.1.2, Serial0/0/0
C    192.168.1.0/24 is directly connected, GigabitEthernet0/0
O    192.168.2.0/24 [110/65] via 1.1.1.2, Serial0/0/0
O IA 192.168.3.0/24 [110/66] via 1.1.1.2, Serial0/0/0
```

> 🔍 R1 knows **all 4 external routes** individually → **4 × `O E2` entries**, and there is **no default route**.

### 🔹 2. Routing table on R2 — BEFORE stub

```text
R2# show ip route
O    10.1.1.1/32    [110/65] via 1.1.1.1, Serial0/0/0
O    30.1.1.1/32    [110/2]  via 2.1.1.2, GigabitEthernet0/1
O E2 192.1.65.0/24  [110/20] via 2.1.1.2, GigabitEthernet0/1
O E2 192.1.66.0/24  [110/20] via 2.1.1.2, GigabitEthernet0/1
O E2 192.1.67.0/24  [110/20] via 2.1.1.2, GigabitEthernet0/1
O E2 192.1.68.0/24  [110/20] via 2.1.1.2, GigabitEthernet0/1
O    192.168.1.0/24 [110/65] via 1.1.1.1, Serial0/0/0
O    192.168.3.0/24 [110/2]  via 2.1.1.2, GigabitEthernet0/1
```

### 🔹 3. Backbone stub attempt (expected error)

```text
R2(config-router)# area 0 stub
OSPF: Backbone can not be configured as stub area
```

### 🔹 4. Adjacency behaviour after `area 1 stub`

| Time | Router | Event |
|------|--------|-------|
| 02:29:51 | R2 | Neighbor 10.1.1.1 **FULL → DOWN** (Adjacency forced to reset) |
| 02:30:47 | R2 | Neighbor 10.1.1.1 **LOADING → FULL** (after R1 was also set to stub) |
| 00:15:46 | R1 | Neighbor 20.1.1.1 **LOADING → FULL, Loading Done** |

### 🔹 5. `show ip ospf` on R1 (internal router)

```text
R1# show ip ospf
 Routing Process "ospf 1" with ID 10.1.1.1
 Number of areas in this router is 1. 0 normal 1 stub 0 nssa
    Area 1
        Number of interfaces in this area is 3
        It is a stub area
```

### 🔹 6. `show ip ospf` on R2 (ABR)

```text
R2# show ip ospf
 Routing Process "ospf 1" with ID 20.1.1.1
 It is an area border router
 Number of areas in this router is 2. 1 normal 1 stub 0 nssa
    Area 1
        Number of interfaces in this area is 2
        It is a stub area
          generates stub default route with cost 1
    Area BACKBONE(0)
        Number of interfaces in this area is 2
```

> 🎯 **Key line:** `generates stub default route with cost 1` — the ABR is the source of the default route.

### 🔹 7. Routing table on R1 — AFTER stub

```text
R1# show ip route
Gateway of last resort is 1.1.1.2 to network 0.0.0.0

C    1.1.1.0/24 is directly connected, Serial0/0/0
O IA 2.1.1.0/24     [110/65] via 1.1.1.2, Serial0/0/0
O IA 20.1.1.1/32    [110/65] via 1.1.1.2, Serial0/0/0
O IA 30.1.1.1/32    [110/66] via 1.1.1.2, Serial0/0/0
C    192.168.1.0/24 is directly connected, GigabitEthernet0/0
O    192.168.2.0/24 [110/65] via 1.1.1.2, Serial0/0/0
O IA 192.168.3.0/24 [110/66] via 1.1.1.2, Serial0/0/0
O*IA 0.0.0.0/0      [110/65] via 1.1.1.2, Serial0/0/0
```

> ✅ All four `O E2` routes are **gone**. A single **`O*IA 0.0.0.0/0`** default route now points to the ABR (R2).

**Why is the default route cost 65?**
`64` (Serial link cost, 100 Mbps ÷ 1.544 Mbps) + `1` (stub default-route cost from the ABR) = **65**

---

## 📊 Before vs After Comparison

| Feature | Before Stub | After Stub |
|---------|-------------|------------|
| External routes on R1 (`O E2`) | 4 routes (192.1.65–68.0/24) | **0 routes** |
| Default route on R1 | ❌ Not set | ✅ `O*IA 0.0.0.0/0` via 1.1.1.2 |
| Gateway of last resort | Not set | 1.1.1.2 |
| Type 5 LSAs in Area 1 | Present | **Filtered by ABR** |
| Inter-area routes (`O IA`) | Present | Still present |
| Area 1 type | Normal | **Stub** |
| Routing table size on R1 | Larger | **Smaller** |
| Connectivity to external networks | Via specific E2 routes | Via default route |

---

## 🛠️ Troubleshooting Notes

| Symptom | Cause | Fix |
|---------|-------|-----|
| `OSPF: Backbone can not be configured as stub area` | Tried `area 0 stub` | Area 0 can never be stub — use `area 1 stub` |
| Neighbor drops to DOWN after configuring stub on one router | Stub flag (E-bit) mismatch in Hello packets | Configure `area X stub` on **every** router in that area |
| Neighbor stuck in INIT / EXSTART | Stub flag mismatch or MTU issue | Verify `show ip ospf` on both ends |
| No default route in stub area | ABR not configured with the stub area | Check `show ip ospf` on the ABR for *generates stub default route* |
| External routes still visible in stub area | Stub not configured on all routers / adjacency not reformed | Verify `It is a stub area` and clear OSPF process if needed |

---

## 💡 Key Learnings

1. **Stub areas block Type 4 & Type 5 LSAs** but still allow Type 1, 2, and 3.
2. The **ABR automatically injects a default route** (Type 3 LSA) into the stub area.
3. **Every router in the area must agree** on the stub flag, otherwise the adjacency will not form.
4. **Area 0 (Backbone) cannot be configured as a stub area.**
5. **Inter-area (`O IA`) routes are NOT filtered** — only external routes are removed.
6. Stub areas dramatically **reduce routing table size** on internal routers.
7. The default route cost = **ABR-to-router link cost + stub default cost (1)**.
8. `show ip ospf` is the fastest way to confirm **stub status** and the **ABR default-route generation**.
9. ASBRs cannot live inside a stub area — if you need redistribution inside, use **NSSA**.

---

## 🏢 Real-World Use Cases

- 🏬 **Branch offices / remote sites** with a single exit toward HQ
- 📡 **ISP edge networks** where access routers do not need full Internet/external tables
- 🧠 **Low-memory or low-CPU routers** at the network edge
- 🔒 **Hub-and-spoke** designs where spokes only need a default route

---

## 🧰 Useful Commands Cheat Sheet

```text
router ospf 1
 area <id> stub                  ! Configure a stub area
 area <id> stub no-summary       ! Totally stubby area (ABR only)
 area <id> nssa                  ! Not-so-stubby area

show ip ospf                     ! Area type, ABR/ASBR role, stub default route
show ip ospf neighbor            ! Adjacency state
show ip ospf database            ! LSDB (verify Type 5 filtered)
show ip route                    ! Routing table / gateway of last resort
show ip route ospf               ! OSPF routes only
show ip protocols                ! OSPF process summary
```

---

## 🚀 Future Improvements

- [ ] Extend to a **Totally Stubby Area** (`area 1 stub no-summary`) and compare
- [ ] Implement an **NSSA** to allow redistribution inside the area
- [ ] Add **OSPF authentication** (MD5) between R1 and R2
- [ ] Automate configuration with **Python (Netmiko)** or **Ansible**
- [ ] Rebuild the topology in **GNS3 / EVE-NG** with real IOS images

---

## 👨‍💻 Author

**Maaz Khan**
🎓 BS Computer Science — University of Malakand
🏅 CCNA Certified | Network & NOC Engineer
📍 Khyber Pakhtunkhwa, Pakistan

[![GitHub](https://img.shields.io/badge/GitHub-maazkhanms-181717?style=for-the-badge&logo=github)](https://github.com/maazkhanms)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/YOUR-LINKEDIN-URL)

⭐ **If this lab helped you, please star the repository and share it with fellow network engineers!**

---

## 🏷️ Tags

`#CCNA` `#CCNP` `#OSPF` `#StubArea` `#CiscoPacketTracer` `#Networking` `#Routing` `#NOC` `#NetworkEngineer` `#LSA`
