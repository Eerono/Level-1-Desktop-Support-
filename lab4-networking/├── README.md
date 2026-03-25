
# Lab 4: Cisco Networking — Packet Tracer

## Overview
2-subnet topology with inter-VLAN routing. Verified ping across subnets.

## Topology
Internet (209.165.200.254)
|
Router0 (Gig0/0: 192.168.1.1 / Gig0/1: 192.168.2.1)
|
| |
Switch0 (VLAN1) Switch1 (VLAN1)
PC0(192.168.1.10) PC3(192.168.2.10)
PC1(192.168.1.11) PC4(192.168.2.11)
PC2(192.168.1.12)

text

## Setup Instructions (15 mins)

### 1. Build in Packet Tracer
Add: 1 Router (2911), 2 Switches (2960), 5 PCs
Cables: Copper Straight-Through

text

### 2. PC Config (Desktop → IP Configuration → DHCP)

### 3. Router CLI Config
enable
conf t
interface gig0/0
ip address 192.168.1.1 255.255.255.0
no shut
ip dhcp pool IT_SUBNET
network 192.168.1.0 255.255.255.0
default-router 192.168.1.1
dns-server 8.8.8.8

text

## Test
PC0> ping 192.168.2.10 → Success (inter-subnet)

text
