
Lab 4 Cheatsheet - Copy this
text
# Network Troubleshooting Cheatsheet

| Issue | Command | Expected |
|-------|---------|----------|
| IP Config | `ipconfig /all` | Valid IP, Gateway, DNS |
| DNS Cache | `ipconfig /flushdns` | "Cleared" |
| Internet | `ping 8.8.8.8` | <50ms replies |
| Name Res | `ping google.com` | Resolves + replies |
| Route | `tracert google.com` | Hops to destination |
