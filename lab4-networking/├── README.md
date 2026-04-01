
# Lab 4: Cisco Networking — Packet Tracer
[README (1).md](https://github.com/user-attachments/files/26399557/README.1.md)
# Lab 4: Cisco Networking — Packet Tracer

> Inter-VLAN routing lab using a 2-subnet topology. Configured on Cisco Packet Tracer with a 2911 router, two 2960 switches, and five PCs.

---

## Quick Summary

| Field | Details |
|---|---|
| Simulator | Cisco Packet Tracer |
| Devices | 1x Router (2911), 2x Switch (2960), 5x PC |
| Subnets | 192.168.1.0/24 · 192.168.2.0/24 |
| Routing | Inter-VLAN via router interfaces |
| Addressing | DHCP (server on router) |
| Outcome | Successful ping across subnets ✅ |

---

## Topology

```
         Internet (209.165.200.254)
                    |
                Router0 (2911)
          Gig0/0 ┘         └ Gig0/1
       192.168.1.1           192.168.2.1
            |                     |
        Switch0               Switch1
        (VLAN 1)              (VLAN 1)
       /   |   \               /     \
    PC0   PC1  PC2           PC3     PC4
   .1.10 .1.11 .1.12        .2.10   .2.11
```

---

## Files

| File | Description |
|---|---|
| `README.md` | This file |
| `PLAYBOOK.md` | Quick-reference command guide for the lab |
| `Lab4_Cisco_PacketTracer_Guide.docx` | Full step-by-step lab guide with tables and troubleshooting |

---

## Setup (15 min)

**1. Build the topology**

Add devices and connect with Copper Straight-Through cables:
- Router0 Gig0/0 → Switch0
- Router0 Gig0/1 → Switch1
- Switch0 → PC0, PC1, PC2
- Switch1 → PC3, PC4

**2. Set all PCs to DHCP**

Desktop → IP Configuration → DHCP

**3. Configure the router**

```
enable
conf t

interface gig0/0
 ip address 192.168.1.1 255.255.255.0
 no shut
 exit

interface gig0/1
 ip address 192.168.2.1 255.255.255.0
 no shut
 exit

ip dhcp pool IT_SUBNET
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8
 exit

ip dhcp pool OFFICE_SUBNET
 network 192.168.2.0 255.255.255.0
 default-router 192.168.2.1
 dns-server 8.8.8.8
 exit
```

---

## Test

```
PC0> ping 192.168.2.10   ← inter-subnet ping → should succeed
```

---

## Key Concepts

- **Inter-VLAN routing** — the router forwards packets between subnets using its two interfaces, one per subnet
- **DHCP on the router** — no dedicated server; the router itself hands out addresses via `ip dhcp pool`
- **Default gateway** — each PC's gateway points to the router interface on its subnet (`.1.1` or `.2.1`)

---

*Cisco Networking · Lab 4 · Packet Tracer*
