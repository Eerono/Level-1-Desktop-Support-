# 🔧 Lab 4: Cisco Networking — Packet Tracer Playbook

> **Goal:** Build a 2-subnet topology, configure inter-VLAN routing, and verify cross-subnet connectivity using Cisco Packet Tracer.

---

## 📋 Lab Specs

| Field | Details |
|---|---|
| Simulator | Cisco Packet Tracer |
| Estimated Time | ~15 minutes |
| Difficulty | Introductory |
| Outcome | Successful ping across subnets |

---

## 🗺️ Topology

```
Internet (209.165.200.254)
          |
       Router0
    Gig0/0      Gig0/1
192.168.1.1    192.168.2.1
      |               |
   Switch0          Switch1
   (VLAN1)          (VLAN1)
  /   |   \          /   \
PC0  PC1  PC2      PC3   PC4
.10  .11  .12      .10   .11
```

### IP Address Reference

| Device | Interface | IP Address | Subnet Mask |
|---|---|---|---|
| Router0 | Gig0/0 | 192.168.1.1 | 255.255.255.0 |
| Router0 | Gig0/1 | 192.168.2.1 | 255.255.255.0 |
| PC0 | NIC | 192.168.1.10 (DHCP) | 255.255.255.0 |
| PC1 | NIC | 192.168.1.11 (DHCP) | 255.255.255.0 |
| PC2 | NIC | 192.168.1.12 (DHCP) | 255.255.255.0 |
| PC3 | NIC | 192.168.2.10 (DHCP) | 255.255.255.0 |
| PC4 | NIC | 192.168.2.11 (DHCP) | 255.255.255.0 |

---

## 🚀 Quickstart

```
Step 1 → Build topology
Step 2 → Set PCs to DHCP
Step 3 → Configure router interfaces + DHCP pools
Step 4 → Verify IP addressing
Step 5 → Test with ping
```

---

## Step 1 — Build the Topology

**Devices to add:**
- `1x` Router — **Cisco 2911**
- `2x` Switch — **Cisco 2960**
- `5x` PC (generic)

**Cables — all Copper Straight-Through:**

| From | To |
|---|---|
| Router0 Gig0/0 | Switch0 (any Fa port) |
| Router0 Gig0/1 | Switch1 (any Fa port) |
| Switch0 | PC0, PC1, PC2 |
| Switch1 | PC3, PC4 |

> **Tip:** Links will show amber until interfaces are configured and brought up.

---

## Step 2 — Configure PCs

For each PC (PC0 through PC4):

1. Click the PC → **Desktop** tab → **IP Configuration**
2. Select **DHCP**

> PCs won't get addresses until the router DHCP pools are set up in Step 3.

---

## Step 3 — Configure the Router

Click **Router0** → **CLI** tab. Enter the commands below.

### Enter Config Mode

```
Router> enable
Router# conf t
```

### Configure Gig0/0 — Subnet 1

```
Router(config)# interface gig0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shut
Router(config-if)# exit
```

### Configure Gig0/1 — Subnet 2

```
Router(config)# interface gig0/1
Router(config-if)# ip address 192.168.2.1 255.255.255.0
Router(config-if)# no shut
Router(config-if)# exit
```

### DHCP Pool — Subnet 1

```
Router(config)# ip dhcp pool IT_SUBNET
Router(dhcp-config)# network 192.168.1.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.1.1
Router(dhcp-config)# dns-server 8.8.8.8
Router(dhcp-config)# exit
```

### DHCP Pool — Subnet 2

```
Router(config)# ip dhcp pool OFFICE_SUBNET
Router(dhcp-config)# network 192.168.2.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.2.1
Router(dhcp-config)# dns-server 8.8.8.8
Router(dhcp-config)# exit
```

---

## Step 4 — Verify IP Addressing

On each PC, check **Desktop → IP Configuration** and confirm:

| PC | Expected IP | Expected Gateway |
|---|---|---|
| PC0, PC1, PC2 | 192.168.1.x | 192.168.1.1 |
| PC3, PC4 | 192.168.2.x | 192.168.2.1 |

> If a PC still shows `0.0.0.0`, toggle it from DHCP → Static → DHCP to re-trigger a DHCP request.

---

## Step 5 — Test Connectivity

Open **Desktop → Command Prompt** on PC0.

**Intra-subnet (same subnet):**
```
PC0> ping 192.168.1.11
```
Expected: 4/4 replies ✅

**Inter-subnet (across router):**
```
PC0> ping 192.168.2.10
```
Expected: 4/4 replies ✅

**Return path (from Subnet 2):**
```
PC3> ping 192.168.1.10
```
Expected: 4/4 replies ✅

> The first packet may time out while ARP resolves — this is normal. Subsequent packets will succeed.

---

## 🛠️ Troubleshooting

| Symptom | Likely Cause | Fix |
|---|---|---|
| PCs not getting DHCP address | Router interface is down | Re-enter interface and run `no shut` |
| Ping fails within same subnet | Bad cable connection | Check all links show green in Packet Tracer |
| Inter-subnet ping fails | Gig0/1 not configured or down | Verify IP and `no shut` on Gig0/1 |
| Amber link lights | Interface administratively down | Enter interface, run `no shut` |
| DHCP pool conflict error | Excluded addresses overlap | Add `ip dhcp excluded-address` before pool config |

---

## ✅ Completion Checklist

- [ ] All 5 PCs receive DHCP addresses
- [ ] PC0 can ping PC1 (intra-subnet)
- [ ] PC0 can ping PC3 (inter-subnet)
- [ ] PC3 can ping PC0 (return path)
- [ ] Both router interfaces show as `up/up`

---

## 📁 Files in This Repo

| File | Description |
|---|---|
| `PLAYBOOK.md` | This file — quick reference for the lab |
| `Lab4_Cisco_PacketTracer_Guide.docx` | Full step-by-step guide with detail |

---

*Lab 4 — Cisco Networking · Packet Tracer · Inter-VLAN Routing*
