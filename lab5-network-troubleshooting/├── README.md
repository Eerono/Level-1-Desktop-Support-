
# Lab 5: L1 Network Troubleshooting

## Overview
5 Packet Tracer scenarios simulating helpdesk tickets. Triage with ping/nslookup.

## Scenarios
1. No connectivity (NIC down)
2. DNS failure (wrong server)
3. Gateway missing
4. WiFi "connected no internet"
5. Intermittent ping (cable fault)

## L1 Decision Tree
ping 127.0.0.1? No → netsh winsock reset
Yes → ping gateway? No → Layer 1 (cable/adapter)
Yes → ping 8.8.8.8? No → Routing/GW
Yes → ping domain? No → DNS (nslookup)
