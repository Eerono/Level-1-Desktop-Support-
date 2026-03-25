
# L1 Network Triage Guide

## Step-by-Step Flow
1. **Local Loop:** `ping 127.0.0.1` → Fail? `netsh int ip reset && reboot`
2. **Layer 1/2:** `ipconfig` → No IP? Check adapter/cable/switch LED
3. **Layer 3:** `ping <gateway>` → Fail? Verify GW IP on client/router
4. **External:** `ping 8.8.8.8` → Fail? Firewall/router ACL?
5. **DNS:** `nslookup google.com` → Fail? `ipconfig /flushdns` + check DNS server
